<template>
  <div class="container">
    <div class="row justify-content-center">
      <draggable element="div" class="col-md-12" v-model="categories" :options="dragOptions">
        <transition-group class="row">
          <div class="col-md-4" v-for="(category, index) in categories" :key="category.id">
            <div class="card">
              <div class="card-header">
                <h4 class="card-title">{{ category.name }}</h4>
              </div>
              <div class="card-body card-body-dark">
                <draggable
                  :options="dragOptions"
                  element="div"
                  @end="changeOrder"
                  v-model="category.tasks"
                >
                  <transition-group :id="category.id">
                    <div
                      v-for="task in category.tasks"
                      :key="`${task.category_id},${task.order}`"
                      class="transit-1"
                    >
                      <div class="small-card">
                        <textarea
                          v-if="task === editingTask"
                          class="text-input"
                          @keyup.enter="endEditing(task)"
                          @blur="endEditing(task)"
                          v-model="task.name"
                        ></textarea>
                        <label
                          for="checkbox"
                          v-if="task !== editingTask"
                          @dblclick="editTask(task)"
                        >{{ task.name }}</label>
                      </div>
                    </div>
                  </transition-group>
                </draggable>
                <div class="small-card">
                  <h5 class="text-center" @click="addNew(index)">Add new card</h5>
                </div>
              </div>
            </div>
          </div>
        </transition-group>
      </draggable>
    </div>
  </div>
</template>

<style scoped>
.card {
  border: 0;
  border-radius: 0.5rem;
}
.transit-1 {
  transition: all 1s;
}
.small-card {
  padding: 1rem;
  background: #f5f8fa;
  margin-bottom: 5px;
  border-radius: 0.25rem;
}
.card-body-dark {
  background-color: #ccc;
}
textarea {
  overflow: visible;
  outline: 1px dashed black;
  border: 0;
  padding: 6px 0 2px 8px;
  width: 100%;
  height: 100%;
  resize: none;
}
</style>

<script>
import draggable from "vuedraggable";
export default {
  components: {
    draggable
  },
  data() {
    return {
      categories: [],
      editingTask: null
    };
  },
  methods: {
    addNew(id) {
      const user_id = 1;
      const name = "New task";
      const category_id = this.categories[id].id;
      const order = this.categories[id].tasks.length;

      axios
        .post("api/task", { user_id, name, order, category_id })
        .then(response => {
          this.categories[id].tasks.push(response.data.data);
        })
        .catch(err => {
          console.error(err);
        });
    },
    loadTasks() {
      this.categories.map(category => {
        axios
          .get(`api/category/${category.id}/tasks`)
          .then(response => {
            category.tasks = response.data;
          })
          .catch(err => {
            console.error(err);
          });
      });
    },
    changeOrder(data) {
      const toTask = data.to;
      const fromTask = data.from;
      const task_id = data.item.id;
      const category_id = fromTask.id == toTask.id ? null : toTask.id;
      const order = data.newIndex == data.oldIndex ? false : data.newIndex;

      if (order) {
        axios
          .patch(`api/task/${task.id}`, { order, category_id })
          .then(response => {
            // Something cool can happen here
          })
          .catch(err => {
            console.error(err);
          });
      }
    },
    endEditing(task) {
      this.editingTask = null;

      axios
        .patch(`api/task/${task.id}`, { name: task.name })
        .then(response => {
          // Something cool can happend here
        })
        .catch(err => {
          console.error(err);
        });
    },
    editTask(task) {
      this.editingTask = task;
    }
  },
  mounted() {
    const token = localStorage.getItem("jwt");

    axios.defaults.headers.common["Content-Type"] = "application/json";
    axios.defaults.headers.common["Authorization"] = `Bearer ${token}`;

    axios
      .get("api/category")
      .then(response => {
        this.categories = response.data.map(data => {
          return {
            id: data.id,
            name: data.name,
            tasks: []
          };
        });
        this.loadTasks();
      })
      .catch(err => {
        console.error(err);
      });
  },
  computed: {
    dragOptions() {
      return {
        animation: 1,
        group: "description",
        ghostClass: "ghost"
      };
    }
  },
  beforeRouteEnter(to, from, next) {
    if (!localStorage.getItem("jwt")) return next("login");
    next();
  }
};
</script>

<template>
  <div class="container">
    <div class="row justify-content-center">
      <div class="col-md-8">
        <div class="card card-default">
          <div class="card-header">Register</div>

          <div class="card-body">
            <form action="/register" method="post">
              <div class="form-group row">
                <label for="name" class="col-md-4 col-form-label text-md-right">Name</label>
                <div class="col-md-6">
                  <input
                    type="text"
                    class="form-control"
                    v-model="name"
                    id="name"
                    required
                    autofocus
                  >
                </div>
              </div>

              <div class="form-group row">
                <label for="email" class="col-md-4 col-form-label text-md-right">Email</label>
                <div class="col-md-6">
                  <input type="email" class="form-control" v-model="email" id="email" required>
                </div>
              </div>

              <div class="form-group row">
                <label for="password" class="col-md-4 col-form-label text-md-right">Password</label>
                <div class="col-md-6">
                  <input
                    type="password"
                    class="form-control"
                    v-model="password"
                    id="password"
                    required
                  >
                </div>
              </div>

              <div class="form-group row">
                <label
                  for="password-confirmation"
                  class="col-md-4 col-form-label text-md-right"
                >Confirm Password</label>
                <div class="col-md-6">
                  <input
                    type="password"
                    class="form-control"
                    v-model="password_confirmation"
                    id="password-confirm"
                    required
                  >
                </div>
              </div>

              <div class="form-group row mb-0">
                <div class="col-md-6 offset-md-4">
                  <button type="submit" class="btn btn-primary" @click="handleSubmit">Register</button>
                </div>
              </div>
            </form>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      name: "",
      email: "",
      password: "",
      password_confirmation: ""
    };
  },
  methods: {
    handleSubmit(e) {
      e.preventDefault();

      if (
        this.password === this.password_confirmation &&
        this.password.length > 0
      ) {
        axios
          .post("api/register", {
            name: this.name,
            email: this.email,
            password: this.password,
            c_password: this.password_confirmation
          })
          .then(response => {
            localStorage.setItem("user", response.data.success.name);
            localStorage.setItem("jwt", response.data.success.token);

            if (localStorage.getItem("jwt") != null) {
              this.$router.go("/board");
            }
          })
          .catch(err => {
            console.error(err);
          });
      } else {
        this.password = "";
        this.password_confirmation = "";

        return alert("Passwords do not match");
      }
    },
    beforeRouteEnter(to, from, next) {
      if (localStorage.getItem("jwt")) {
        return next("board");
      }

      next();
    }
  }
};
</script>

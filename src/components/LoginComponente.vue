<template>
  <div id="form">
    <div class="container my-3 border rounded bg-light" style="width: 50%">
      <div class="row">
        <div class="col-12 text-center my-2"><h3>Ingreso al Sistema</h3></div>
      </div>
      <div class="form-group my-1">
        <label for="email" class="cols-sm-2 control-label">Correo Electrónico</label>
        <div class="cols-sm-10">
          <div class="input-group">
            <span class="pt-2 px-3 input-group-addon bg-info">
              <i class="fa fa-envelope fa" aria-hidden="true"></i>
            </span>
            <input
              v-model="correo"
              type="text"
              class="form-control"
              placeholder="Ingrese el Correo Electrónico"
            />
          </div>
        </div>
      </div>
      <div class="form-group my-1">
        <label for="password" class="cols-sm-2 control-label">Contraseña</label>
        <div class="cols-sm-10">
          <div class="input-group">
            <span class="pt-2 px-3 input-group-addon bg-info">
              <i class="fa fa-lock fa-lg" aria-hidden="true"></i>
            </span>
            <input
              v-model="clave"
              type="password"
              class="form-control"
              placeholder="Ingrese la Contraseña"
            />
          </div>
        </div>
      </div>
      <div class="form-group my-2">
        <button type="button" class="btn btn-primary" @click="ingresar">
          Ingresar
        </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "loginForm",
  data: function () {
    return {
      correo: "",
      clave: "",
    };
  },
  props: ["firebase"],
  methods: {
    ingresar: function () {
      this.firebase
        .auth()
        .signInWithEmailAndPassword(this.correo, this.clave)
        .then((response) => {
          console.log("Ingreso correcto con User: " + response.user.email);
          this.$emit("ingresoCorrecto", response.user.email);
        })
        .catch((error) => {
          console.log("Error: " + error.code + " - " + error.message);
        });
    },
  },
};
</script>

<style>
#form {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}
</style>

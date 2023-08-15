<template>
  <div id='app'>
    <div id='login' v-if='!logon' >
      <loginComponente
        v-bind:firebase="firebase"
        v-on:ingresoCorrecto='ingresoCorrecto($event)'>
      </loginComponente>
    </div>
    <div id='inicio' class="container mx-auto p-2" v-else>
      
      <!-- Encabezado: lista de gastos + botonPlus -->
      <div class="container">
        <div class="row bg-dark">
          <div class="col-10 text-white text-center my-auto">
            <h1 class="mb-0">Lista de Gastos</h1>
          </div>
          <div class="col-2 bg-dark my-auto">
            <span
              v-bind:id="'agregar'"
              v-bind:class="['btn', 'btn-primary', boton]"
              style="font-size: 10px"
              v-on:click="butonPlus($event)"
            ></span>
          </div>
        </div>
      </div>

      <!--    Seccion informacion de usuario    -->
      <div class="container-fluid mt-1 rounded bg-light">
        <div class="row p-1" style="font-size:20px">
          <div class="col-md-3 col-sm-12"><b>Bienvenido:</b></div>
          <div class="col-md-9 col-sm-12">
            <i class="fa fa-user"></i> Usuario: {{idUsuario}}
          </div>
        </div>
      </div>

      <!-- Seccion para agregar/actualizar gastos ( mostrar/ocultar ) -->
      <div v-if="despliegue" key="despliegue" class="container border bg-light rounded text-primary lead p-2 my-2">
        <!-- Header contenido agregar/actualizar -->
        <div :class="['text-dark', 'my-1', colorHeader = actualizar ? 'bg-info' : 'bg-primary' ]">
          <h3>{{encabezado}} Gasto</h3>
        </div>
        <!-- Campos de ingreso gasto ( nombre - monto - tipo ) -->
        <div class="row justify-content-md-center my-2">
          <div class="col-sm-10 col-md-4 mx-auto my-auto">Nombre del gasto</div>
          <div class="col-sm-8 col-md-4 mx-auto my-auto">
            <input v-model="nombreGasto" class="text-primary" type="text" style="width:100%"/>
          </div>
        </div>
        <div class="row justify-content-md-center my-2">
          <div class="col-sm-10 col-md-4 mx-auto my-auto">Monto del gasto</div>
          <div class="col-sm-8 col-md-4 mx-auto my-auto">
            <input v-model="montoGasto" class="text-primary" type="text" style="width:100%"/>
          </div>
        </div>
        <div class="row justify-content-md-center my-2">
          <div class="col-sm-10 col-md-4 mx-auto my-auto">Tipo de gasto</div>
          <div class="col-sm-8 col-md-4 mx-auto my-auto">
            <input v-model="tipoGasto" class="text-primary" type="text" style="width:100%"/>
          </div>
        </div>
        <div class="row justify-content-md-center my-3">
          <div v-if="actualizar" class="col-6 col-md-3 mx-auto">
            <div id="actualizar" class="btn btn-info" v-on:click="manejarClick($event)" style="width:100%">Actualizar</div>
          </div>
          <div v-else class="col-6 col-md-3 mx-auto">
            <div id="agregar" class="btn btn-primary" v-on:click="manejarClick($event)" style="width:100%">Agregar</div>
          </div>
        </div>
      </div>

      <!-- filtros -->
      <div class="container my-1">
        <div class="row">
          <div class="col-sm-4 col-12 my-1">
            <button
              id="hogar" type="button" style="width:100%"
              v-bind:class="['btn', 'btn-light', 'btn-outline-secondary', mostrarHogar]"
              v-on:click="filtro($event)"
            >Hogar</button>
          </div>
          <div class="col-sm-4 col-12 my-1">
            <button
              id="trabajo" type="button" style="width:100%"
              v-bind:class="['btn', 'btn-light', 'btn-outline-secondary', mostrarTrabajo]"
              v-on:click="filtro($event)"
            >Trabajo</button>
          </div>
          <div class="col-sm-4 col-12 my-1">
            <button
              id="carros" type="button" style="width:100%"
              v-bind:class="['btn', 'btn-light', 'btn-outline-secondary', mostrarCarros]"
              v-on:click="filtro($event)"
            >Carros</button>
          </div>
        </div>
      </div>

      <!-- contenedor de la lista de gastos -->
      <div class="container text-center my-1">
        <div class="row lead border rounded bg-dark text-white text-center py-1">
          <div class="col-4">Nombre del gasto</div>
          <div class="col-3">Monto del gasto</div>
          <div class="col-3">Tipo de gasto</div>
          <div class="col-2">Accion</div>
        </div>
        <!-- lista de los gastos -->
        <gastosComponente
          v-for="(gasto,index) in gastos"
          v-show="filter==='filtro'"
          v-bind:gasto="gasto"
          v-bind:id="gasto.id"
          v-bind:indice="index"
          v-bind:key="index"
          v-on:eliminarGasto="eliminarGasto($event)"
          v-on:editarGasto="editarGasto($event)"
        ></gastosComponente>
        <!-- lista de filtros -->
        <filtrosComponente
          v-for="(gasto,index) in filtrados"
          v-show="gasto.tipo===filter"
          v-bind:gasto="gasto"
          v-bind:id="gasto.id"
          v-bind:indice="index"
          v-bind:key="filter+index"
          v-on:eliminarGasto="eliminarGasto($event)"
          v-on:editarGasto="editarGasto($event)"
        ></filtrosComponente>
      </div>

      <!-- Seccion total de gastos -->
      <div class="container">
        <div class="row border rounded bg-dark p-1" style="font-size:18px">
          <div class="col-6 text-left text-white"><b>Total</b></div>
          <div v-if="filter==='filtro'" class="col-6 text-center text-warning">
            <b>{{new Intl.NumberFormat("de-DE").format(totalGasto)}}</b>
          </div>
          <div v-else class="col-6 text-center text-info">
            <b>{{new Intl.NumberFormat("de-DE").format(cantidad)}}</b>
          </div>
        </div>
      </div>

      <!-- Seccion Salir del sistema => Reinician arreglos -->
      <div
        class="col-4 btn btn-danger mx-auto mt-4 mb-2"
        @click="salir()"
      >
        Salir
      </div>

    </div>
  </div>
</template>

<script>
import firebase from "firebase";
import "firebase/firestore";
import gastosComponente from "./components/GastosComponente.vue";
import filtrosComponente from "./components/FiltrosComponente.vue";
import loginComponente from "./components/LoginComponente.vue";

export default {
  name:'app',
  data: function() {
    return {
      encabezado:'Agregar',
      boton: "fa fa-plus",
      gastos: [],
      filtrados: [],
      nombreGasto: "",
      tipoGasto: "",
      montoGasto: "",
      coleccion: {},
      logon: false,
      firebase: "",
      idUsuario: "",
      db: "",
      totalGasto: 0.0,
      actualizar: false,
      despliegue: false,
      idEditar: "",
      montoAnt: 0.0,
      indice: 0,
      filter: "filtro",
      mostrarHogar: "",
      mostrarTrabajo: "",
      mostrarCarros: "",
      cantidad: 0.0
    };
  },
  components: {
    loginComponente,
    gastosComponente,
    filtrosComponente
  },
    beforeMount: function () {
    const firebaseConfig = {
      apiKey: "AIzaSyBR1umMRC-2dLGd2FQJ29rXzDtboO52sck",
      authDomain: "fir-firestore-b058c.firebaseapp.com",
      projectId: "fir-firestore-b058c",
      storageBucket: "fir-firestore-b058c.appspot.com",
      messagingSenderId: "66465685071",
      appId: "1:66465685071:web:40a878f476d85eeb77543a"
    };
    firebase.initializeApp(firebaseConfig);
    this.db = firebase.firestore();
    const settings = {timestampsInSnapshots: true};
    this.db.settings(settings);
    this.firebase = firebase
    console.log(this.db)
    console.log(this.firebase)
  },
  methods: {
    ingresoCorrecto: function(usuario) {
      console.log("User: " + usuario);
      this.idUsuario = usuario;
      this.logon = true;
      this.coleccion = this.db.collection("/usuarios/" + usuario + "/gastos");
      this.coleccion.get().then(gastos => {
        gastos.forEach(gasto => {
          this.gastos.push({
            id: gasto.id,
            monto: gasto.data().monto,
            nombre: gasto.data().nombre,
            tipo: gasto.data().tipo
          });
          this.totalGasto += parseFloat(gasto.data().monto);
        });
      });
      console.log(this.gastos)
    },
    butonPlus: function(event) {
      if (event.target.id === "agregar") {
        this.boton = "fa fa-minus";
        document.getElementById("agregar").setAttribute("id", "actualizar");
        this.despliegue = true;
        this.encabezado='Agregar'
      } else if (event.target.id === "actualizar") {
        this.boton = "fa fa-plus";
        document.getElementById("actualizar").setAttribute("id", "agregar");
        this.despliegue = false;
        this.actualizar = false;
      }
      this.montoGasto = "";
      this.nombreGasto = "";
      this.tipoGasto = "";
      this.filter = "filtro";
      this.mostrarHogar = "";
      this.mostrarTrabajo = "";
      this.mostrarCarros = "";
    },
    manejarClick: function(evento) {
      if (evento.target.id === "agregar") {
        // Se agregara los nuevos gastos
        const gastoData = {
          monto: this.montoGasto,
          nombre: this.nombreGasto,
          tipo: this.tipoGasto
        };
        this.totalGasto += parseFloat(this.montoGasto);
        this.coleccion
          .add(gastoData)
          .then(docReference => {
            this.gastos.unshift({
              id: docReference.id,
              monto: gastoData.monto,
              nombre: gastoData.nombre,
              tipo: gastoData.tipo
            });
          })
          .catch(Errro => {
            alert("No se pudo agregar el libro al sistema. Error: " + Errro.message);
          });
        this.montoGasto = "";
        this.nombreGasto = "";
        this.tipoGasto = "";
        this.despliegue = false;
        this.boton = "fa fa-plus";
        document.getElementById("actualizar").setAttribute("id", "agregar");
      }
      if (evento.target.id === "actualizar") {
        // Actualizando los gastos en firebase
        var Ref = this.db
          .collection("/usuarios/" + this.idUsuario + "/gastos")
          .doc(this.idEditar);
        Ref.update({
          monto: this.montoGasto,
          nombre: this.nombreGasto,
          tipo: this.tipoGasto
        });

        // Actualizando los valores de la lista de filtro
        console.log(this.montoAnt);
        this.cantidad -= this.montoAnt;
        console.log(this.cantidad);
        if (this.filter !== "filtro") {
          this.filtrados[this.indice].nombre = this.nombreGasto;
          this.filtrados[this.indice].tipo = this.tipoGasto;
          this.filtrados[this.indice].monto = this.montoGasto;
          this.cantidad += parseFloat(this.montoGasto);
        }

        // actualizando los valores de la lista
        this.totalGasto -= this.montoAnt;
        this.gastos[
          this.gastos.indexOf(
            this.gastos.find(element => element.id === this.idEditar)
          )
        ].nombre = this.nombreGasto;
        this.gastos[
          this.gastos.indexOf(
            this.gastos.find(element => element.id === this.idEditar)
          )
        ].tipo = this.tipoGasto;
        this.gastos[
          this.gastos.indexOf(
            this.gastos.find(element => element.id === this.idEditar)
          )
        ].monto = this.montoGasto;
        this.totalGasto += parseFloat(this.montoGasto);
        
        this.montoGasto = "";
        this.nombreGasto = "";
        this.tipoGasto = "";
        this.despliegue = false;
        this.actualizar = false;
        this.boton = "fa fa-plus";
        document.getElementById("actualizar").setAttribute("id", "agregar");
      }
    },
    editarGasto: function(cambio) {
      this.encabezado='Actualizar'
      // Modificar gasto
      this.actualizar = true;
      this.despliegue = true;
      this.boton = "fa fa-minus";
      if (!document.getElementById("actualizar")) {
        document.getElementById("agregar").setAttribute("id", "actualizar");
      }
      this.montoGasto = cambio.monto;
      this.nombreGasto = cambio.nombre;
      this.tipoGasto = cambio.tipo;
      this.idEditar = cambio.id;
      this.montoAnt = parseFloat(cambio.monto);
      this.indice = parseInt(cambio.indice);
    },
    eliminarGasto: function(gastoID) {
      console.log(gastoID)
      // Eliminar gastos
      // Eliminar gastos en la lista de filtros y la lista genera que es gastos
      if (this.filter !== "filtro") {
        this.cantidad -= parseFloat(gastoID.monto);
        this.totalGasto -= parseFloat(gastoID.monto);
        this.filtrados.splice(gastoID.indice, 1);
        this.coleccion.doc(gastoID.id).delete();
        this.gastos.splice(
          this.gastos.indexOf(this.gastos.find(element => element.id === gastoID.id)), 1
        )
      } else{
        this.totalGasto -= parseFloat(gastoID.monto);
        this.coleccion.doc(gastoID.id).delete();
        this.gastos.splice(gastoID.indice, 1);
      }
    },
    salir: function(){
      this.logon = false
      this.gastos = []
      this.totalGasto = 0.0
      this.cantidad = 0.0;
      console.log(this.gastos)
    },
    filtro: function(filt) {
      // filtros segun el tipo
      if (filt.target.id === "hogar") {
        if (this.mostrarHogar === "active") {
          this.mostrarHogar = "";
          this.filter = "filtro";
        } else {
          this.cantidad = 0.0;
          this.filtrados = [];
          this.gastos.forEach(gasto => {
            if (gasto.tipo === "Hogar" || gasto.tipo === "hogar") {
              this.filtrados.push({
                id: gasto.id,
                monto: gasto.monto,
                nombre: gasto.nombre,
                tipo: gasto.tipo
              });
              this.cantidad += parseFloat(gasto.monto);
            }
          });
          console.log(this.filtrados);
          console.log(this.cantidad)
          this.mostrarHogar = "active";
          this.mostrarTrabajo = "";
          this.mostrarCarros = "";
          this.filter = "hogar";
        }
      } else if (filt.target.id === "trabajo") {
        if (this.mostrarTrabajo === "active") {
          this.mostrarTrabajo = "";
          this.filter = "filtro";
        } else {
          this.cantidad = 0.0;
          this.filtrados = [];
          this.gastos.forEach(gasto => {
            if (gasto.tipo === "Trabajo" || gasto.tipo === "trabajo") {
              this.filtrados.push({
                id: gasto.id,
                monto: gasto.monto,
                nombre: gasto.nombre,
                tipo: gasto.tipo
              });
              this.cantidad += parseFloat(gasto.monto);
            }
          });
          console.log(this.filtrados);
          console.log(this.cantidad)
          this.mostrarHogar = "";
          this.mostrarTrabajo = "active";
          this.mostrarCarros = "";
          this.filter = "trabajo";
        }
      } else {
        if (this.mostrarCarros === "active") {
          this.mostrarCarros = "";
          this.filter = "filtro";
        } else {
          this.cantidad = 0.0;
          this.filtrados = [];
          this.gastos.forEach(gasto => {
            if (gasto.tipo === "Carros" || gasto.tipo === "carros") {
              this.filtrados.push({
                id: gasto.id,
                monto: gasto.monto,
                nombre: gasto.nombre,
                tipo: gasto.tipo
              });
              this.cantidad += parseFloat(gasto.monto);
            }
          });
          console.log(this.filtrados);
          console.log(this.cantidad)
          this.mostrarHogar = "";
          this.mostrarTrabajo = "";
          this.mostrarCarros = "active";
          this.filter = "carros";
        }
      }
    },
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
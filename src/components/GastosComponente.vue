<template>
  <div>
    <div class="row my-1 p-1 rounded bg-light text-primary">
      <div class="col-4 my-auto">{{gasto.nombre}}</div>
      <div class="col-3 my-auto">{{new Intl.NumberFormat("de-DE").format(gasto.monto)}}</div>
      <div class="col-3 my-auto">{{gasto.tipo}}</div>
      <div class="col-1 px-1">
        <div
          id="editar" class="btn btn-success btn-sm" style="width:100%"
          v-on:click="editar($event,{
            indice:indice, id:id, mostrar: true,
            monto: gasto.monto, tipo: gasto.tipo, nombre: gasto.nombre,
          })"
        ><i class="fa fa-edit" aria-hidden="true"></i></div>
      </div>
      <div class="col-1 px-1">
        <div
          id="eliminar" class="btn btn-danger btn-sm" style="width:100%"
          v-on:click="eliminar($event,{indice:indice, id:id, monto: gasto.monto})"
        ><i class="fa fa-trash" aria-hidden="true"></i></div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "gastosComponente",
  props: ["gasto", "id", "indice"],
  data: function() {
    return {
      test: "test",
      mostrar: false
    };
  },
  methods: {
    eliminar: function(evento, gastoID) {
      if (evento.target.id === "eliminar") {
        this.$emit("eliminarGasto", gastoID);
      }
    },
    editar: function(event, cambios) {
      console.log(event);
      this.$emit("editarGasto", cambios);
      this.mostrar = true;
    },
    agregarTarea: function(event) {
      console.log(event.mostrar);
      this.$emit("editarGasto", event);
      if (!event.mostrar) this.mostrar = false;
    }
  }
};
</script>

<style>
</style>

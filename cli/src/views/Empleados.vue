<template>
  <div class="container">
    <h1>-- EMPLEADOS -- </h1>

    <form @submit.prevent="actualizarEmpleado(editEmpleado)" v-if="editar">
      <h4> Editar datos del empleado</h4>
      <br>
      <div class="row">
        <div class="col-4">
          <input type="number" class="col-4 form-control form-control-sm mb-2" name="editDocumento" id="editDocumento" v-model="editEmpleado.documento" placeholder="numero de documento" required>
        </div>
        <div class="col-4">
          <input type="text" class="form-control form-control-sm mb-2" name="editNombres" id="editNombres" v-model="editEmpleado.nombres" placeholder="nombres completos" required>
        </div>
        <div class="col-4">
          <input type="text" class="form-control form-control-sm mb-2" name="editApellidos" id="editApellidos" v-model="editEmpleado.apellidos" placeholder="apellidos" required>
        </div>
        <div class="col">
          <input class="btn btn-outline-success" type="submit" name="actualizar" id="actualizar" value="actualizar"> 
        </div>
        <div class="col">
          <input class="btn btn-outline-warning" type="reset" name="editLimpiar" id="editLimpiar" value="limpiar"> 
        </div>
        <div class="col">
          <button class="btn btn-outline-primary" @click="editar=false" name="cancelar" id="cancelar" >CANCELAR</button> 
        </div>
      </div>
      <br>
      <b-alert
        :show="dismissCountDown"
        dismissible
        :variant="mensaje.color"
        @dismissed="dismissCountDown=0"
        @dismiss-count-down="countDownChanged"
        >
        {{mensaje.texto}}
      </b-alert>
    </form>

    <form @submit.prevent="crearEmpleado()" v-if="!editar">
      <h4>Agregar nuevo empleado</h4>
      <br>
      <div class="row">
        <div class="col-4">
          <input type="number" class="col-4 form-control form-control-sm mb-2" name="documento" id="documento" v-model="empleado.documento" placeholder="numero de documento" required>
        </div>
        <div class="col-4">
          <input type="text" class="form-control form-control-sm mb-2" name="nombres" id="nombres" v-model="empleado.nombres" placeholder="nombres completos" required>
        </div>
        <div class="col-4">
          <input type="text" class="form-control form-control-sm mb-2" name="apellidos" id="apellidos" v-model="empleado.apellidos" placeholder="apellidos" required>
        </div>
        <div class="col">
          <input class="btn btn-outline-success" type="submit" name="crear" id="crear" value="Crear"> 
        </div>
        <div class="col">
          <input class="btn btn-outline-warning" type="reset" name="limpiar" id="limpiar" value="limpiar"> 
        </div>
      </div>
      <br>
      <b-alert
        :show="dismissCountDown"
        dismissible
        :variant="mensaje.color"
        @dismissed="dismissCountDown=0"
        @dismiss-count-down="countDownChanged"
        >
        {{mensaje.texto}}
      </b-alert>
    </form>

    <table class="table table-striped mt-5" v-if="!editar">
      <thead>
        <tr>
          <th scope="col">Id</th>
          <th scope="col">Marca Temporal</th>
          <th scope="col">Documento</th>
          <th scope="col">Nombres</th>
          <th scope="col">Apellidos</th>
          <th scope="col">Acciones</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(item, index) in empleados" :key="index">
          <th scope="row">{{item._id}}</th>
          <td>{{item.marcaTemporal}}</td>
          <td>{{item.documento}}</td>
          <td>{{item.nombres}}</td>
          <td>{{item.apellidos}}</td>
          <td>
              <div>
              <b-button class="btn btn-danger m-1" @click="eliminarEmpleado(item._id)">ELIMINAR</b-button>
              <b-button class="btn btn-warning m-1" @click="editarEmpleado(item._id)">EDITAR</b-button>
              </div>
          </td>
        </tr>
        
      </tbody>
    </table>
  </div>
</template>

<script>
export default{
    data(){
        return{
          empleados: [],
          empleado: {documento:"",nombres:"",apellidos:""},
          editEmpleado: {},
          editar: false,

          mensaje: {color: 'success', texto: ''},
          dismissSecs: 5,
          dismissCountDown: 0
        }
    },

    created(){
        this.listarEmpleados();
    },

    methods:{
        listarEmpleados(){
          this.axios.get('/empleado').then(res=>{
              this.empleados=res.data;
          })
          .catch(e=>{
              console.log(e.response)
          })
        },

        crearEmpleado(){
          this.axios.post('/empleado-nuevo',this.empleado).then(res=>{
            this.empleados.push(res.data);
            this.empleado.documento="";
            this.empleado.nombres="";
            this.empleado.apellidos="";
            this.mensaje.color="success";
            this.mensaje.texto="Empleado creado";
            this.showAlert();
          }).catch(e=>{
            console.log(e.response);
          })
        },

        eliminarEmpleado(id){
          console.log("ID recibido: " + id)
          this.axios.delete(`/empleado/${id}`).then(res=>{
            const index = this.empleados.findIndex(item=> item._id===res.data._id);
            console.log('Index de elemento' + index);
            this.empleados.splice(index,1);
            this.mensaje.color="danger";
            this.mensaje.texto="Empleado eliminado ";
            this.showAlert();

          }).catch(e=>{
            console.log(e.response);
          })
        },

        editarEmpleado(id){
          this.editar=true;
          console.log("ID recibido: " + id)
          this.axios.get(`/empleado/${id}`).then(res=>{
            
            this.editEmpleado=res.data;

            this.mensaje.color="warning";
            this.mensaje.texto="Modificará los datos del empleado ";
            this.showAlert();

          }).catch(e=>{
            console.log(e.response);
          })
        },

        actualizarEmpleado(item){
          console.log("ID recibido: " + item._id)
          this.axios.put(`/empleado/${item._id}`,item).then(res=>{

            const index = this.empleados.findIndex(i=> i._id===res.data._id);
            console.log('Index de elemento' + index);
            this.empleados[index].documento=res.data.documento;
            this.empleados[index].nombres=res.data.nombres;
            this.empleados[index].apellidos=res.data.apellidos;
            this.editar=false;
            
            this.mensaje.color="danger";
            this.mensaje.texto="Empleado actualizado";
            this.showAlert();

          }).catch(e=>{
            console.log(e.response);
          })
        },

        countDownChanged(dismissCountDown) {
          this.dismissCountDown = dismissCountDown
        },
        showAlert() {
          this.dismissCountDown = this.dismissSecs
        }

    }
}
</script>

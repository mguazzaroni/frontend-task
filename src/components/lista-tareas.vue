<template>
    <div class="container">
        <h1 class="display-4 text-center">Lista de tareas</h1>
        <hr>
        <div class="row">
            <div class="col-lg-8 offset-lg-2">
                <div class="card mt-4">
                    <div class="card-body">
                        <div class="input-group">
                            <!--con la directiva v-model="tarea" enlazo el valor que toma el input con la variable tarea en la funcion data del script-->
                            <input v-model="tarea"
                            type="text" class="form-control form-control-lg" placeholder="Agregar Tarea">
                            <!--la directiva v-on:click llama a la funcion agregartarea(),que toma el valor de tarea y la pushea a un array de tareas-->
                            <button v-on:click="agragarTarea()"
                            class="btn btn-success" type="button">Agregar</button>
                        </div>
                        <br>
                        <!--La directiva v-if valida que haya tareas a realizar, si no hay, muestra un h5 con un mensaje-->
                        <div v-if="loading" class="text-center">
                            <div class="spinner-border text-success" role="status">
                                <span class="visually-hidden">Loading...</span>
                            </div>
                        </div>
                        <h5 v-if="listTareas.length == 0" class="text-center">No hay tareas pendientes</h5>
                        <ul v-if="!loading" class="list-group">
                            <!--Con la directiva v-for, recorro el array listTareas y muestro el nombre de la tarea-->
                            <li v-for="(tarea, index) of listTareas " :key="index"
                            class="list-group-item d-flex justify-content-between">
                            <!--Cuando toco el circulo, pasa a check, directiva v-bind:class, cuando el estado de la tarea cambia (editarTarea(tarea, index) => le paso la tarea a cambiar de estado) me va a cambiar el icono, si esta en false muestra circulo, si esta en true muestra check-->
                                <span v-on:click="editTarea(tarea, tarea.id)"
                                    v-bind:class="{'text-success' : tarea.state}">
                                    <i v-bind:class="[tarea.state ? 'fa-solid fa-circle-check' : 'fa-regular fa-circle cursor']"></i>                   
                                </span>
                                {{tarea.taskName}} <!--Aca muestro la iteracion del v-for de arriba, por cada tarea que tenga el array muestro el nombre-->
                                <span>
                                    <!--Cuando toco el basurero, llamo a la funcion deleteTarea, que la elimina-->
                                    <i class="fa-solid fa-trash text-danger cursor" v-on:click="deleteTarea(tarea.id)"></i>
                                </span>
                            </li>
                        </ul>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>

    import axios from "axios";
    const URL = "https://localhost:7132/api/tarea/";
    export default {
        //aca estoy usando optionsApi (revisar concepto y practicar)
        /*OptionsApi se basa en el uso de un objeto que contiene varias propiedades clave para el funcionamiento de los componentes Vue, como por ejemplo las propiedades props, data, computed, methods*/

        name: 'lista-tareas', //Establece el nombre del componente
        //data = devuelve un objeto con las variables que va a utilizar el componente
        data() {
            return {
                tarea: '',
                listTareas: [],
                loading: false
            }
        },
        //methods = contiene la lista de metodos que se van a utilizar en el componente
        methods: {
            //POST
            agragarTarea(){
                //Empiezo a cargar el loading
                this.loading = true;
                //creo el objeto tarea
                const tarea = {
                    taskName: this.tarea,
                    state: false
                }
                
                // this.listTareas.push(tarea);
                //Llamo al metodo POST de axios, le paso la ruta de la api, con el objeto tarea que cree arriba, como parametro
                axios.post(URL, tarea).then( res => {
                    console.log(res.status); //Si sale todo bien responde el 200 de la api
                    this.loading = false; //Corta el circulo de cargando
                    this.getTareas(); //Actualiza y muestra la nueva tarea
                }).catch( error => {
                    console.error(error); //Si falla muestra el mensaje de error que devuelve la api
                    this.loading = false;//Corta el circulo de cargando
                })
                this.tarea = ''; //Pone en blanco el campo de texto de nueva tarea
            },
            deleteTarea(id){
                this.loading = true;
                //this.listTareas.splice(index, 1)
                //Llamo al metodo delete de axios, le paso la ruta de la api, con el id como parametro
                axios.delete(URL + id).then( res => {
                    console.log(res.status); //Si sale todo bien muestra el 200 que devuelve la api
                    this.loading = false; // corta el circulo de cargando
                    this.getTareas();//Actualiza la pagina
                }).catch(error => {
                    console.error(error); // si falla muestra por consola la falla
                    this.loading = false; //Corta el circulo de cargando
                });
            },
            //PUT
            editTarea(tarea, id){
                // this.listTareas[index].estado = !tarea.estado
                this.loading = true;
                //Llamo al metodo put de axios, le paso la ruta de la api, el id y la tarea a modificar
                axios.put(URL + id, tarea).then(() =>{
                    this.getTareas();//Muestro las tareas de nuevo
                    this.loading = false; //Corto el circulo de carga
                }).catch(error => {
                    console.error(error); //Si falla muestro el error por consola
                    this.loading = false; //Corto el circulo de carga
                })
            },
            //GET
            getTareas(){
                this.loading = true;
                //Llamo al metodo get de axios, le paso la ruta de la api
                axios.get(URL).then(res => {
                    //Este res me va a devolver un objeto, que dentro de data, va a contener lo que devuelve la api, un array de tareas
                    console.log(res.status);//me devuelve un 200 si sale todo bien
                    this.listTareas = res.data; //Le asigno lo que me trae en data la peticion get de axios
                    this.loading = false;
                }).catch( () => {
                    this.loading = false;
                })
            }
        },
        //Ciclo de vida created, cuando se crea o levanta el componente hace lo que le digo
        created: function () { 
            this.getTareas();
         }
    }
</script>
<!--Aca se escriben los estilos, directamente en el componente-->
<style>
.cursor {
    cursor: pointer;
}

</style>
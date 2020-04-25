<template>
    <div >
        <div v-if="!loading">
            <img  class="rounded mx-auto d-block" :src="image" alt="loader" >
        </div>
        <div v-else >
        <button @click="createModal" class="btn btn-primary btn-block" >Add New Task</button>
        <table class="table" v-if="tasks">
            <thead>
                <tr>
                    <th>id</th>
                    <th>name</th>
                    <th>body</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for= "(task, index) in tasks">
                    <td>{{index + 1}}</td>
                    <td>{{task.name}}</td>
                    <td>{{task.body}}</td>
                    <td><button @click="updateModal(index)" class="btn btn-info">Edit</button></td>
                    <td> <button @click="deleteTask(index)"  class="btn btn-danger">Delete</button> </td>
                </tr>
            </tbody>
        </table>

        <!-- create Modal -->
<div class="modal fade" id="createModal" tabindex="-1" role="dialog" aria-labelledby="exampleModalLongTitle" aria-hidden="true">
  <div class="modal-dialog" role="document">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title" id="">Create Modal</h5>
        <button type="button" class="cloexampleModalLongTitlese" data-dismiss="modal" aria-label="Close">
          <span aria-hidden="true">&times;</span>
        </button>
      </div>
      <div class="modal-body">
          <div class="alert alert-danger" v-if="errors.length > 0 " >
              <ul>
                  <li v-for="error in errors"> {{error}}</li>
              </ul>
          </div>

          <div class="form-group" >
                <label for="name" >Name </label>
                <input v-model="task.name" type="text" id="name" class="from-control" >
          </div>
          <div class="form-group" >
                <label for="description" >Description </label>
                <input v-model="task.body" type="text" id="description" class="from-control" >
          </div>
      </div>
      
      <div class="modal-footer">
        <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
        <button @click="createTask" type="button" class="btn btn-primary">Save changes</button>
      </div>
    </div>
  </div>
</div>

<div class="modal fade" id="update-modal" tabindex="-1" role="dialog" aria-labelledby="exampleModalLongTitle" aria-hidden="true">
  <div class="modal-dialog" role="document">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title" id="">update</h5>
        <button type="button" class="cloexampleModalLongTitlese" data-dismiss="modal" aria-label="Close">
          <span aria-hidden="true">&times;</span>
        </button>
      </div>
      <div class="modal-body">
          <div class="alert alert-danger" v-if="errors.length > 0 " >
              <ul>
                  <li v-for="error in errors"> {{error}}</li>
              </ul>
          </div>

          <div class="form-group" >
                <label for="name" >Name </label>
                <input v-model="newUpdateTask.name" type="text" id="name" class="from-control" >
          </div>
          <div class="form-group" >
                <label for="description" >Description </label>
                <input v-model="newUpdateTask.body" type="text" id="description" class="from-control" >
          </div>
      </div>
      
      <div class="modal-footer">
        <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
        <button @click="updateTask" type="button" class="btn btn-primary">Save changes</button>
      </div>
    </div>
  </div>
</div>

  </div>
    </div>
</template>

<script>
    export default {
        data(){
            return{
                task:{
                    name:'',
                    body:'',
                },

                tasks:[],
                uri: 'http://localhost:8000/tasks/',
                errors:[],
                newUpdateTask: [],
                image: 'images/loader1.gif',
                loading: false,
           }
        },
        methods:{
            createModal(){
                $("#createModal").modal("show");
            },
            updateModal(index){
                this.errors = [];
                $("#update-modal").modal("show");
                this.newUpdateTask = this.tasks[index];

            },

            createTask(){
                console.log(this.task.name);
                axios.post(this.uri, {name:this.task.name, body:this.task.body }).then(response=>{
                    this.tasks.push(response.data.task);
                        this.resetData();
                    $("#createModal").modal('hide');
                }).catch(error=>{
                    this.errors = [];
                    if(error.response.data.errors.name){
                        this.errors.push(error.response.data.errors.name[0]);
                    }
                    if(error.response.data.errors.body){
                        this.errors.push(error.response.data.errors.body[0]);
                    }
                });
            },
            updateTask(){
                console.log(this.newUpdateTask.name);
                axios.patch(this.uri + this.newUpdateTask.id, {
                    name: this.newUpdateTask.name,
                    body: this.newUpdateTask.body,
                }).then(response => {
                    $("#update-modal").modal('hide');

                }).catch(error=>{
                                        // this.errors = [];
                    if(error.response.data.errors.name){
                        this.errors.push(error.response.data.errors.name[0]);
                    }
                    if(error.response.data.errors.body){
                        this.errors.push(error.response.data.errors.body[0]);
                    }
                });
            },
            deleteTask(index){
                let confirmBox = confirm('Do you want to delete this?');
                if(confirmBox == true){
                    axios.delete(this.uri + this.tasks[index].id)
                    .then(response=>{
                        this.$delete(this.tasks, index);
                    }).catch(error=> {
                        console.log('could not delete');
                    });
                }

            },
            resetData(){
                this.task.name = '';
                this.task.body = '';
            },

            loadTasks(){
                axios.get(this.uri).then(response=> {
                    this.tasks = response.data.tasks
                    this.loading = true;
                });
            },

        },
        mounted() {
            this.loadTasks();
        }
    }
</script>

<template>
    <div class="container">
      <div class="row mt-5" v-if="$gate.isAdminOrAuthor()">
        <div class="col-md-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">User Table</h3>

                <div class="card-tools">
                    <button class="btn btn-success" @click="modalAdd"> 
                      <i class="fas fa-user-plus fa-fw"></i>
                      Add New
                      </button>
                </div>

              </div>
              <!-- /.card-header -->
              <div class="card-body table-responsive p-0">
                <table class="table table-hover">
                  <tbody>
                    <tr>
                    <th>ID</th>
                    <th>Name</th>
                    <th>Email</th>
                    <th>Type</th>
                    <th>Registered At</th>
                    <th>Modify</th>
                  </tr>

                  <tr v-for="user in users.data" :key="user.id" >
                    <td>{{user.id}}</td>
                    <td>{{user.name}}</td>
                    <td>{{user.email}}</td>
                    <td>{{user.type | upText}}</td>
                    <td>{{user.created_at | myDate}}</td>
                    <td>
                      <a href="#" @click="modalEdit(user)">
                        <i class="fas fa-edit blue"></i>
                      </a>
                      |
                      <a href="#" @click="deleteUser(user.id)">
                        <i class="fas fa-trash red"></i>
                      </a>

                    </td>
                  </tr>
                </tbody></table>
              </div>
              <!-- /.card-body -->
              <div class="card-footer">
                <pagination :data="users" @pagination-change-page="getResults"></pagination>
              </div>
            </div>
            <!-- /.card -->
          </div>
        </div>


  <div v-if="!$gate.isAdminOrAuthor()">
    <not-found> </not-found>
  </div>






<!-- Modal -->
<div class="modal fade" id="addNew" tabindex="-1" role="dialog" aria-labelledby="addNew" aria-hidden="true">
  <div class="modal-dialog modal-dialog-centered" role="document">
    <div class="modal-content">
      <div class="modal-header">
        <h5 v-show="editmode" class="modal-title" id="exampleModalLabel">Update User</h5>
        <h5 v-show="!editmode" class="modal-title" id="exampleModalLabel">Add User</h5>
        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
          <span aria-hidden="true">&times;</span>
        </button>
      </div>

      <form @submit.prevent="editmode ? updateUser() : createUser()">
        <div class="modal-body">
          
          <!-- //form -->

          <div class="form-group">
              <input v-model="form.name" type="text" name="name" id="name"
                placeholder="Name"
                class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
              <has-error :form="form" field="name"></has-error>
          </div>

          <div class="form-group">
              <input v-model="form.email" type="email" name="email" id="email"
                placeholder="Email"
                class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
              <has-error :form="form" field="email"></has-error>
          </div>

          <div class="form-group">
              <textarea v-model="form.bio" name="bio" id="bio"
                placeholder="Short Bio (Optional)"
                class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }"></textarea>
              <has-error :form="form" field="bio"></has-error>
          </div>

          <div class="form-group">
              <select name="type" v-model="form.type" id="type" class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
                  <option value=""> Select User Role</option>
                  <option value="admin">Admin</option>
                  <option value="user">Standard User</option>
                  <option value="author">Author</option>
              </select>
              <has-error :form="form" field="name"></has-error>
          </div>

          <div class="form-group">
              <input v-model="form.password" type="password" name="password" id="password"
                placeholder="Enter Password"
                class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
              <has-error :form="form" field="password"></has-error>
          </div>
          


        <!-- END OF MODAL BODY -->
        </div>


        <div class="modal-footer">
          <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
          <button v-show="editmode" type="submit" class="btn btn-success">Update</button>
          <button v-show="!editmode" type="submit" class="btn btn-primary">Create</button>
        </div>
            
      </form>

    </div>
  </div>
</div>


    </div>
</template>

<script>
    export default {
      data() {
        return{
          editmode: false,
          users : {},
          form: new Form({
            id: '',
            name: '',
            email: '',
            password: '',
            type: '',
            bio: '',
            photo: '',
          })
        } 
      },
      methods:{
        // pagination results
          getResults(page = 1) {
          axios.get('api/user?page=' + page)
            .then(response => {
              this.users = response.data;
            });
        },
        // edit users data
        updateUser(id){
          this.$Progress.start();
          this.form.put('api/user/'+this.form.id)
            .then(() => {
              $('#addNew').modal('hide')
              //success
               Swal.fire(
                  'Updated!',
                  'Info has been updated.',
                  'success'
                )
                this.$Progress.finish();
                Fire.$emit('AfterCreate');
            })
            .catch(() => {
              this.$Progress.fail();
            });
        },
        // show add new user modal
        modalEdit(user){
          this.editmode = true;
          this.form.reset();
           $('#addNew').modal('show');
           this.form.fill(user);
        },
        modalAdd(){
          this.editmode = false;
          this.form.reset();
           $('#addNew').modal('show');
        },
        //delete users data
        deleteUser(id){
          Swal.fire({
            title: 'Are you sure?',
            text: "You won't be able to revert this!",
            type: 'warning',
            showCancelButton: true,
            confirmButtonColor: '#3085d6',
            cancelButtonColor: '#d33',
            confirmButtonText: 'Yes, delete it!'
          }).then((result) => {
            //send request
            if (result.value) {
                this.form.delete('api/user/'+id)
                .then(() => {
                  Swal.fire(
                    'Deleted!',
                    'Your file has been deleted.',
                    'success'
                  )
                  Fire.$emit('AfterCreate');
                }).catch(() => {
                  Swal.fire('Failed','Something wrong in deleting file', 'Warning');
                });
              }
            })

        },

        //load users data
        showUsers(){
          if(this.$gate.isAdminOrAuthor()){
           axios.get('api/user').then(({data}) => (this.users = data));
          }
        },

        //create user
        createUser(){
          this.$Progress.start();

          this.form.post('api/user')
          .then(()=> {
            Fire.$emit('AfterCreate');
            $('#addNew').modal('hide')

            toast.fire({
              type: 'success',
              title: 'User Created successfully'
            })

            this.$Progress.finish();
          })

          .catch(()=>{

          })


        }
      },
        created() {
            Fire.$on('searching', () => {
              let query = this.$parent.search;
              axios.get('api/findUser?q=' + query)
              .then((data) => {
                this.users = data.data
              })
              .catch(()=> {

              })
            })
            this.showUsers();
            Fire.$on('AfterCreate', () => {
              this.showUsers();
            });
            //setInterval(() => this.showUsers(), 3000);
        }
    }
</script>

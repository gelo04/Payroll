<template>
    <div class="container">
        <div class="row">
            <div class="col-md-12">
                <div class="card">
                    <div class="card-header">
                        <h3 class="card-title">Users Table</h3>

                        <div class="card-tools">
                            <button class="btn btn-success" title="Add New" @click="NewModal">
                                <i class="fas fa-user-plus"></i>
                            </button>
                        </div>
                    </div>
                    <!-- /.card-header -->
                    <div class="card-body table-responsive p-0">
                        <table class="table table-hover">
                        <thead>
                            <tr>
                                <th>ID</th>
                                <th>Name</th>
                                <th>Email</th>
                                <th>Type</th>
                                <th>Registration Date</th>
                                <th>Modify</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="user in users" :key="user.id">
                                <td>{{ user.id }}</td>
                                <td>{{ user.name }}</td>
                                <td>{{ user.email }}</td>
                                <td>{{ user.type | upText }}</td>
                                <td>{{ user.created_at | myDate}}</td>
                                <td>
                                    <a href="#" title="Edit" @click="EditModal(user)">
                                        <i class="fas fa-edit blue"></i>
                                    </a>
                                    <span style="margin-left:5px; margin-right:5px">|</span>
                                    <a href="#" title="Delete" @click="DeleteUser(user.id)">
                                        <i class="fas fa-trash red"></i>
                                    </a>
                                </td>
                            </tr>
                        </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </div>

        <!-- Modal -->
        <div class="modal fade" id="AddModal" tabindex="-1" role="dialog" aria-labelledby="AddModalLabel" aria-hidden="true">
            <div class="modal-dialog modal-dialog-centered modal-md" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title" v-show="!editmode" id="AddModalLabel">Add New User</h5>
                        <h5 class="modal-title" v-show="editmode" id="AddModalLabel">Update User's Information</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                        <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <form @submit.prevent="editmode ? UpdateUser() : CreateUser()">
                        <div class="modal-body">
                            <div class="card card-primary">
                                <div class="card-body">

                                    <div class="form-group">
                                        <label for="name">Name</label>
                                        <input v-model="form.name" name="name" type="text" class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                                        <has-error :form="form" field="name"></has-error>
                                    </div>

                                    <div class="form-group">
                                        <label for="email">Email Address</label>
                                        <input v-model="form.email" name="email" type="email" class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                                        <has-error :form="form" field="email"></has-error>
                                    </div>

                                    <div class="form-group">
                                        <label for="bio">Bio</label>
                                        <textarea v-model="form.bio" name="bio" id="bio" placeholder="Short bio for user (Optional)"
                                        class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }"></textarea>
                                        <has-error :form="form" field="bio"></has-error>
                                    </div>

                                    <div class="form-group">
                                        <label for="type">User Type</label>
                                        <select name="type" v-model="form.type" id="type" class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
                                            <option value="">Select User Role</option>
                                            <option value="admin">Admin</option>
                                            <option value="user">Standard User</option>
                                            <option value="author">Author</option>
                                        </select>
                                        <has-error :form="form" field="type"></has-error>
                                    </div>

                                    <div class="form-group">
                                        <label for="password">Password</label>
                                        <input v-model="form.password" name="password" type="password" class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                                        <has-error :form="form" field="password"></has-error>
                                    </div>

                                </div>
                                <!-- /.card-body -->
                            </div>
                        </div>
                        <div class="modal-footer">
                            <button type="button" class="btn btn-danger " data-dismiss="modal">Close</button>
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
    //inserting data using api
    import Form from 'vform';
    export default {
        data() {
            return {
                editmode: true,
                users : {},
                form: new Form({
                    id: '',
                    name : '',
                    email : '',
                    password : '',
                    type : '',
                    bio : '',
                    photo : ''
                })
            }
        },
        methods:{
            NewModal(){
                this.editmode = false;
                this.form.reset();
                $('#AddModal').modal('show');
            },

            EditModal(user){
                this.editmode = true;
                this.form.reset();
                $('#AddModal').modal('show');
                this.form.fill(user);
            },

            LoadUsers(){
                axios.get("api/user").then( ({ data }) => (this.users = data.data));
            },

            CreateUser(){
                this.$Progress.start();
                this.form.post('api/user')
                .then(() => {
                    Fire.$emit('AfterCreate');
                    $('#AddModal').modal('hide')

                    toast.fire({
                        type: 'success',
                        title: 'User Created Successfully'
                    })

                    this.$Progress.finish();
                })
                .catch(() => {
                    this.$Progress.fail();
                })
            },

            UpdateUser(){
                this.$Progress.start();
                this.form.put('api/user/'+this.form.id)
                .then(() => {
                    Fire.$emit('AfterCreate');
                    $('#AddModal').modal('hide');
                    toast.fire({
                        type: 'success',
                        title: 'User Updated Successfully'
                    })
                    this.$$Progress.finish();
                }).catch(() => {
                    this.$Progress.fail();
                });
            },

            DeleteUser($id){
                swal.fire({
                    title: 'Are you Sure?',
                    text: 'You won\'t be able to revert this',
                    type: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#0e25f0',
                    cancelButtonColor: '#f50c0c',
                    confirmButtonText: 'Yes, delete it!',
                }).then(( result ) => {
                    if (result.value){
                        this.form.delete('api/user/'+$id).then(() => {
                            // swal.fire(
                            //     'Deleted!',
                            //     'Your file has been deleted',
                            //     'success'
                            // )
                            toast.fire({
                                type: 'success',
                                title: 'User Deleted Successfully'
                            })
                            Fire.$emit('AfterCreate');
                        }).catch(() => {
                            swal.fire("Failed!", "There was something wrong.", "warning");
                        });
                    }
                })
            }
        },

        created() {
            this.LoadUsers();
            Fire.$on('AfterCreate', () => {
                this.LoadUsers();
            });
        }
    }
</script>

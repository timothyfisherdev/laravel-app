<template>
    <div>
        <div class="card">
            <div class="card-header">
                <h3 class="card-title mb-0">Users Table</h3>

                <div class="card-tools">
                    <button type="button" class="btn btn-success" @click="openCreateModal">
                        Create User
                        <i class="fas fa-user-plus fa-fw"></i>
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
                            <th>Created</th>
                            <th>Updated</th>
                            <th>Modify</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="(user, index) in users" :key="user.id">
                            <td>{{ user.id }}</td>
                            <td>{{ user.name }}</td>
                            <td>{{ user.email }}</td>
                            <td>{{ user.type | ucfirst }}</td>
                            <td>{{ user.created_at | moment('MMMM Do, YYYY, h:mm:a') }}</td>
                            <td>{{ user.updated_at | moment('MMMM Do, YYYY, h:mm:a') }}</td>
                            <td>
                                <button type="button" class="btn" @click="openEditModal(user)">
                                    <i class="fa fa-edit text-primary"></i>
                                </button>
                                /
                                <button type="button" class="btn" @click="deleteUser(user.id, index)">
                                    <i class="fa fa-trash text-danger"></i>
                                </button>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>
            <!-- /.card-body -->
        </div>

        <div class="modal fade" id="userDataModal" tabindex="-1" role="dialog" aria-labelledby="userDataModalLabel" aria-hidden="true">
            <div class="modal-dialog modal-dialog-centered" role="document">
                <div class="modal-content">
                    <form @submit.prevent="(userDataModalMode === 'create') ? createUser() : updateUser()">
                        <div class="modal-header">
                            <h5 class="modal-title" id="userDataModalLabel">{{ (userDataModalMode === 'create') ? 'Create' : 'Edit' }} User</h5>
                            <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                                <span aria-hidden="true">&times;</span>
                            </button>
                        </div>
                        <div class="modal-body">
                            <div class="form-group">
                                <input 
                                    type="text" 
                                    name="name" 
                                    class="form-control" 
                                    placeholder="Name" 
                                    v-model="form.name" 
                                    :class="{ 'is-invalid': form.errors.has('name') }"
                                />
                                <has-error :form="form" field="name"></has-error>
                            </div>

                            <div class="form-group">
                                <input 
                                    type="email" 
                                    name="email" 
                                    class="form-control" 
                                    placeholder="Email" 
                                    v-model="form.email" 
                                    :class="{ 'is-invalid': form.errors.has('email') }"
                                />
                                <has-error :form="form" field="email"></has-error>
                            </div>

                            <div class="form-group">
                                <textarea 
                                    id="bio"
                                    name="bio" 
                                    class="form-control" 
                                    rows="4"
                                    placeholder="Short bio (optional)" 
                                    v-model="form.bio" 
                                    :class="{ 'is-invalid': form.errors.has('bio') }"
                                ></textarea>
                                <has-error :form="form" field="bio"></has-error>
                            </div>

                            <div class="form-group">
                                <select 
                                    id="type"
                                    name="type" 
                                    class="form-control" 
                                    v-model="form.type" 
                                    :class="{ 'is-invalid': form.errors.has('type') }"
                                >
                                    <option value="">Select User Role</option>
                                    <option value="admin">Admin</option>
                                    <option value="user">Standard User</option>
                                    <option value="author">Author</option>
                                </select>
                                <has-error :form="form" field="type"></has-error>
                            </div>

                            <div class="form-group">
                                <input 
                                    type="password" 
                                    name="password" 
                                    class="form-control" 
                                    placeholder="Password" 
                                    v-model="form.password" 
                                    :class="{ 'is-invalid': form.errors.has('password') }"
                                />
                                <has-error :form="form" field="password"></has-error>
                            </div>

                            <div class="form-group">
                                <input 
                                    type="text" 
                                    name="photo" 
                                    class="form-control" 
                                    placeholder="Photo" 
                                    v-model="form.photo" 
                                    :class="{ 'is-invalid': form.errors.has('photo') }"
                                />
                                <has-error :form="form" field="photo"></has-error>
                            </div>
                        </div>
                        <div class="modal-footer">
                            <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
                            <button type="submit" class="btn btn-primary">
                                {{ (userDataModalMode === 'create') ? 'Create' : 'Update' }}
                            </button>
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
            return {
                users: {},
                form: new Form({
                    id: '',
                    name: '',
                    email: '',
                    password: '',
                    type: '',
                    bio: '',
                    photo: ''
                }),
                userDataModalMode: 'create'
            }
        },
        computed: {
            userDataModal: () => {
                return $('#userDataModal');
            }
        },
        methods: {
            openCreateModal() {
                this.userDataModalMode = 'create';
                this.form.reset();
                this.userDataModal.modal('show');
            },
            openEditModal(user) {
                this.userDataModalMode = 'edit';
                this.form.fill(user);
                this.userDataModal.modal('show');
            },
            loadUsers() {
                axios.get('/api/users').then(({ data: { data } }) => (this.users = data));
            },
            createUser() {
                this.$Progress.start();

                this.form.post('/api/users')
                    .then(({ data }) => {
                        this.users.unshift(data);

                        this.userDataModal.modal('hide');

                        Toast.fire({
                            type: 'success',
                            title: 'User created successfully'
                        });

                        this.$Progress.finish();
                    })
                    .catch(() => {
                        this.$Progress.fail();
                    });
            },
            deleteUser(id, index) {
                Swal.fire({
                    title: 'Are you sure?',
                    text: "You won't be able to revert this!",
                    type: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Yes, delete it!'
                }).then((result) => {
                    if (result.value) {
                        this.$Progress.start();

                        this.form.delete(`/api/users/${id}`)
                            .then(() => {
                                this.users.splice(index, 1);

                                Swal.fire(
                                    'Deleted!',
                                    'The user has been deleted.',
                                    'success'
                                );

                                this.$Progress.finish();
                            })
                            .catch(() => {
                                this.$Progress.fail();
                            });
                    }
                })
            },
            updateUser() {
                this.$Progress.start();

                this.form.patch(`/api/users/${this.form.id}`)
                    .then(({ data }) => {
                        Vue.set(this.users, this.users.map(user => user.id).indexOf(data.id), data);
                        //Vue.set(this.users, this.currentlyEditingIndex, data);

                        this.userDataModal.modal('hide');

                        Toast.fire({
                            type: 'success',
                            title: 'User updated successfully'
                        });

                        this.$Progress.finish();
                    })
                    .catch(() => {
                        this.$Progress.fail();
                    });
            }
        },
        created() {
            this.loadUsers();
        }
    }
</script>

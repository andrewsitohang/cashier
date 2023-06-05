<template>
    <v-row class="frame-content">
        <v-col cols="10" offset="1">
            <v-card class="my-3">
                <v-toolbar color="primary" dark>
                    Users
                    <v-spacer />
                    <v-text-field v-model="search" append-icon="mdi-magnify" label="Search" single-line hide-details>
                    </v-text-field>
                </v-toolbar>
                <v-card-text>
                    <v-alert v-if="alert.show" :type="alert.type">{{ alert.message }}</v-alert>
                    <div class="d-flex mb-4">
                        <v-breadcrumbs :items="breadcrumbs" class="pa-0" />
                        <v-spacer />
                        <v-btn to="/users/create" color="primary" elevation="3" small>
                            User <v-icon>mdi-plus-circle</v-icon>
                        </v-btn>
                    </div>
                    <v-data-table :headers="headers" :item-per-page="10" :server-items-length="totalData" :items="users"
                        :options.sync="options" :search.sync="search" :loading="loading" :footer-props="{
                            itemsPerPageOptions: [10, 20, 30, 40, 50]
                        }">
                        <template v-slot:top>
                            <v-dialog v-model="dialogDelete" max-width="400px">
                                <v-card>
                                    <v-card-title class="headline">Anda yakin mau hapus data {{ itemDelete.fullname }}
                                        ?</v-card-title>
                                    <v-card-actions>
                                        <v-spacer />
                                        <v-btn color="blue darken-1" text @click="closeDelete">Cancel</v-btn>
                                        <v-btn color="blue darken-1" text
                                            @click="deleteConfirm(itemDelete._id)">Delete</v-btn>
                                    </v-card-actions>
                                </v-card>
                            </v-dialog>
                        </template>
                        <template v-slot:item.actions="{ item }">
                            <v-btn :to="`/users/edit/${item._id}`" icon><v-icon small>mdi-pencil</v-icon></v-btn>
                            <v-btn @click="deleteItem(item)" icon><v-icon small>mdi-delete</v-icon></v-btn>
                        </template>
                    </v-data-table>
                </v-card-text>
            </v-card>
        </v-col>
    </v-row>
</template>

<script>
export default {
    head: {
        title: 'Users'
    },
    middleware: ['authenticated'],
    data() {
        return {
            alert: {
                show: false,
                message: '',
                type: ''
            },
            dialogDelete: false,
            itemDelete: false,
            search: '',
            options: {},
            loading: false,
            totalData: 0,
            users: [],
            headers: [
                { text: '#', value: 'row', sortable: false },
                { text: 'Fullname', value: 'fullname', sortable: false },
                { text: 'Email', value: 'email', sortable: false },
                { text: 'Role', value: 'role', sortable: false },
                { text: 'Actions', value: 'actions', sortable: false },
            ],
            breadcrumbs: [
                {
                    text: 'Users',
                    disabled: true,
                    href: '/users'
                }
            ]
        }
    },
    methods: {
        fetchUser() {
            const { page, itemsPerPage } = this.options
            this.loading = true
            this.$axios.$get(`/users?page=${page}&limit=${itemsPerPage}&search=${this.search}`)
                .then(response => {
                    this.loading = false
                    this.users = response.users.docs
                    this.totalData = response.users.totalDocs
                    // let startItem = (page - 1) * itemsPerPage + 1
                    let startItem = response.users.pagingCounter
                    this.users.map(user => user.row = startItem++)
                })
                .catch(err => {
                    this.loading = false
                    console.log(err)
                })
        },
        deleteConfirm(id) {
            this.$axios.$delete(`/users/${id}`)
                .then(response => {
                    // proses hapus data

                    this.users = this.users.filter(user => user._id != id)
                    this.closeDelete()
                    let params = {
                        message: 'DELETE_SUCCESS',
                        fullname: this.itemDelete.fullname
                    }
                    this.showAlert(params)
                })
                .catch(err => {
                    console.log(err)
                    this.closeDelete()
                })
        },
        deleteItem(item) {
            this.itemDelete = item
            this.dialogDelete = true
        },
        closeDelete() {
            this.dialogDelete = false
        },
        showAlert(params) {
            if (params.message == 'UPDATE_SUCCESS') {
                this.alert.show = true
                this.alert.type = 'success'
                this.alert.message = this.$t(params.message, {
                    title: params.fullname
                })
            } else if (params.message == 'ID_NOT_FOUND') {
                this.alert.show = true
                this.alert.type = 'error'
                this.alert.message = this.$t(params.message)
            } else if (params.message == 'CREATE_SUCCESS') {
                this.alert.show = true
                this.alert.type = 'success'
                this.alert.message = this.$t(params.message, {
                    title: params.fullname
                })
            } else if (params.message == 'DELETE_SUCCESS') {
                this.alert.show = true
                this.alert.type = 'success'
                this.alert.message = this.$t(params.message, {
                    title: params.fullname
                })
            }
        }
    },
    watch: {
        search: {
            handler() {
                this.fetchUser()
            },
            deep: true
        },
        options: {
            handler() {
                this.fetchUser()
            }
        },
    },
    mounted() {
        this.showAlert(this.$route.params)
    }
}
</script>
<template>
    <v-row>
        <v-col cols="10" offset="1">
            <v-card class="mb-2">
                <v-toolbar color="primary" class="white--text">Edit User</v-toolbar>
                <v-card-text>
                    <v-breadcrumbs :items="breadcrumbs" class="pa-0" />
                    <v-form ref="form">
                        <v-text-field name="fullname" label="Full Name" type="text" :rules="rules.fullname"
                            v-model="form.fullname" />
                        <v-text-field name="email" label="Email" type="email" :rules="rules.email" v-model="form.email"
                            @keydown="checkEmail" />
                        <v-text-field name="password" label="Password" type="password" :rules="rules.password"
                            v-model="form.password" />
                        <v-text-field name="retype_password" label="Re-Password" :rules="rules.retype_password"
                            type="password" v-model="form.retype_password" />
                        <v-select v-model="form.role" :items="roles" label="Role"></v-select>
                    </v-form>
                </v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn @click="onSubmit" color="primary" :loading="isDisable">Update</v-btn>
                </v-card-actions>
            </v-card>
        </v-col>
    </v-row>
</template>

<script>
export default ({
    head: {
        title: 'Edit User'
    },
    middleware: ['authenticated'],
    async asyncData({ params }) {
        return {
            id: params.id
        }
    },
    data() {
        return {
            breadcrumbs: [
                { text: 'Users', disabled: false, to: '/users', exact: true },
                { text: 'Edit', disabled: true }
            ],
            emailExist: false,
            isDisable: false,
            roles: ['employee', 'cashier', 'admin'],
            form: {
                fullname: '',
                email: '',
                password: '',
                retype_password: '',
                role: ''
            },
            rules: {
                fullname: [
                    v => !!v || this.$t('FIELD_REQUIRED', { field: 'Nama Lengkap' }),
                ],
                email: [
                    v => !!v || this.$t('FIELD_REQUIRED', { field: 'Email' }),
                    v => /.+@.+\..+/.test(v) || this.$t('EMAIL_INVALID'),
                    v => !this.emailExist || this.$t('EMAIL_EXIST')
                ],
                password: [
                    // v => v.length == 0 || !!v || this.$t('FIELD_REQUIRED', { field: 'Password' }),
                    v => v.length == 0 || (v && v.length >= 6) || this.$t('FIELD_MIN_LENGTH', { field: 'Password', length: 6 })
                ],
                retype_password: [
                    v => v === this.form.password || this.$t('FIELD_CONFIRM', { field: 'Password', confirm: 'Re-Password' })
                ],
                role: [
                    v => !!v || this.$t('FIELD_REQUIRED', { field: 'Role' }),
                ],
            }
        }
    },
    methods: {
        checkEmail() {
            this.emailExist = false;
        },
        fetchData() {
            this.$axios.$get(`/users/${this.id}`)
                .then(response => {
                    this.form.fullname = response.user.fullname;
                    this.form.email = response.user.email;
                    this.form.role = response.user.role;
                })
                .catch((error) => {
                    // redirect to user page
                    this.$router.push({
                        name: 'users___' + this.$i18n.locale,
                        params: { message: 'ID_NOT_FOUND' }
                    })
                })
        },
        onSubmit() {
            if (this.$refs.form.validate()) {
                this.isDisable = true
                this.$axios.$put(`/users/${this.id}`, this.form)
                    .then(response => {
                        this.isDisable = false

                        // redirect to user page
                        this.$router.push({
                            name: 'users___' + this.$i18n.locale,
                            params: { message: 'UPDATE_SUCCESS', fullname: this.form.fullname }
                        });
                    })
                    .catch((error) => {
                        if (error.response.data.message == "EMAIL_EXIST") {
                            this.emailExist = true
                            this.$refs.form.validate()
                        }
                        this.isDisable = false
                    })
            }
        }
    },
    mounted() {
        this.fetchData()
    }
})
</script>
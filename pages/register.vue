<template>
    <v-row>
        <v-col cols="10" offset="1" md="4" offset-md="4">
            <v-card class="mb-2">
                <v-toolbar color="primary" class="white--text">Register</v-toolbar>
                <v-card-text>
                    <v-form>
                        <v-text-field name="fullname" label="Full Name" type="text" :rules="rules.fullname"
                            v-model="form.fullname" />
                        <v-text-field name="email" label="Email" type="email" :rules="rules.email" v-model="form.email"
                            @keyup="checkEmail" />
                        <v-text-field name="password" label="Password" type="password" :rules="rules.password"
                            v-model="form.password" />
                        <v-text-field name="retype_password" label="Re-Password" :rules="rules.retype_password"
                            type="password" v-model="form.retype_password" />
                    </v-form>
                </v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn @click="onSubmit" color="primary" :disable="isDisable">
                        <span v-if="!isDisable">
                            Register
                        </span>
                        <v-progress-circular v-else color="primary" indeterminate>
                        </v-progress-circular>
                    </v-btn>
                </v-card-actions>
            </v-card>
            <p>Kamu sudah punya akun ?<v-btn href="/login" plain>login</v-btn>
            </p>
        </v-col>
    </v-row>
</template>

<script>
export default ({
    data() {
        return {
            emailExist: false,
            isDisable: false,
            form: {
                fullname: '',
                email: '',
                password: '',
                retype_password: ''
            },
            rules: {
                fullname: [
                    v => !!v || 'Full Name is required',
                ],
                email: [
                    v => !!v || 'E-mail is required',
                    v => /.+@.+\..+/.test(v) || 'E-mail must be valid',
                    // v => !!this.emailExist || 'E-mail already exist'
                ],
                password: [
                    v => !!v || 'Password is required',
                    v => (v && v.length >= 6) || 'Password must be greater than 6 characters'
                ],
                retype_password: [
                    v => !!v || 'Re-Password is required',
                    v => v === this.form.password || 'Password not match'
                ]
            }
        }
    },
    methods: {
        checkEmail() {
            this.$axios.$post('http://localhost:3000/auth/check-email', this.form)
                .then(response => {
                    this.emailExist = false
                })
                .catch((error) => {
                    this.emailExist = true
                })
        },
        onSubmit() {
            this.isDisable = true
            this.$axios.$post('http://localhost:3000/auth/register', this.form)
                .then(response => {
                    this.isDisable = false
                    // redirect to login page
                    this.$router.push('/login')
                })
                .catch((error) => {
                    this.isDisable = false
                    console.log(error)
                })
        }
    }
})
</script>
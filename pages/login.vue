<template>
    <v-row>
        <v-col cols="10" offset="1" md="4" offset-md="4">
            <v-card class="mb-2">
                <v-toolbar color="primary" class="white--text">Login</v-toolbar>
                <v-card-text>
                    <v-form>
                        <v-text-field name="email" label="Email" type="email" v-model="form.email" />
                        <v-text-field name="password" label="Password" type="password" v-model="form.password" />
                    </v-form>
                </v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn @click="onSubmit" color="primary">Login</v-btn>
                </v-card-actions>
            </v-card>
            <p>Kamu belum punya akun ?<v-btn href="/register" plain>register</v-btn>
            </p>
        </v-col>
    </v-row>
</template>

<script>
export default ({
    data() {
        return {
            isDisable: false,
            form: {
                email: '',
                password: ''
            }
        }
    },
    methods: {
        storeWelcomeScreen() {
            localStorage.setItem('welcomeScreen', true)
        },
        onSubmit() {
            this.isDisable = true
            this.$axios.$post('http://localhost:3000/auth/login', this.form)
                .then(response => {
                    this.isDisable = false
                    // store passed welcome screen
                    if (!localStorage.WelcomeScreen) {
                        this.storeWelcomeScreen()
                    }
                    // redirect to login page
                    // this.$router.push('/')
                })
                .catch((error) => {
                    console.log(error)
                })
        }
    }
})
</script>
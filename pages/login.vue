<template>
    <v-row>
        <v-col cols="10" offset="1" md="4" offset-md="4">
            <v-card class="mb-2">
                <v-toolbar color="primary" class="white--text">Login</v-toolbar>
                <v-card-text>
                    <v-alert v-if="isError" color="red lighten-2" dark>{{ $t(message) }}</v-alert>
                    <v-form>
                        <v-text-field name="email" label="Email" type="email" v-model="form.email" autofocus />
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
import { mapMutations } from 'vuex'
export default ({
    middleware: ['unauthenticated'],
    data() {
        return {
            isDisable: false,
            isError: false,
            message: '',
            form: {
                email: '',
                password: ''
            }
        }
    },
    methods: {
        ...mapMutations('auth', {
            setFullname: 'setFullname',
            setAccessToken: 'setAccessToken',
            setRefreshToken: 'setRefreshToken',
        }),
        storeWelcomeScreen() {
            localStorage.setItem('WelcomeScreen', true)
        },
        onSubmit() {
            this.isDisable = true
            this.$axios.$post('/auth/login', this.form)
                .then(response => {
                    // login success
                    this.isDisable = false

                    // store passed welcome screen
                    if (!localStorage.WelcomeScreen) {
                        this.storeWelcomeScreen()
                    }

                    // store auth data
                    this.setFullname(response.fullname)
                    this.setAccessToken(response.accessToken)
                    this.setRefreshToken(response.refreshToken)

                    // redirect to login page
                    this.$router.push('/dashboard')
                })
                .catch((error) => {
                    this.message = error.response.data.message
                    this.isError = true
                    this.isDisable = false
                })
        }
    },
    mounted() {
        if (this.$route.params.message == "AUTH_REQUIRED") {
            this.message = this.$route.params.message
            this.isError = true
        }
    }
})
</script>
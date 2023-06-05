<template>
    <v-row>
        <v-col cols="10" offset="1" md="4" offset-md="4">
            <v-card class="mb-2">
                <v-card-text>
                    <h2>Logout <v-icon class="ml-3 spin-loading">mdi-loading</v-icon>
                    </h2>
                </v-card-text>
            </v-card>
        </v-col>
    </v-row>
</template>

<script>
import { mapMutations } from 'vuex'
export default {
    head: {
        title: 'Logout'
    },
    middleware: ['authenticated'],
    methods: {
        ...mapMutations('auth', {
            setFullname: 'setFullname',
            setAccessToken: 'setAccessToken',
            setRefreshToken: 'setRefreshToken',
        }),
    },
    mounted() {
        setTimeout(() => {
            this.setFullname(null),
                this.setAccessToken(null),
                this.setRefreshToken(null),
                localStorage.removeItem('WelcomeScreen'),
                this.$router.push('/login')
        }, 2000)
    }
}
</script>
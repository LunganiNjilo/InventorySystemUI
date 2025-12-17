<template>
  <v-container>
    <v-card max-width="400" class="mx-auto mt-10 pa-4">
      <v-text-field v-model="username" label="Username" />
      <v-text-field v-model="password" label="Password" type="password" />
      <v-btn color="primary" block @click="login">Login</v-btn>
    </v-card>
  </v-container>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'

const username = ref('admin')
const password = ref('admin123')

const router = useRouter()

async function login() {
  const res = await $fetch('http://localhost:8080/api/auth/login', {
    method: 'POST',
    body: {
      username: username.value,
      password: password.value
    }
  })

  localStorage.setItem('token', res.token)
  router.push('/')
}
</script>


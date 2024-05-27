<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated v-if="authState.isLoggedIn">
      <q-toolbar>
        <q-btn
          flat
          dense
          round
          icon="menu"
          aria-label="Menu"
          @click="drawer = !drawer"
        />
        <q-toolbar-title>
          TypeScript Portal
        </q-toolbar-title>
        <div>Portal v{{ $q.version }}</div>
      </q-toolbar>
    </q-header>

    <q-drawer
      v-model="drawer"
      show-if-above
      :width="200"
      :breakpoint="400"
      v-if="authState.isLoggedIn"
    >
      <q-scroll-area style="height: calc(100% - 150px); margin-top: 150px; border-right: 1px solid #ddd">
        <q-list padding>
          <q-item clickable v-ripple>
            <q-item-section avatar>
              <q-icon name="bar_chart" />
            </q-item-section>
            <q-item-section>
              Reports
            </q-item-section>
          </q-item>
          <q-item active clickable v-ripple>
            <q-item-section avatar>
              <q-icon name="person" />
            </q-item-section>
            <q-item-section>
              Profile
            </q-item-section>
          </q-item>
          <q-item clickable v-ripple @click="logout">
            <q-item-section avatar>
              <q-icon name="logout" />
            </q-item-section>
            <q-item-section>Logoff</q-item-section>
          </q-item>
        </q-list>
      </q-scroll-area>

      <q-img class="absolute-top" src="TS-Zoom-2_resized.jpg" style="height: 150px">
        <div class="absolute-bottom bg-transparent">
          <q-avatar size="56px" class="q-mb-sm">
            <img src="silhouette_transparent.webp">
          </q-avatar>
          <div class="text-weight-bold">Nestor Urquiza</div>
          <div>nurquiza</div>
        </div>
      </q-img>
    </q-drawer>

    <q-page-container>
      <router-view />
    </q-page-container>

    <q-dialog v-model="showLoginDialog">
      <q-card style="width: 400px;">
        <q-card-section>
          <div class="text-h6">Please log in</div>
          <q-input filled v-model="email" label="Email" />
          <q-input filled v-model="password" label="Password" type="password" v-if="!useSSO" />
          <q-checkbox v-model="useSSO" label="Use SSO" />
        </q-card-section>
        <q-card-section align="right">
          <q-btn flat label="Cancel" v-close-popup />
          <q-btn flat label="Login" color="primary" @click="login" />
        </q-card-section>
        <q-card-section>
          <div v-if="loginError" class="text-negative">{{ loginError }}</div>
        </q-card-section>
      </q-card>
    </q-dialog>
  </q-layout>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { authState } from 'src/store/auth' // Adjust path as needed for your project structure

const drawer = ref(false)
const email = ref('')
const password = ref('')
const useSSO = ref(false)
const showLoginDialog = ref(false)
const loginError = ref('')

onMounted(checkLogin)

async function checkLogin() {
  try {
    const response = await fetch('/api/profile', {
      method: 'GET',
      credentials: 'include'  // Important for sessions
    })
    if (response.status === 401) {
      showLoginDialog.value = true
    } else {
      const userData = await response.json()
      authState.user = userData
      authState.isLoggedIn = true
    }
  } catch (err) {
    console.error("Error checking login status", err)
  }
}

async function login() {
  try {
    const requestBody = { email: email.value }
    if (useSSO.value) {
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(requestBody),
        credentials: 'include'
      })
      if (!response.ok) {
        loginError.value = (await response.json()).error
        return
      }
      const data = await response.json()
      window.location.href = data.ssoUrl
    } else {
      if (!password.value) {
        loginError.value = "Password is required"
        return
      }
      requestBody.password = password.value
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(requestBody),
        credentials: 'include'
      })
      if (!response.ok) {
        loginError.value = (await response.json()).error
        return
      }
      showLoginDialog.value = false
      authState.isLoggedIn = true
    }
  } catch (err) {
    loginError.value = "Failed to log in"
  }
}

async function logout() {
  try {
    await fetch('/api/logout', {
      method: 'POST',
      credentials: 'include'
    })
    authState.isLoggedIn = false
    authState.user = null
    showLoginDialog.value = true  // Reopen login dialog or redirect as necessary
  } catch (err) {
    console.error("Error during logout", err)
  }
}
</script>


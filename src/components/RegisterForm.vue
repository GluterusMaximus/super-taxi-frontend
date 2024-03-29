<script setup lang="ts">
import { computed, ref } from 'vue'
import { RouterLink } from 'vue-router'
import { emailRule, requiredRule } from '@/helpers/rules'
import { Role } from '@/types/role'
import { createUser } from '@/api'
import router from '@/router'
import { useUserStore } from '@/stores/user'

const password = ref('')
const isPasswordVisible = ref(false)
const repeatPassword = ref('')
const isRepeatPasswordVisible = ref(false)
const email = ref('')
const name = ref('')
const role = ref<Role | null>(null)
const registrationError = ref<null | Error>(null)

const passwordMatchRule = computed(() => password.value === repeatPassword.value || 'Passwords must match')
const isRegisterDisabled = computed(() =>
  !email.value
  || !name.value
  || !password.value
  || !repeatPassword.value
  || (emailRule(email.value) !== true)
  || (passwordMatchRule.value !== true)
  || !role.value)

const { setUser } = useUserStore()

async function register(): Promise<void> {
  if (role.value === null) {
    return
  }

  try {
    const user = await createUser({
      name: name.value,
      role: role.value,
      email: email.value,
      password: password.value,
    })

    setUser(user)

    router.push('/')
  }
  catch (error) {
    registrationError.value = error as Error
  }
}
</script>

<template>
  <div class="login-form">
    <v-img
      class="mx-auto my-6"
      max-width="100"
      src="/favicon.svg"
    />

    <v-card
      class="mx-auto pa-12 pb-8"
      elevation="8"
      max-width="448"
      rounded="lg"
      title="Register"
    >
      <div class="text-subtitle-1 text-medium-emphasis">
        Email
      </div>

      <v-text-field
        v-model="email"
        :rules="[requiredRule, emailRule]"
        density="compact"
        placeholder="Email address"
        prepend-inner-icon="mdi-email-outline"
        variant="outlined"
      />

      <div class="text-subtitle-1 text-medium-emphasis">
        Name
      </div>

      <v-text-field
        v-model="name"
        :rules="[requiredRule]"
        density="compact"
        placeholder="Your name"
        prepend-inner-icon="mdi-face-man"
        variant="outlined"
      />

      <div class="text-subtitle-1 text-medium-emphasis d-flex align-center justify-space-between mb-2">
        Choose your role
      </div>

      <v-btn-toggle
        v-model="role"
        color="#1155DC"
        rounded="0"
        class="mb-5 d-flex justify-center"
        group
      >
        <v-btn :value="Role.Passenger">
          Passenger
        </v-btn>

        <v-btn :value="Role.Driver">
          Driver
        </v-btn>
      </v-btn-toggle>

      <div class="text-subtitle-1 text-medium-emphasis d-flex align-center justify-space-between">
        Password
      </div>

      <v-text-field
        v-model="password"
        :rules="[requiredRule]"
        :append-inner-icon="isPasswordVisible ? 'mdi-eye-off' : 'mdi-eye'"
        :type="isPasswordVisible ? 'text' : 'password'"
        density="compact"
        placeholder="Enter your password"
        prepend-inner-icon="mdi-lock-outline"
        variant="outlined"
        @click:append-inner="isPasswordVisible = !isPasswordVisible"
      />

      <div class="text-subtitle-1 text-medium-emphasis d-flex align-center justify-space-between">
        Repeat password
      </div>

      <v-text-field
        v-model="repeatPassword"
        :append-inner-icon="isRepeatPasswordVisible ? 'mdi-eye-off' : 'mdi-eye'"
        :type="isRepeatPasswordVisible ? 'text' : 'password'"
        :rules="[requiredRule, passwordMatchRule]"
        density="compact"
        placeholder="Repeat your password"
        prepend-inner-icon="mdi-repeat"
        variant="outlined"
        @click:append-inner="isRepeatPasswordVisible = !isRepeatPasswordVisible"
      />

      <v-btn
        class="mb-5"
        color="blue"
        size="large"
        variant="tonal"
        :disabled="isRegisterDisabled"
        block
        @click="register"
      >
        Register
      </v-btn>

      <v-alert v-if="registrationError" :text="registrationError.message" type="error" />

      <v-card-text class="text-center">
        <RouterLink
          class="text-blue text-decoration-none"
          to="/login"
        >
          Already have an account? Log in
        </RouterLink>
      </v-card-text>
    </v-card>
  </div>
</template>

<style scoped>
.login-form {
  min-width: 500px;
}
</style>

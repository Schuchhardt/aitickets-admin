<script setup lang="ts">
import axios, { AxiosError } from 'axios'
import { VForm } from 'vuetify/components/VForm'
import { useAppAbility } from '@/plugins/casl/useAppAbility'
import type { UserAbility } from '@/plugins/casl/AppAbility'
import { useGenerateImageVariant } from '@core/composable/useGenerateImageVariant'
import { requiredValidator } from '@validators'
import { default as authV2MaskDark, default as authV2MaskLight } from '@images/pages/bg.png'

console.log("🧹 localStorage LIMPIO")

const isPasswordVisible = ref(false)
const authThemeMask = useGenerateImageVariant(authV2MaskLight, authV2MaskDark)

const route = useRoute()
const router = useRouter()
const ability = useAppAbility()

if (route.query.logout === '1') {
  console.log("👋 Logout detectado. Limpiando sesión...")
  localStorage.removeItem('userData')
  localStorage.removeItem('accessToken')
  localStorage.removeItem('userAbilities')
  ability.update([])
  router.replace('/login')
}

const errorMessages = {
  USER_NOT_FOUND: 'Usuario no encontrado.',
  PASSWORD_NOT_MATCH: 'Contraseña incorrecta.',
  USER_NO_MODULES: 'Usuario no tiene acceso a ningún módulo.',
  DEFAULT: 'Los datos ingresados son incorrectos.'
} as const

interface ErrorResponse {
  code: keyof typeof errorMessages
  message: string
}

const errorDescription = ref('')
const refVForm = ref<VForm>()
const email = ref('')
const password = ref('')
const showErrors = ref(false)
const isLoading = ref(false)

const handleLoginError = (error: AxiosError<ErrorResponse>) => {
  showErrors.value = true
  isLoading.value = false
  if (error.response?.data?.code) {
    const errorKey = error.response.data.code
    errorDescription.value = errorMessages[errorKey] || errorMessages.DEFAULT
  } else {
    errorDescription.value = 'Error de conexión. Por favor, intente nuevamente.'
  }
  console.error('Login error:', error)
}

const login = async () => {
  console.log("🔐 Entrando a login()...")

  const userAbilities: UserAbility[] = [
    { action: 'manage', subject: 'all' }
  ]

  try {
    isLoading.value = true
    showErrors.value = false

    const response = await axios.post<any>(
      `${process.env.API_URL}/login`,
      { email: email.value, password: password.value }
    )

    console.log("📦 Respuesta completa del login:", response.data)

    if (!response?.data?.auth) {
      throw new Error('Login failed: Invalid response format')
    }

    const { auth: accessToken, user: userData, login_iframe_url } = response.data

    console.log("🧾 userData:", userData)
    console.log("🔍 userType:", userData?.userType)
    console.log("📧 Email:", userData?.email)


const role = userData?.role || 'unknown'

if (role === 'ticket_validator') {
  console.log("Usuario es ticket_validator. Redirigiendo a QR...")
  window.location.replace(`https://aitickets.cl/qr?token=${accessToken}`)
  return
}

if (role === 'admin') {
  console.log("Usuario admin. Redirigiendo a Eventos...")
  window.location.replace(`https://aitickets.retool.com/p/my-events/Eventos?token=${accessToken}`)
  return
}

if (role === 'ticket_admin') {
  console.log("Usuario ticket_admin. Redirigiendo a Entradas...")
  window.location.replace(`https://aitickets.retool.com/p/my-events/Entradas?token=${accessToken}`)
  return
}

if (role === 'finance_admin') {
  console.log("Usuario finance_admin. Redirigiendo a Cobros...")
  window.location.replace(`https://aitickets.retool.com/p/my-events/Cobros?token=${accessToken}`)
  return
}

if (role === 'equipo_admin') {
  console.log("Usuario equipo_admin. Redirigiendo a Equipo...")
  window.location.replace(`https://aitickets.retool.com/p/my-events/Equipo?token=${accessToken}`)
  return
}

console.log("Rol desconocido. Redirigiendo al dashboard general...")
window.location.replace(`https://aitickets.retool.com/p/my-events/Eventos?token=${accessToken}`)


  } catch (error) {
    handleLoginError(error as AxiosError<ErrorResponse>)
    throw error
  }
}

const onSubmit = () => {
  refVForm.value?.validate().then(({ valid }) => {
    if (valid) login()
  })
}
</script>




<template>
  <div>

    <VRow
      no-gutters
      class="auth-wrapper pa-0"
    >
      <VCol
        md="8"
        class="d-none d-md-flex align-center justify-center position-relative bg-black"
      >
        <div class="d-flex align-center justify-center w-100 pa-0 pe-0">
          <VImg
            max-width="1024px"
            :src="authThemeMask"
            class="auth-illustration"
          />
        </div>
      </VCol>

      <VCol
        cols="12"
        md="4"
        class="auth-card-v2 d-flex align-center justify-center"
      >
      
      <VCard
      flat
      :max-width="500"
      class="mt-4 mt-sm-0 pa-4"
      >
          <VRow>
            <img src="/logo.png" class="logo-img" alt="Logo">
          </VRow>
          <VCardText>
            <h5 class="text-h5 mb-1 text-center">
              👋🏻
            </h5>
            <p class="mb-0 text-center">
              Para empezar inicia sesión en tu cuenta.
            </p>
          </VCardText>
          <VCardText v-if="showErrors">
            <VAlert
              color="primary"
              variant="tonal"
            >
              <p class="text-caption mb-0">
                <strong>{{ errorDescription }}</strong>
              </p>
            </VAlert>
          </VCardText>
          <VCardText>
            <VForm ref="refVForm" @submit.prevent="onSubmit()">
              <VRow>
                <!-- email -->
                <VCol cols="12">
                  <VTextField
                    v-model="email"
                    autofocus
                    label="Correo"
                    type="text"
                    :rules="[requiredValidator]"
                  />
                </VCol>

                <!-- password -->
                <VCol cols="12">
                  <VTextField
                    v-model="password"
                    label="Contraseña"
                    :type="isPasswordVisible ? 'text' : 'password'"
                    :append-inner-icon="isPasswordVisible ? 'mdi-eye-off-outline' : 'mdi-eye-outline'"
                    @click:append-inner="isPasswordVisible = !isPasswordVisible"
                    :rules="[requiredValidator]"
                  />

                  <!-- <div class="d-flex align-center flex-wrap justify-space-between mt-1 mb-4">
                    <RouterLink
                      class="text-primary text-center ms-2 mb-1"
                      :to="{ name: 'pages-authentication-forgot-password-v2' }"
                    >
                      ¿Olvidaste tu contraseña?
                    </RouterLink>
                  </div> -->

                </VCol>
                <VCol cols="12">
                  <VBtn
                    block
                    :loading="isLoading"
                    type="submit"
                  >
                    Ingresar
                  </VBtn>
                </VCol>

              </VRow>
            </VForm>
          </VCardText>
        </VCard>
      </VCol>
    </VRow>
  </div>
</template>

<style lang="scss">
@use "@core/scss/template/pages/page-auth.scss";

.text-h5{
  font-family: 'Unbounded', sans-serif !important;
  font-weight: 600;
}

</style>

<route lang="yaml">
meta:
  layout: blank
</route>

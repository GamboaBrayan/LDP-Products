<template>
  <div class="register-container">
    <div class="register-card">
      <div class="logo-section">
        <h1>LDP MERCHANDISING</h1>
      </div>

      <h2>Crear Cuenta</h2>
      <p class="subtitle">Unete a LDP</p>

      <form @submit.prevent="handleRegister">
        <div class="form-row">
          <div class="form-group">
            <label>Nombre Completo</label>
            <input 
              type="text" 
              v-model="formData.full_name" 
              placeholder="John Doe"
              required
            />
          </div>

          <div class="form-group">
            <label>Nombre de Usuario</label>
            <input 
              type="text" 
              v-model="formData.username" 
              placeholder="johndoe"
              required
            />
          </div>
        </div>

        <div class="form-group">
          <label>Email</label>
          <input 
            type="email" 
            v-model="formData.email" 
            placeholder="john@example.com"
            required
          />
        </div>

        <div class="form-group">
          <label>Número (opcional)</label>
          <input 
            type="tel" 
            v-model="formData.phone" 
            placeholder="+51 987 654 321"
          />
        </div>

        <div class="form-group">
          <label>Compañia (opcional)</label>
          <input 
            type="text" 
            v-model="formData.company" 
            placeholder="Company Name"
          />
        </div>

        <div class="form-group">
          <label>Contraseña</label>
          <div class="password-group">
            <input 
              :type="showPassword ? 'text' : 'password'" 
              v-model="formData.password" 
              placeholder="Minimo 6 caracteres"
              required
              minlength="6"
            />
            <button
              type="button"
              class="toggle-password"
              @click="togglePassword"
              :aria-pressed="showPassword.toString()"
              :title="showPassword ? 'Ocultar contraseña' : 'Mostrar contraseña'"
            >
              <svg v-if="!showPassword" xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
                <path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"></path>
                <circle cx="12" cy="12" r="3"></circle>
              </svg>
              <svg v-else xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
                <path d="M17.94 17.94A10.94 10.94 0 0 1 12 20c-7 0-11-8-11-8a21.8 21.8 0 0 1 5.11-6.14"></path>
                <path d="M1 1l22 22"></path>
                <path d="M3.6 3.6A20.5 20.5 0 0 1 12 4c7 0 11 8 11 8a21.2 21.2 0 0 1-2.83 4.12"></path>
                <path d="M14.12 14.12A3 3 0 0 1 9.88 9.88"></path>
              </svg>
            </button>
          </div>
        </div>

        <div class="form-group">
          <label>Confirmar Contraseña</label>
          <div class="password-group">
            <input 
              :type="showConfirmPassword ? 'text' : 'password'" 
              v-model="formData.confirmPassword" 
              placeholder="Repite tu contraseña"
              required
            />
            <button
              type="button"
              class="toggle-password"
              @click="toggleConfirmPassword"
              :aria-pressed="showConfirmPassword.toString()"
              :title="showConfirmPassword ? 'Ocultar contraseña' : 'Mostrar contraseña'"
            >
              <svg v-if="!showConfirmPassword" xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
                <path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"></path>
                <circle cx="12" cy="12" r="3"></circle>
              </svg>
              <svg v-else xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
                <path d="M17.94 17.94A10.94 10.94 0 0 1 12 20c-7 0-11-8-11-8a21.8 21.8 0 0 1 5.11-6.14"></path>
                <path d="M1 1l22 22"></path>
                <path d="M3.6 3.6A20.5 20.5 0 0 1 12 4c7 0 11 8 11 8a21.2 21.2 0 0 1-2.83 4.12"></path>
                <path d="M14.12 14.12A3 3 0 0 1 9.88 9.88"></path>
              </svg>
            </button>
          </div>
        </div>

        <div v-if="errorMessage" class="error-message">
          {{ errorMessage }}
        </div>

        <div v-if="successMessage" class="success-message">
          {{ successMessage }}
        </div>

        <button type="submit" class="register-button" :disabled="loading">
          {{ loading ? 'Creando cuenta...' : 'Crear Cuenta' }}
        </button>
      </form>

      <div class="login-section">
        <p>Ya tiene cuenta? <router-link to="/login">Iniciar sesión</router-link></p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { supabase } from '../lib/supabase'

const router = useRouter()
const loading = ref(false)
const errorMessage = ref('')
const successMessage = ref('')

const formData = ref({
  full_name: '',
  username: '',
  email: '',
  phone: '',
  company: '',
  password: '',
  confirmPassword: ''
})

const showPassword = ref(false)
const showConfirmPassword = ref(false)

const togglePassword = () => {
  showPassword.value = !showPassword.value
}

const toggleConfirmPassword = () => {
  showConfirmPassword.value = !showConfirmPassword.value
}

const handleRegister = async () => {
  try {
    loading.value = true
    errorMessage.value = ''
    successMessage.value = ''

    // Validar que las contraseñas coincidan
    if (formData.value.password !== formData.value.confirmPassword) {
      throw new Error('Passwords do not match')
    }

    // Registrar usuario en Supabase Auth
    const { data: authData, error: authError } = await supabase.auth.signUp({
      email: formData.value.email,
      password: formData.value.password,
      options: {
        data: {
          full_name: formData.value.full_name,
          username: formData.value.username
        }
      }
    })

    if (authError) throw authError

    // Actualizar perfil con información adicional
    if (authData.user) {
      const { error: profileError } = await supabase
        .from('profiles')
        .update({
          username: formData.value.username,
          phone: formData.value.phone,
          company: formData.value.company
        })
        .eq('id', authData.user.id)

      if (profileError) console.error('Error updating profile:', profileError)
    }

    successMessage.value = 'Account created! Please check your email to verify your account.'
    
    // Redirigir al login después de 3 segundos
    setTimeout(() => {
      router.push('/login')
    }, 3000)

  } catch (error) {
    errorMessage.value = error.message
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
.register-container {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 40px 20px;
}

.register-card {
  background: white;
  border-radius: 16px;
  padding: 48px;
  width: 100%;
  max-width: 600px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
  max-height: 90vh;
  overflow-y: auto;
}

.logo-section {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 32px;
}

.logo-section h1 {
  font-size: 28px;
  font-weight: 700;
  color: #1f2937;
}

h2 {
  font-size: 24px;
  font-weight: 700;
  color: #1f2937;
  margin-bottom: 8px;
}

.subtitle {
  color: #6b7280;
  margin-bottom: 32px;
}

.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
}

.form-group {
  margin-bottom: 20px;
}

label {
  display: block;
  font-weight: 600;
  color: #374151;
  margin-bottom: 8px;
  font-size: 14px;
}

input {
  width: 100%;
  padding: 12px 16px;
  border: 2px solid #e5e7eb;
  border-radius: 8px;
  font-size: 16px;
  transition: all 0.2s;
}

input:focus {
  outline: none;
  border-color: #3b82f6;
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
}

.error-message {
  background-color: #fee2e2;
  color: #dc2626;
  padding: 12px;
  border-radius: 8px;
  margin-bottom: 16px;
  font-size: 14px;
}

.success-message {
  background-color: #d1fae5;
  color: #059669;
  padding: 12px;
  border-radius: 8px;
  margin-bottom: 16px;
  font-size: 14px;
}

.register-button {
  width: 100%;
  padding: 14px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: transform 0.2s;
}

.register-button:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 8px 16px rgba(102, 126, 234, 0.4);
}

.register-button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.login-section {
  margin-top: 24px;
  text-align: center;
  color: #6b7280;
}

.login-section a {
  color: #3b82f6;
  text-decoration: none;
  font-weight: 600;
}

.login-section a:hover {
  text-decoration: underline;
}

/* estilos para campo de contraseña con ojo */
.password-group {
  position: relative;
}
.password-group input {
  padding-right: 48px; /* espacio para el botón ojo */
}
.toggle-password {
  position: absolute;
  right: 10px;
  top: 50%;
  transform: translateY(-50%);
  background: transparent;
  border: none;
  padding: 6px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  color: #6b7280;
  cursor: pointer;
  border-radius: 6px;
}
.toggle-password:hover {
  background: rgba(0,0,0,0.04);
}
.toggle-password:focus {
  outline: none;
  box-shadow: 0 0 0 3px rgba(59,130,246,0.12);
}
</style>
<template>
  <div class="profile-page">
    <Navbar @logout="handleLogout" />
    
    <div class="container">
      <div class="profile-header">
        <h1>Mi Perfíl</h1>
        <p>Configura tu información personal</p>
      </div>

      <div class="profile-content">
        <!-- Avatar Section -->
        <div class="avatar-section">
          <div class="avatar-large">
            <svg v-if="!profile.avatar_url" width="80" height="80" viewBox="0 0 80 80" fill="none">
              <circle cx="40" cy="32" r="16" fill="#3b82f6"/>
              <path d="M16 68c0-16 12-24 24-24s24 8 24 24" stroke="#3b82f6" stroke-width="8" stroke-linecap="round"/>
            </svg>
            <img v-else :src="profile.avatar_url" alt="Avatar" />
            
            <!-- Loading overlay -->
            <div v-if="uploadingAvatar" class="upload-overlay">
              <div class="spinner"></div>
            </div>
          </div>
          
          <input 
            type="file" 
            ref="fileInput" 
            @change="handleFileSelect" 
            accept="image/*"
            style="display: none"
          />
          
          <button 
            class="change-avatar-btn" 
            @click="triggerFileInput"
            :disabled="uploadingAvatar"
          >
            {{ uploadingAvatar ? 'Uploading...' : 'Change Photo' }}
          </button>
          
          <button 
            v-if="profile.avatar_url" 
            class="remove-avatar-btn"
            @click="removeAvatar"
            :disabled="uploadingAvatar"
          >
            Remove Photo
          </button>
          
          <p class="avatar-hint">JPG, PNG or GIF. Max 2MB</p>
        </div>

        <!-- Profile Form -->
        <div class="profile-form-section">
          <form @submit.prevent="updateProfile">
            <div class="form-section">
              <h3>Información Personal</h3>
              
              <div class="form-row">
                <div class="form-group">
                  <label>Nombre Completo</label>
                  <input 
                    type="text" 
                    v-model="profile.full_name" 
                    placeholder="John Doe"
                  />
                </div>

                <div class="form-group">
                  <label>Nombre de Usuario</label>
                  <input 
                    type="text" 
                    v-model="profile.username" 
                    placeholder="johndoe"
                  />
                </div>
              </div>

              <div class="form-row">
                <div class="form-group">
                  <label>Email</label>
                  <input 
                    type="email" 
                    v-model="profile.email" 
                    disabled
                    class="disabled-input"
                  />
                  <small>Email no puede ser cambiado</small>
                </div>

                <div class="form-group">
                  <label>Número</label>
                  <input 
                    type="tel" 
                    v-model="profile.phone" 
                    placeholder="+51 999 999 999"
                  />
                </div>
              </div>

              <div class="form-group">
                <label>Compañia</label>
                <input 
                  type="text" 
                  v-model="profile.company" 
                  placeholder="Company Name"
                />
              </div>
            </div>

            <div class="form-section">
              <h3>Lugar de Residencia</h3>
              
              <div class="form-group">
                <label>Dirección</label>
                <input 
                  type="text" 
                  v-model="profile.address" 
                  placeholder="123 Main Street"
                />
              </div>

              <div class="form-row">
                <div class="form-group">
                  <label>Ciudad</label>
                  <input 
                    type="text" 
                    v-model="profile.city" 
                    placeholder="Lima"
                  />
                </div>

                <div class="form-group">
                  <label>País</label>
                  <input 
                    type="text" 
                    v-model="profile.country" 
                    placeholder="Peru"
                  />
                </div>
              </div>
            </div>

            <div v-if="errorMessage" class="error-message">
              {{ errorMessage }}
            </div>

            <div v-if="successMessage" class="success-message">
              {{ successMessage }}
            </div>

            <div class="form-actions">
              <button type="button" @click="loadProfile" class="cancel-btn">Cancelar</button>
              <button type="submit" class="save-btn" :disabled="loading">
                {{ loading ? 'Guardando...' : 'Guardar Cambios' }}
              </button>
            </div>
          </form>

          <!-- Change Password Section -->
          <div class="form-section password-section">
            <h3>Cambiar Contraseña</h3>
            
            <form @submit.prevent="updatePassword">
              <div class="form-group">
                <label>Nueva Contraseña</label>
                <input 
                  type="password" 
                  v-model="passwordData.newPassword" 
                  placeholder="Ingresa nueva contraseña"
                  minlength="6"
                />
              </div>

              <div class="form-group">
                <label>Confirma nueva contraseña</label>
                <input 
                  type="password" 
                  v-model="passwordData.confirmPassword" 
                  placeholder="Confirma nueva contraseña"
                />
              </div>

              <div v-if="passwordError" class="error-message">
                {{ passwordError }}
              </div>

              <div v-if="passwordSuccess" class="success-message">
                {{ passwordSuccess }}
              </div>

              <button type="submit" class="save-btn" :disabled="loadingPassword">
                {{ loadingPassword ? 'Actualizando...' : 'Actualizar Contraseña' }}
              </button>
            </form>
          </div>

          <!-- Account Info -->
          <div class="account-info">
            <h3>Información de cuenta</h3>
            <div class="info-item">
              <span class="info-label">Cuenta Creada:</span>
              <span class="info-value">{{ formatDate(profile.created_at) }}</span>
            </div>
            <div class="info-item">
              <span class="info-label">Ultima Actualización:</span>
              <span class="info-value">{{ formatDate(profile.updated_at) }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { supabase } from '../lib/supabase'
import Navbar from '../components/Navbar.vue'

const router = useRouter()
const loading = ref(false)
const loadingPassword = ref(false)
const uploadingAvatar = ref(false)
const errorMessage = ref('')
const successMessage = ref('')
const passwordError = ref('')
const passwordSuccess = ref('')
const fileInput = ref(null)

const profile = ref({
  id: '',
  email: '',
  full_name: '',
  username: '',
  avatar_url: '',
  phone: '',
  company: '',
  address: '',
  city: '',
  country: '',
  created_at: '',
  updated_at: ''
})

const passwordData = ref({
  newPassword: '',
  confirmPassword: ''
})

const loadProfile = async () => {
  try {
    const { data: { user } } = await supabase.auth.getUser()
    
    if (!user) {
      router.push('/login')
      return
    }

    const { data, error } = await supabase
      .from('profiles')
      .select('*')
      .eq('id', user.id)
      .single()

    if (error) throw error

    profile.value = {
      ...data,
      email: user.email
    }
  } catch (error) {
    console.error('Error cargando perfil:', error)
    errorMessage.value = 'Error loading profile data'
  }
}

const updateProfile = async () => {
  try {
    loading.value = true
    errorMessage.value = ''
    successMessage.value = ''

    const { error } = await supabase
      .from('profiles')
      .update({
        full_name: profile.value.full_name,
        username: profile.value.username,
        phone: profile.value.phone,
        company: profile.value.company,
        address: profile.value.address,
        city: profile.value.city,
        country: profile.value.country,
        updated_at: new Date().toISOString()
      })
      .eq('id', profile.value.id)

    if (error) throw error

    successMessage.value = 'Perfil actualizado con éxito!'
    
    setTimeout(() => {
      successMessage.value = ''
    }, 3000)
  } catch (error) {
    errorMessage.value = error.message
  } finally {
    loading.value = false
  }
}

const updatePassword = async () => {
  try {
    loadingPassword.value = true
    passwordError.value = ''
    passwordSuccess.value = ''

    if (passwordData.value.newPassword !== passwordData.value.confirmPassword) {
      throw new Error('Las contraseñas no coinciden')
    }

    if (passwordData.value.newPassword.length < 6) {
      throw new Error('La contraseña debe tener al menos 6 caracteres')
    }

    const { error } = await supabase.auth.updateUser({
      password: passwordData.value.newPassword
    })

    if (error) throw error

    passwordSuccess.value = 'La contraseña ha sido actualizada con éxito!'
    passwordData.value.newPassword = ''
    passwordData.value.confirmPassword = ''

    setTimeout(() => {
      passwordSuccess.value = ''
    }, 3000)
  } catch (error) {
    passwordError.value = error.message
  } finally {
    loadingPassword.value = false
  }
}

const triggerFileInput = () => {
  fileInput.value.click()
}

const handleFileSelect = async (event) => {
  const file = event.target.files[0]
  if (!file) return

  // Validar tipo de archivo
  if (!file.type.startsWith('image/')) {
    errorMessage.value = 'Please select an image file'
    return
  }

  // Validar tamaño (2MB máximo)
  if (file.size > 2 * 1024 * 1024) {
    errorMessage.value = 'Image size must be less than 2MB'
    return
  }

  await uploadAvatar(file)
}

const uploadAvatar = async (file) => {
  try {
    uploadingAvatar.value = true
    errorMessage.value = ''

    const { data: { user } } = await supabase.auth.getUser()
    if (!user) throw new Error('No user found')

    // Crear nombre único para el archivo
    const fileExt = file.name.split('.').pop()
    const fileName = `${user.id}/avatar.${fileExt}`

    // Eliminar avatar anterior si existe
    if (profile.value.avatar_url) {
      const oldPath = profile.value.avatar_url.split('/').pop()
      await supabase.storage
        .from('avatars')
        .remove([`${user.id}/${oldPath}`])
    }

    // Subir nuevo avatar
    const { error: uploadError } = await supabase.storage
      .from('avatars')
      .upload(fileName, file, { upsert: true })

    if (uploadError) throw uploadError

    // Obtener URL pública
    const { data: { publicUrl } } = supabase.storage
      .from('avatars')
      .getPublicUrl(fileName)

    // Actualizar perfil con nueva URL
    const { error: updateError } = await supabase
      .from('profiles')
      .update({ 
        avatar_url: publicUrl,
        updated_at: new Date().toISOString()
      })
      .eq('id', user.id)

    if (updateError) throw updateError

    profile.value.avatar_url = publicUrl
    successMessage.value = 'Foto de perfil actualizada con éxito!'

    setTimeout(() => {
      successMessage.value = ''
    }, 3000)

  } catch (error) {
    console.error('Error cargando perfil:', error)
    errorMessage.value = 'Error cargando foto:' + error.message
  } finally {
    uploadingAvatar.value = false
    if (fileInput.value) {
      fileInput.value.value = ''
    }
  }
}

const removeAvatar = async () => {
  if (!confirm('Estas seguro de querer borrar la foto?')) return

  try {
    uploadingAvatar.value = true
    errorMessage.value = ''

    const { data: { user } } = await supabase.auth.getUser()
    if (!user) throw new Error('No user found')

    // Eliminar archivo del storage
    if (profile.value.avatar_url) {
      const fileName = profile.value.avatar_url.split('/').pop()
      const { error: deleteError } = await supabase.storage
        .from('avatars')
        .remove([`${user.id}/${fileName}`])

      if (deleteError) console.error('Error deleting file:', deleteError)
    }

    // Actualizar perfil
    const { error: updateError } = await supabase
      .from('profiles')
      .update({ 
        avatar_url: null,
        updated_at: new Date().toISOString()
      })
      .eq('id', user.id)

    if (updateError) throw updateError

    profile.value.avatar_url = ''
    successMessage.value = 'La foto de perfil ha sido eliminada!'

    setTimeout(() => {
      successMessage.value = ''
    }, 3000)

  } catch (error) {
    console.error('Error removing avatar:', error)
    errorMessage.value = 'Error removing photo: ' + error.message
  } finally {
    uploadingAvatar.value = false
  }
}

const formatDate = (dateString) => {
  if (!dateString) return 'N/A'
  const date = new Date(dateString)
  return date.toLocaleDateString('en-US', { 
    year: 'numeric', 
    month: 'long', 
    day: 'numeric' 
  })
}

const handleLogout = async () => {
  await supabase.auth.signOut()
  router.push('/login')
}

onMounted(() => {
  loadProfile()
})
</script>

<style scoped>
.profile-page {
  min-height: 100vh;
  background-color: #f5f5f5;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 24px;
}

.profile-header {
  margin-bottom: 32px;
}

.profile-header h1 {
  font-size: 32px;
  font-weight: 700;
  color: #1f2937;
  margin-bottom: 8px;
}

.profile-header p {
  color: #6b7280;
  font-size: 16px;
}

.profile-content {
  display: grid;
  grid-template-columns: 280px 1fr;
  gap: 24px;
}

.avatar-section {
  background: white;
  border-radius: 12px;
  padding: 32px;
  text-align: center;
  height: fit-content;
}

.avatar-large {
  width: 150px;
  height: 150px;
  border-radius: 50%;
  background: #dbeafe;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto 24px;
  overflow: hidden;
  position: relative;
}

.avatar-large img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.upload-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}

.spinner {
  width: 40px;
  height: 40px;
  border: 4px solid rgba(255, 255, 255, 0.3);
  border-top-color: white;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.change-avatar-btn,
.remove-avatar-btn {
  width: 100%;
  padding: 12px;
  border: none;
  border-radius: 8px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
  margin-bottom: 8px;
}

.change-avatar-btn {
  background: #3b82f6;
  color: white;
}

.change-avatar-btn:hover:not(:disabled) {
  background: #2563eb;
  transform: translateY(-2px);
}

.change-avatar-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.remove-avatar-btn {
  background: #f3f4f6;
  color: #dc2626;
}

.remove-avatar-btn:hover:not(:disabled) {
  background: #fee2e2;
}

.avatar-hint {
  color: #6b7280;
  font-size: 12px;
  margin-top: 8px;
}

.profile-form-section {
  background: white;
  border-radius: 12px;
  padding: 32px;
}

.form-section {
  margin-bottom: 40px;
  padding-bottom: 40px;
  border-bottom: 2px solid #f3f4f6;
}

.form-section:last-child {
  margin-bottom: 0;
  padding-bottom: 0;
  border-bottom: none;
}

.form-section h3 {
  font-size: 20px;
  font-weight: 700;
  color: #1f2937;
  margin-bottom: 24px;
}

.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
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

.disabled-input {
  background: #f9fafb;
  color: #9ca3af;
  cursor: not-allowed;
}

small {
  color: #6b7280;
  font-size: 12px;
  margin-top: 4px;
  display: block;
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

.form-actions {
  display: flex;
  gap: 12px;
  justify-content: flex-end;
  padding-bottom: 16px;
}

.cancel-btn,
.save-btn {
  padding: 12px 32px;
  border-radius: 8px;
  font-weight: 600;
  cursor: pointer;
  border: none;
  transition: all 0.2s;
}

.cancel-btn {
  background: #f3f4f6;
  color: #374151;
}

.cancel-btn:hover {
  background: #e5e7eb;
}

.save-btn {
  background: #3b82f6;
  color: white;
}

.save-btn:hover:not(:disabled) {
  background: #2563eb;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(59, 130, 246, 0.3);
}

.save-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.password-section {
  background: #f9fafb;
  padding: 24px;
  border-radius: 8px;
}

.account-info {
  background: #f9fafb;
  padding: 24px;
  border-radius: 8px;
}

.account-info h3 {
  font-size: 18px;
  font-weight: 700;
  color: #1f2937;
  margin-bottom: 16px;
}

.info-item {
  display: flex;
  justify-content: space-between;
  padding: 12px 0;
  border-bottom: 1px solid #e5e7eb;
}

.info-item:last-child {
  border-bottom: none;
}

.info-label {
  color: #6b7280;
  font-weight: 600;
}

.info-value {
  color: #1f2937;
}

@media (max-width: 768px) {
  .profile-content {
    grid-template-columns: 1fr;
  }

  .form-row {
    grid-template-columns: 1fr;
  }
}
</style>
<template>
  <nav class="navbar">
    <div class="navbar-content">
      <div class="navbar-left">
        <div class="logo">
            <img src="/logo.jpg" alt="LDP MERCHANDISING Logo" width="40" height="40" />
          <span>LDP MERCHANDISING</span>
        </div>
      </div>

      <div class="navbar-center">
        <router-link to="/dashboard" class="nav-link">Inicio</router-link>
      </div>

      <div class="navbar-right">
        <div class="user-menu">
          <button class="user-button" @click="showMenu = !showMenu">
            <div class="avatar">
              <img v-if="avatarUrl" :src="avatarUrl" alt="Avatar" />
              <svg v-else width="24" height="24" viewBox="0 0 24 24" fill="none">
                <circle cx="12" cy="8" r="4" fill="#3b82f6"/>
                <path d="M4 20c0-4 3-6 8-6s8 2 8 6" stroke="#3b82f6" stroke-width="2" stroke-linecap="round"/>
              </svg>
            </div>
          </button>
          
          <div v-if="showMenu" class="dropdown-menu">
            <router-link to="/profile" class="menu-item">Perfíl</router-link>
            <div class="menu-divider"></div>
            <a href="#" @click.prevent="$emit('logout')" class="menu-item">Cerrar Sesión</a>
          </div>
        </div>
      </div>
    </div>

  </nav>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import { supabase } from '../lib/supabase'

defineEmits(['logout'])

const showMenu = ref(false)
const avatarUrl = ref('')

const loadUserAvatar = async () => {
  try {
    const { data: { user } } = await supabase.auth.getUser()
    if (!user) return

    const { data } = await supabase
      .from('profiles')
      .select('avatar_url')
      .eq('id', user.id)
      .single()

    if (data?.avatar_url) {
      avatarUrl.value = data.avatar_url
    }
  } catch (error) {
    console.error('Error cargando avatar:', error)
  }
}

const onAvatarUpdated = (e) => {
  if (e?.detail && typeof e.detail.avatar_url !== 'undefined') {
    avatarUrl.value = e.detail.avatar_url || ''
  }
}

onMounted(() => {
  loadUserAvatar()
  window.addEventListener('profile:avatar-updated', onAvatarUpdated)
})

onUnmounted(() => {
  window.removeEventListener('profile:avatar-updated', onAvatarUpdated)
})
</script>

<style scoped>
.navbar {
  background: white;
  border-bottom: 1px solid #e5e7eb;
  padding: 0 118px;
}

.navbar-content {
  max-width: 1400px;
  margin: 0 auto;
  display: flex;
  align-items: center;
  justify-content: space-between;
  height: 70px;
}

.navbar-left {
  display: flex;
  align-items: center;
}

.logo {
  display: flex;
  align-items: center;
  gap: 12px;
  font-size: 24px;
  font-weight: 700;
  color: #1f2937;
}

.navbar-center {
  display: flex;
  gap: 32px;
}

.nav-link {
  text-decoration: none;
  color: #6b7280;
  font-weight: 600;
  font-size: 16px;
  transition: color 0.2s;
}

.nav-link:hover {
  color: #3b82f6;
}

.navbar-right {
  display: flex;
  align-items: center;
  gap: 16px;
}

/* responsive rules */
@media (max-width: 768px) {
  .navbar {
    padding: 0 20px;
  }
  .navbar-content { height: 64px }
}

.icon-button {
  width: 40px;
  height: 40px;
  border-radius: 8px;
  border: none;
  background: #f9fafb;
  color: #6b7280;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
}

.icon-button:hover {
  background: #f3f4f6;
  color: #374151;
}

.user-menu {
  position: relative;
}

.user-button {
  background: none;
  border: none;
  cursor: pointer;
  padding: 0;
}

.avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: #dbeafe;
  display: flex;
  align-items: center;
  justify-content: center;
}

.avatar img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 50%;
}

.dropdown-menu {
  position: absolute;
  top: 50px;
  right: 0;
  background: white;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  min-width: 200px;
  padding: 8px;
  z-index: 100;
}

.menu-item {
  display: block;
  padding: 12px 16px;
  color: #374151;
  text-decoration: none;
  border-radius: 6px;
  transition: background 0.2s;
}

.menu-item:hover {
  background: #f9fafb;
}

.menu-divider {
  height: 1px;
  background: #e5e7eb;
  margin: 8px 0;
}
</style>
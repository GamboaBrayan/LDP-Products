<template>
  <div class="dashboard">
    <Navbar @logout="handleLogout" />
    
    <div class="container">
      <div class="header">
        <h1>Productos</h1>
        <button class="add-button" @click="showAddModal = true">
          <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
            <path d="M10 4v12M4 10h12" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
          </svg>
          Añadir nuevo Producto
        </button>
      </div>

      <div class="content-wrapper">
        <div class="main-content">
          <div class="filters">
            <div class="search-box">
              <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
                <path d="M9 17A8 8 0 1 0 9 1a8 8 0 0 0 0 16zM19 19l-4.35-4.35" stroke="#9ca3af" stroke-width="2" stroke-linecap="round"/>
              </svg>
              <input 
                type="text" 
                placeholder="Buscar por nombre de producto"
                v-model="searchQuery"
              />
            </div>

            <select v-model="filterCategory" class="filter-select">
              <option value="">Todos</option>
              <option value="Impresiones">Impresiones</option>
              <option value="Empastado">Empastado</option>
              <option value="Lapiceros">Lapiceros</option>
            </select>


          </div>

          <div class="table-container">
            <h2 class="table-title">Lista de Precios</h2>
            
            <table class="products-table">
              <thead>
                <tr>
                  <th>Nombre</th>
                  <th>Precio unidad</th>
                  <th>Cantidad</th>
                  <th>Total</th>
                  <th>Acciones</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="product in paginatedProducts" :key="product.id">
                  <td class="product-name">{{ product.name }}</td>
                  <td>${{ product.unit_price.toFixed(2) }}</td>
                  <td>
                    <input 
                      type="number" 
                      :value="product.quantity" 
                      @change="updateQuantity(product.id, $event.target.value)"
                      class="quantity-input"
                      min="1"
                    />
                  </td>
                  <td class="total-price">${{ (product.unit_price * product.quantity).toFixed(2) }}</td>
                  <td class="actions-cell">
                    <button @click="resetQuantity(product.id)" class="reset-btn">Reset</button>
                    <button @click="deleteProduct(product.id)" class="delete-btn">Borrar</button>
                  </td>
                </tr>
              </tbody>
            </table>

            <div class="pagination">
              <span>Mostrando {{ startIndex }} - {{ endIndex }} of {{ filteredProducts.length }}</span>
              <div class="pagination-buttons">
                <button @click="prevPage" :disabled="currentPage === 1">&lt;</button>
                <button v-for="page in totalPages" :key="page" @click="goToPage(page)" :class="{ active: page === currentPage }">{{ page }}</button>
                <button @click="nextPage" :disabled="currentPage === totalPages || totalPages === 0">&gt;</button>
              </div>
            </div>
          </div>
        </div>

        <div class="sidebar">
          <h3>Acciones Rapidas</h3>
          
          <a href="https://www.ilovepdf.com/es/pdf_a_word" target="_blank"  class="resource-link">
            <span>Convertir PDF a WORD</span>
            <svg width="16" height="16" viewBox="0 0 16 16" fill="none">
              <path d="M6 4l4 4-4 4" stroke="#9ca3af" stroke-width="2" stroke-linecap="round"/>
            </svg>
          </a>

          <a href="https://www.ilovepdf.com/es/word_a_pdf" target="_blank" class="resource-link">
            <span>Convertir WORD a PDF</span>
            <svg width="16" height="16" viewBox="0 0 16 16" fill="none">
              <path d="M6 4l4 4-4 4" stroke="#9ca3af" stroke-width="2" stroke-linecap="round"/>
            </svg>
          </a>

          <a href="https://www.ilovepdf.com/es/excel_a_pdf" target="_blank"  class="resource-link">
            <span>Convertir EXCEL a PDF</span>
            <svg width="16" height="16" viewBox="0 0 16 16" fill="none">
              <path d="M6 4l4 4-4 4" stroke="#9ca3af" stroke-width="2" stroke-linecap="round"/>
            </svg>
          </a>

          <a href="https://www.ilovepdf.com/es/powerpoint_a_pdf" target="_blank"  class="resource-link">
            <span>Convertir PowerPoint a PDF</span>
            <svg width="16" height="16" viewBox="0 0 16 16" fill="none">
              <path d="M6 4l4 4-4 4" stroke="#9ca3af" stroke-width="2" stroke-linecap="round"/>
            </svg>
          </a>
        </div>
      </div>
    </div>

    <!-- Modal para agregar producto -->
    <div v-if="showAddModal" class="modal-overlay" @click="showAddModal = false">
      <div class="modal" @click.stop>
        <h2>Añadir nuevo Producto</h2>
        <form @submit.prevent="addProduct">
          <div class="form-group">
            <label>Nombre del Producto</label>
            <input type="text" v-model="newProduct.name" required />
          </div>
          <div class="form-group">
            <label>Categoria</label>
            <select v-model="newProduct.category" required>
              <option value="">Select category</option>
              <option value="Impresiones">Impresiones</option>
              <option value="Empastado">Empastado</option>
              <option value="Lapiceros">Lapiceros</option>
            </select>
          </div>
          <div class="form-group">
            <label>Precio Unico</label>
            <input type="number" step="0.01" v-model="newProduct.unit_price" required />
          </div>
          <div class="form-group">
            <label>Cantidad</label>
            <input type="number" v-model="newProduct.quantity" required />
          </div>
          <div class="modal-buttons">
            <button type="button" @click="showAddModal = false" class="cancel-btn">Cancelar</button>
            <button type="submit" class="submit-btn">Añadir Producto</button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import { useRouter } from 'vue-router'
import { supabase } from '../lib/supabase'
import Navbar from '../components/Navbar.vue'

const router = useRouter()
const products = ref([])
const searchQuery = ref('')
const filterCategory = ref('')
const filterStatus = ref('')
const showAddModal = ref(false)

const newProduct = ref({
  name: '',
  sku: '',
  category: '',
  unit_price: 0,
  quantity: 1,
  status: 'active'
})

const currentPage = ref(1)
const itemsPerPage = 16

const filteredProducts = computed(() => {
  return products.value.filter(product => {
    const matchesSearch = product.name.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
                         (product.sku && product.sku.toLowerCase().includes(searchQuery.value.toLowerCase()))
    const matchesCategory = !filterCategory.value || product.category === filterCategory.value
    const matchesStatus = !filterStatus.value || product.status === filterStatus.value
    
    return matchesSearch && matchesCategory && matchesStatus
  })
})

const totalPages = computed(() => Math.ceil(filteredProducts.value.length / itemsPerPage))

const paginatedProducts = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage
  return filteredProducts.value.slice(start, start + itemsPerPage)
})

const startIndex = computed(() => {
  if (filteredProducts.value.length === 0) return 0
  return (currentPage.value - 1) * itemsPerPage + 1
})

const endIndex = computed(() => Math.min(currentPage.value * itemsPerPage, filteredProducts.value.length))

const goToPage = (page) => {
  if (page < 1) page = 1
  if (page > totalPages.value) page = totalPages.value || 1
  currentPage.value = page
}

const prevPage = () => { if (currentPage.value > 1) currentPage.value-- }
const nextPage = () => { if (currentPage.value < totalPages.value) currentPage.value++ }

const loadProducts = async () => {
  try {
    const { data, error } = await supabase
      .from('products')
      .select('*')
      .order('created_at', { ascending: false })
    
    if (error) throw error
    products.value = data
  } catch (error) {
    console.error('Error loading products:', error)
  }
}

const addProduct = async () => {
  try {
    const { data: { user } } = await supabase.auth.getUser()

    // Auto-generate SKU if not provided. Format: SKU-0001, SKU-0002, ...
    if (!newProduct.value.sku) {
      let maxNum = 0
      products.value.forEach(p => {
        if (p && p.sku) {
          const nums = String(p.sku).match(/\d+/g)
          if (nums && nums.length) {
            const num = parseInt(nums[nums.length - 1], 10)
            if (!isNaN(num) && num > maxNum) maxNum = num
          }
        }
      })
      const next = maxNum + 1
      newProduct.value.sku = 'SKU-' + String(next).padStart(4, '0')
    }

    const { data, error } = await supabase
      .from('products')
      .insert([{
        ...newProduct.value,
        user_id: user.id
      }])
      .select()

    if (error) throw error

    products.value.unshift(data[0])
    showAddModal.value = false

    // Reset form
    newProduct.value = {
      name: '',
      sku: '',
      category: '',
      unit_price: 0,
      quantity: 1,
      status: 'active'
    }
  } catch (error) {
    console.error('Error adding product:', error)
    alert('Error adding product: ' + error.message)
  }
}

const updateQuantity = async (id, newQuantity) => {
  try {
    const { error } = await supabase
      .from('products')
      .update({ quantity: parseInt(newQuantity) })
      .eq('id', id)
    
    if (error) throw error
    
    const product = products.value.find(p => p.id === id)
    if (product) {
      product.quantity = parseInt(newQuantity)
    }
  } catch (error) {
    console.error('Error updating quantity:', error)
  }
}

const resetQuantity = async (id) => {
  try {
    const { error } = await supabase
      .from('products')
      .update({ quantity: 1 })
      .eq('id', id)

    if (error) throw error

    const product = products.value.find(p => p.id === id)
    if (product) product.quantity = 1
  } catch (error) {
    console.error('Error resetting quantity:', error)
    alert('Error resetting quantity: ' + error.message)
  }
}

const deleteProduct = async (id) => {
  if (!confirm('Are you sure you want to delete this product?')) return
  
  try {
    const { error } = await supabase
      .from('products')
      .delete()
      .eq('id', id)
    
    if (error) throw error
    
    products.value = products.value.filter(p => p.id !== id)
  } catch (error) {
    console.error('Error deleting product:', error)
  }
}

const handleLogout = async () => {
  await supabase.auth.signOut()
  router.push('/login')
}

// When filters/search or product list change, reset or clamp current page
watch([searchQuery, filterCategory, filterStatus, () => products.value.length], () => {
  if (currentPage.value > totalPages.value) currentPage.value = totalPages.value || 1
  if (currentPage.value < 1) currentPage.value = 1
})

onMounted(() => {
  loadProducts()
})
</script>

<style scoped>
.dashboard {
  min-height: 100vh;
  background-color: #f5f5f5;
}

.container {
  max-width: 1400px;
  margin: 0 auto;
  padding: 24px;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 32px;
}

.header h1 {
  font-size: 32px;
  font-weight: 700;
  color: #1f2937;
}

.add-button {
  display: flex;
  align-items: center;
  gap: 8px;
  background: #3b82f6;
  color: white;
  border: none;
  padding: 12px 24px;
  border-radius: 8px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
}

.add-button:hover {
  background: #2563eb;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(59, 130, 246, 0.3);
}

.content-wrapper {
  display: grid;
  grid-template-columns: 1fr 300px;
  gap: 24px;
}

.main-content {
  background: white;
  border-radius: 12px;
  padding: 24px;
}

.filters {
  display: flex;
  gap: 12px;
  margin-bottom: 24px;
}

.search-box {
  flex: 1;
  position: relative;
  display: flex;
  align-items: center;
}

.search-box svg {
  position: absolute;
  left: 16px;
}

.search-box input {
  width: 100%;
  padding: 12px 12px 12px 48px;
  border: 2px solid #e5e7eb;
  border-radius: 8px;
  font-size: 14px;
}

.filter-select {
  padding: 12px 16px;
  border: 2px solid #e5e7eb;
  border-radius: 8px;
  font-size: 14px;
  background: white;
  cursor: pointer;
}

.table-title {
  font-size: 20px;
  font-weight: 700;
  color: #1f2937;
  margin-bottom: 24px;
}

.products-table {
  width: 100%;
  border-collapse: collapse;
}

.products-table th {
  text-align: left;
  padding: 16px;
  font-size: 12px;
  font-weight: 700;
  color: #6b7280;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  border-bottom: 2px solid #f3f4f6;
}

.products-table td {
  padding: 20px 16px;
  border-bottom: 1px solid #f3f4f6;
  color: #1f2937;
}

.product-name {
  font-weight: 600;
}

.total-price {
  color: #3b82f6;
  font-weight: 700;
  font-size: 18px;
}

.quantity-input {
  width: 80px;
  padding: 8px 12px;
  border: 2px solid #e5e7eb;
  border-radius: 6px;
  text-align: center;
}

.delete-btn {
  background: #ef4444;
  color: white;
  border: none;
  padding: 6px 12px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

.delete-btn:hover {
  background: #dc2626;
}

.reset-btn {
  background: #f59e0b; /* amber */
  color: white;
  border: none;
  padding: 6px 10px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  margin-right: 8px;
}

.reset-btn:hover {
  background: #d97706;
} 
.actions-cell {
  display: flex;
  gap: 8px;
}

.pagination {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 24px;
  padding-top: 24px;
  border-top: 2px solid #f3f4f6;
}

.pagination-buttons {
  display: flex;
  gap: 8px;
}

.pagination-buttons button {
  padding: 8px 12px;
  border: 2px solid #e5e7eb;
  background: white;
  border-radius: 6px;
  cursor: pointer;
  font-weight: 600;
}

.pagination-buttons button.active {
  background: #3b82f6;
  color: white;
  border-color: #3b82f6;
}

.sidebar {
  background: white;
  border-radius: 12px;
  padding: 24px;
  height: fit-content;
}

.sidebar h3 {
  font-size: 18px;
  font-weight: 700;
  color: #1f2937;
  margin-bottom: 20px;
}

.resource-link {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 16px;
  border-radius: 8px;
  text-decoration: none;
  color: #1f2937;
  margin-bottom: 8px;
  transition: background 0.2s;
}

.resource-link:hover {
  background: #f9fafb;
}

.resource-icon {
  width: 40px;
  height: 40px;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.resource-link span {
  flex: 1;
  font-weight: 500;
  font-size: 14px;
}

/* Modal */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal {
  background: white;
  border-radius: 12px;
  padding: 32px;
  width: 90%;
  max-width: 500px;
}

.modal h2 {
  font-size: 24px;
  font-weight: 700;
  margin-bottom: 24px;
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  font-weight: 600;
  color: #374151;
  margin-bottom: 8px;
  font-size: 14px;
}

.form-group input,
.form-group select {
  width: 100%;
  padding: 12px 16px;
  border: 2px solid #e5e7eb;
  border-radius: 8px;
  font-size: 16px;
}

.modal-buttons {
  display: flex;
  gap: 12px;
  margin-top: 24px;
}

.cancel-btn,
.submit-btn {
  flex: 1;
  padding: 12px;
  border-radius: 8px;
  font-weight: 600;
  cursor: pointer;
  border: none;
}

.cancel-btn {
  background: #f3f4f6;
  color: #374151;
}

.submit-btn {
  background: #3b82f6;
  color: white;
}

.submit-btn:hover {
  background: #2563eb;
}
</style>
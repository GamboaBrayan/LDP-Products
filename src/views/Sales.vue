<template>
  <div class="sales-page">
    <Navbar @logout="handleLogout" />
    
    <div class="container">
      <div class="header">
        <h1>Registro de Ventas</h1>
        <button class="add-button" @click="showAddModal = true">
          <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
            <path d="M10 4v12M4 10h12" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
          </svg>
          Agregar Venta
        </button>
      </div>

      <div class="content">
        <div class="filters">
          <div class="search-box">
            <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
              <path d="M9 17A8 8 0 1 0 9 1a8 8 0 0 0 0 16zM19 19l-4.35-4.35" stroke="#9ca3af" stroke-width="2" stroke-linecap="round"/>
            </svg>
            <input 
              type="text" 
              placeholder="Buscar por nombre de cliente"
              v-model="searchQuery"
            />
          </div>

          <div class="date-filter">
            <label>Filtrar por fecha:</label>
            <input 
              type="date" 
              v-model="filterDate"
              class="date-input"
            />
            <button v-if="filterDate" @click="clearDateFilter" class="clear-filter">
              <svg width="16" height="16" viewBox="0 0 16 16" fill="none">
                <path d="M12 4L4 12M4 4l8 8" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
              </svg>
            </button>
          </div>
        </div>

        <!-- Tabla de Ventas -->
        <div class="table-container">
          <div class="table-header-section">
            <h2 class="table-title">Historial de Ventas</h2>
            <div class="total-sales">
              <span class="total-label">Total Ventas:</span>
              <span class="total-amount">S/. {{ totalSales.toFixed(2) }}</span>
            </div>
          </div>
          
          <table class="sales-table">
            <thead>
              <tr>
                <th>FECHA</th>
                <th>CLIENTE</th>
                <th>DESCRIPCIÓN</th>
                <th>PRECIO</th>
                <th>ACCIONES</th>
              </tr>
            </thead>
            <tbody>
              <tr v-if="filteredSales.length === 0">
                <td colspan="5" class="no-data">
                  <div class="no-data-content">
                    <svg width="48" height="48" viewBox="0 0 48 48" fill="none">
                      <path d="M24 44c11.046 0 20-8.954 20-20S35.046 4 24 4 4 12.954 4 24s8.954 20 20 20z" stroke="#d1d5db" stroke-width="2"/>
                      <path d="M24 16v12M24 32h.01" stroke="#d1d5db" stroke-width="2" stroke-linecap="round"/>
                    </svg>
                    <p>No hay ventas registradas</p>
                  </div>
                </td>
              </tr>
              <tr v-for="sale in filteredSales" :key="sale.id" class="sale-row">
                <td class="date-cell" data-label="FECHA">{{ formatDate(sale.created_at) }}</td>
                <td class="customer-name" data-label="CLIENTE">{{ sale.customer_name }}</td>
                <td class="description" data-label="DESCRIPCIÓN">{{ sale.description || '-' }}</td>
                <td class="price" data-label="PRECIO">S/. {{ sale.price.toFixed(2) }}</td>
                <td data-label="ACCIONES">
                  <div class="actions">
                    <button @click="editSale(sale)" class="edit-btn" title="Editar">
                      <svg width="16" height="16" viewBox="0 0 16 16" fill="none">
                        <path d="M11.333 2A2.827 2.827 0 0 1 14 4.667L5.5 13.167l-4.167.833.833-4.167L11.333 2z" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
                      </svg>
                    </button>
                    <button @click="openDeleteModal(sale)" class="delete-btn" title="Eliminar">
                      <svg width="16" height="16" viewBox="0 0 16 16" fill="none">
                        <path d="M2 4h12M5.333 4V2.667a1.333 1.333 0 0 1 1.334-1.334h2.666a1.333 1.333 0 0 1 1.334 1.334V4m2 0v9.333a1.333 1.333 0 0 1-1.334 1.334H4.667a1.333 1.333 0 0 1-1.334-1.334V4h9.334z" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
                      </svg>
                    </button>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>

          <div class="pagination">
            <span>Mostrando {{ filteredSales.length }} de {{ sales.length }} ventas</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Modal para confirmar eliminación -->
    <div v-if="showDeleteModal" class="modal-overlay" @click="closeDeleteModal">
      <div class="modal" @click.stop>
        <div class="modal-header">
          <h2>Confirmar Eliminación</h2>
          <button class="close-btn" @click="closeDeleteModal">
            <svg width="24" height="24" viewBox="0 0 24 24" fill="none">
              <path d="M18 6L6 18M6 6l12 12" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
            </svg>
          </button>
        </div>

        <div class="modal-content">
          <p class="confirmation-message">
            ¿Estás seguro que deseas eliminar la venta de <strong>{{ saleToDelete?.customer_name }}</strong>?
          </p>
          <p class="sale-details">
            <span class="detail-label">Fecha:</span> {{ saleToDelete ? formatDate(saleToDelete.created_at) : '' }}<br>
            <span class="detail-label">Monto:</span> S/. {{ saleToDelete?.price.toFixed(2) }}
          </p>
          
          <div class="modal-buttons">
            <button type="button" @click="closeDeleteModal" class="cancel-btn">Cancelar</button>
            <button 
              type="button" 
              class="delete-confirm-btn" 
              :disabled="deleting"
              @click="confirmDelete"
            >
              {{ deleting ? 'Eliminando...' : 'Eliminar Venta' }}
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Modal para agregar/editar venta -->
    <div v-if="showAddModal" class="modal-overlay" @click="closeModal">
      <div class="modal" @click.stop>
        <div class="modal-header">
          <h2>{{ editingId ? 'Editar Venta' : 'Nueva Venta' }}</h2>
          <button class="close-btn" @click="closeModal">
            <svg width="24" height="24" viewBox="0 0 24 24" fill="none">
              <path d="M18 6L6 18M6 6l12 12" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
            </svg>
          </button>
        </div>

        <form @submit.prevent="saveSale">
          <div class="form-group">
            <label>Nombre del Cliente *</label>
            <input 
              type="text" 
              v-model="newSale.customer_name" 
              placeholder="Ingrese el nombre del cliente"
              required
            />
          </div>

          <div class="form-group">
            <label>Descripción de la Venta</label>
            <textarea 
              v-model="newSale.description" 
              placeholder="Opcional: Detalles de la venta"
              rows="3"
            ></textarea>
          </div>

          <div class="form-group">
            <label>Precio de la Venta *</label>
            <div class="price-input">
              <span class="currency">S/.</span>
              <input 
                type="number" 
                step="0.01" 
                min="0"
                v-model="newSale.price" 
                placeholder="0.00"
                required
              />
            </div>
          </div>

          <div v-if="errorMessage" class="error-message">
            {{ errorMessage }}
          </div>

          <div class="modal-buttons">
            <button type="button" @click="closeModal" class="cancel-btn">Cancelar</button>
            <button type="submit" class="submit-btn" :disabled="loading">
              {{ loading ? 'Guardando...' : (editingId ? 'Actualizar' : 'Guardar') }}
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { supabase } from '../lib/supabase'
import Navbar from '../components/Navbar.vue'

const router = useRouter()
const sales = ref([])
const searchQuery = ref('')
const filterDate = ref('')
const showAddModal = ref(false)
const loading = ref(false)
const errorMessage = ref('')
const showDeleteModal = ref(false)
const saleToDelete = ref(null)
const deleting = ref(false)
const editingId = ref(null)

const newSale = ref({
  customer_name: '',
  description: '',
  price: 0
})

const filteredSales = computed(() => {
  return sales.value.filter(sale => {
    const matchesSearch = sale.customer_name.toLowerCase().includes(searchQuery.value.toLowerCase())
    
    let matchesDate = true
    if (filterDate.value) {
      const saleDate = new Date(sale.created_at).toISOString().split('T')[0]
      matchesDate = saleDate === filterDate.value
    }
    
    return matchesSearch && matchesDate
  })
})

const totalSales = computed(() => {
  return filteredSales.value.reduce((sum, sale) => sum + parseFloat(sale.price), 0)
})

const loadSales = async () => {
  try {
    const { data, error } = await supabase
      .from('sales')
      .select('*')
      .order('created_at', { ascending: false })
    
    if (error) throw error
    sales.value = data
  } catch (error) {
    console.error('Error loading sales:', error)
  }
}

const saveSale = async () => {
  try {
    loading.value = true
    errorMessage.value = ''

    const { data: { user } } = await supabase.auth.getUser()

    if (editingId.value) {
      // Actualizar venta existente
      const { error } = await supabase
        .from('sales')
        .update({
          customer_name: newSale.value.customer_name,
          description: newSale.value.description,
          price: newSale.value.price
        })
        .eq('id', editingId.value)

      if (error) throw error

      const index = sales.value.findIndex(s => s.id === editingId.value)
      if (index !== -1) {
        sales.value[index] = {
          ...sales.value[index],
          ...newSale.value
        }
      }
    } else {
      // Crear nueva venta
      const { data, error } = await supabase
        .from('sales')
        .insert([{
          customer_name: newSale.value.customer_name,
          description: newSale.value.description,
          price: newSale.value.price,
          user_id: user.id
        }])
        .select()

      if (error) throw error

      sales.value.unshift(data[0])
    }

    closeModal()
  } catch (error) {
    console.error('Error saving sale:', error)
    errorMessage.value = 'Error al guardar la venta: ' + error.message
  } finally {
    loading.value = false
  }
}

const editSale = (sale) => {
  editingId.value = sale.id
  newSale.value = {
    customer_name: sale.customer_name,
    description: sale.description || '',
    price: sale.price
  }
  showAddModal.value = true
}

const openDeleteModal = (sale) => {
  saleToDelete.value = sale
  showDeleteModal.value = true
}

const closeDeleteModal = () => {
  showDeleteModal.value = false
  saleToDelete.value = null
  deleting.value = false
}

const confirmDelete = async () => {
  if (!saleToDelete.value) return
  
  try {
    deleting.value = true
    const { error } = await supabase
      .from('sales')
      .delete()
      .eq('id', saleToDelete.value.id)

    if (error) throw error

    sales.value = sales.value.filter(s => s.id !== saleToDelete.value.id)
    closeDeleteModal()
  } catch (error) {
    console.error('Error deleting sale:', error)
    alert('Error al eliminar la venta: ' + error.message)
  } finally {
    deleting.value = false
  }
}

const closeModal = () => {
  showAddModal.value = false
  editingId.value = null
  newSale.value = {
    customer_name: '',
    description: '',
    price: 0
  }
  errorMessage.value = ''
}

const clearDateFilter = () => {
  filterDate.value = ''
}

const formatDate = (dateString) => {
  const date = new Date(dateString)
  return date.toLocaleString('es-ES', {
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit'
  })
}

const handleLogout = async () => {
  await supabase.auth.signOut()
  router.push('/login')
}

onMounted(() => {
  loadSales()
})
</script>

<style scoped>
.sales-page {
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
  background: #10b981;
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
  background: #059669;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(16, 185, 129, 0.3);
}

.content {
  background: white;
  border-radius: 12px;
  padding: 24px;
}

.filters {
  display: flex;
  gap: 16px;
  margin-bottom: 24px;
  flex-wrap: wrap;
}

.search-box {
  flex: 1;
  min-width: 250px;
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

.date-filter {
  display: flex;
  align-items: center;
  gap: 12px;
}

.date-filter label {
  font-weight: 600;
  color: #374151;
  font-size: 14px;
  white-space: nowrap;
}

.date-input {
  padding: 12px 16px;
  border: 2px solid #e5e7eb;
  border-radius: 8px;
  font-size: 14px;
}

.clear-filter {
  padding: 8px;
  background: #fee2e2;
  color: #dc2626;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}

.clear-filter:hover {
  background: #fecaca;
}

.table-container {
  overflow-x: auto;
}

.table-header-section {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24px;
}

.table-title {
  font-size: 20px;
  font-weight: 700;
  color: #1f2937;
}

.total-sales {
  display: flex;
  align-items: center;
  gap: 12px;
  background: #f0fdf4;
  padding: 12px 24px;
  border-radius: 8px;
}

.total-label {
  font-weight: 600;
  color: #166534;
}

.total-amount {
  font-size: 24px;
  font-weight: 700;
  color: #16a34a;
}

.sales-table {
  width: 100%;
  border-collapse: collapse;
}

.sales-table th {
  text-align: left;
  padding: 16px;
  font-size: 12px;
  font-weight: 700;
  color: #6b7280;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  border-bottom: 2px solid #f3f4f6;
}

.sales-table td {
  padding: 20px 16px;
  border-bottom: 1px solid #f3f4f6;
  color: #1f2937;
}

.sale-row:hover {
  background: #f9fafb;
}

.date-cell {
  color: #6b7280;
  font-size: 14px;
}

.customer-name {
  font-weight: 600;
}

.description {
  color: #6b7280;
  max-width: 300px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.price {
  color: #10b981;
  font-weight: 700;
  font-size: 18px;
}

.actions {
  display: flex;
  gap: 8px;
}

.edit-btn,
.delete-btn {
  padding: 8px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
}

.edit-btn {
  background: #dbeafe;
  color: #1e40af;
}

.edit-btn:hover {
  background: #bfdbfe;
}

.delete-btn {
  background: #fee2e2;
  color: #dc2626;
}

.delete-btn:hover {
  background: #fecaca;
}

.no-data {
  text-align: center;
  padding: 60px 20px !important;
}

.no-data-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
}

.no-data-content p {
  color: #9ca3af;
  font-size: 16px;
}

.pagination {
  display: flex;
  justify-content: center;
  margin-top: 24px;
  padding-top: 24px;
  border-top: 2px solid #f3f4f6;
  color: #6b7280;
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
  padding: 0;
  width: 90%;
  max-width: 500px;
  max-height: 90vh;
  overflow-y: auto;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 24px;
  border-bottom: 2px solid #f3f4f6;
}

.modal-header h2 {
  font-size: 24px;
  font-weight: 700;
}

.close-btn {
  background: none;
  border: none;
  color: #9ca3af;
  cursor: pointer;
  padding: 4px;
  display: flex;
  align-items: center;
}

.close-btn:hover {
  color: #374151;
}

.modal form {
  padding: 24px;
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
.form-group textarea {
  width: 100%;
  padding: 12px 16px;
  border: 2px solid #e5e7eb;
  border-radius: 8px;
  font-size: 16px;
  font-family: inherit;
}

.form-group input:focus,
.form-group textarea:focus {
  outline: none;
  border-color: #10b981;
  box-shadow: 0 0 0 3px rgba(16, 185, 129, 0.1);
}

.price-input {
  position: relative;
}

.currency {
  position: absolute;
  left: 16px;
  top: 50%;
  transform: translateY(-50%);
  font-weight: 600;
  color: #6b7280;
  font-size: 18px;
}

.price-input input {
  padding-left: 36px;
}

/* Delete confirmation modal styles */
.modal-content {
  padding: 24px;
}

.confirmation-message {
  color: #1f2937;
  font-size: 16px;
  margin-bottom: 16px;
}

.sale-details {
  background: #f3f4f6;
  padding: 16px;
  border-radius: 8px;
  margin-bottom: 24px;
  color: #4b5563;
  font-size: 14px;
  line-height: 1.6;
}

.detail-label {
  font-weight: 600;
  color: #374151;
  margin-right: 8px;
}

.delete-confirm-btn {
  flex: 1;
  padding: 12px;
  border-radius: 8px;
  font-weight: 600;
  cursor: pointer;
  border: none;
  font-size: 16px;
  background: #ef4444;
  color: white;
}

.delete-confirm-btn:hover:not(:disabled) {
  background: #dc2626;
}

.delete-confirm-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.error-message {
  background-color: #fee2e2;
  color: #dc2626;
  padding: 12px;
  border-radius: 8px;
  margin-bottom: 16px;
  font-size: 14px;
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
  font-size: 16px;
}

.cancel-btn {
  background: #f3f4f6;
  color: #374151;
}

.cancel-btn:hover {
  background: #e5e7eb;
}

.submit-btn {
  background: #10b981;
  color: white;
}

.submit-btn:hover:not(:disabled) {
  background: #059669;
}

.submit-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}
</style>

/* Responsive: convertir la tabla en tarjetas en pantallas pequeñas */
@media (max-width: 768px) {
  .container {
    padding: 16px;
  }

  .header {
    flex-direction: column;
    align-items: flex-start;
    gap: 12px;
    margin-bottom: 20px;
  }

  .header h1 {
    font-size: 24px;
  }

  .add-button {
    width: 100%;
    justify-content: center;
    padding: 12px;
  }

  .filters {
    flex-direction: column;
    gap: 12px;
  }

  .date-filter {
    width: 100%;
    justify-content: space-between;
  }

  .table-header-section {
    flex-direction: column;
    align-items: flex-start;
    gap: 12px;
  }

  .total-sales {
    width: 100%;
    justify-content: space-between;
  }

  /* Hide table header and display rows as cards */
  .sales-table thead {
    display: none;
  }

  .sales-table,
  .sales-table tbody,
  .sales-table tr,
  .sales-table td {
    display: block;
    width: 100%;
  }

  .sales-table tr {
    margin-bottom: 12px;
    border: 1px solid #f3f4f6;
    border-radius: 8px;
    padding: 12px;
    background: #fff;
  }

  .sales-table td {
    padding: 8px 12px;
    border-bottom: none;
  }

  .sales-table td::before {
    content: attr(data-label);
    display: block;
    font-weight: 700;
    color: #6b7280;
    margin-bottom: 6px;
    font-size: 13px;
    text-transform: uppercase;
    letter-spacing: 0.4px;
  }

  .description {
    white-space: normal;
    max-width: none;
    overflow: visible;
    text-overflow: unset;
  }

  .actions {
    justify-content: flex-end;
  }

  .modal {
    width: 96%;
    max-width: 560px;
  }
}
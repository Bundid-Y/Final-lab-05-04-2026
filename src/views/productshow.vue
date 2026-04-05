<template>
  <section class="product-show-page">
    <div class="container py-5">
      <div class="headline mb-4">
        <div>
          <p class="eyebrow mb-2">Inventory Portal</p>
          <h2 class="title mb-1">Webhook Data Viewer</h2>
          <p class="subtitle mb-0">Showing all data from n8n webhook: /webhook/formdata</p>
        </div>
        <button class="btn reload-btn" @click="fetchProducts" :disabled="loading">
          <span v-if="loading" class="spinner-border spinner-border-sm me-2"></span>
          {{ loading ? 'Loading...' : 'Reload Data' }}
        </button>
      </div>

      <div class="table-shell">
        <div v-if="errorMessage" class="alert alert-danger mb-4">
          {{ errorMessage }}
        </div>

        <div v-if="loading" class="state-block">
          <div class="spinner-border text-primary"></div>
          <p class="mt-2 mb-0">Loading product data...</p>
        </div>

        <div v-else-if="products.length" class="table-responsive">
          <table class="table product-table align-middle text-center mb-0">
            <thead>
              <tr>
                <th>#</th>
                <th v-for="column in columns" :key="column">
                  {{ formatColumnLabel(column) }}
                </th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(item, index) in products" :key="index">
                <td class="row-index">{{ index + 1 }}</td>
                <td
                  v-for="column in columns"
                  :key="`${index}-${column}`"
                  :class="getCellClass(column)"
                >
                  {{ formatCellValue(item[column], column) }}
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <div v-else class="state-block">
          <p class="mb-0">No product data found.</p>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup>
import { onMounted, ref } from 'vue'

const WEBHOOK_URL = 'http://localhost:5678/webhook/formdata'

const products = ref([])
const columns = ref([])
const loading = ref(false)
const errorMessage = ref('')

const normalizeProducts = (payload) => {
  if (Array.isArray(payload)) {
    return payload
      .map((item) => (item?.json && typeof item.json === 'object' ? item.json : item))
      .filter((item) => item && typeof item === 'object' && !Array.isArray(item))
  }

  if (Array.isArray(payload?.data)) {
    return normalizeProducts(payload.data)
  }

  if (Array.isArray(payload?.items)) {
    return normalizeProducts(payload.items)
  }

  if (payload?.json && typeof payload.json === 'object') {
    return normalizeProducts(payload.json)
  }

  if (payload && typeof payload === 'object') {
    return [payload]
  }

  return []
}

const buildColumns = (items) => {
  const keySet = new Set()

  items.forEach((item) => {
    Object.keys(item).forEach((key) => keySet.add(key))
  })

  const priorityColumns = ['productId', 'productName', 'quantity', 'price']
  const orderedKeys = []

  priorityColumns.forEach((key) => {
    if (keySet.has(key)) {
      orderedKeys.push(key)
      keySet.delete(key)
    }
  })

  return [...orderedKeys, ...Array.from(keySet)]
}

const formatColumnLabel = (column) => {
  return String(column)
    .replace(/_/g, ' ')
    .replace(/([a-z])([A-Z])/g, '$1 $2')
    .replace(/\b\w/g, (char) => char.toUpperCase())
}

const formatNumber = (value) => {
  const parsedValue = Number(value)

  if (Number.isNaN(parsedValue)) {
    return value
  }

  return new Intl.NumberFormat('en-US', {
    maximumFractionDigits: 2
  }).format(parsedValue)
}

const formatCellValue = (value, column) => {
  if (value === '' || value === null || value === undefined) {
    return '-'
  }

  if (Array.isArray(value)) {
    return value.map((item) => (typeof item === 'object' ? JSON.stringify(item) : String(item))).join(', ')
  }

  if (typeof value === 'object') {
    return JSON.stringify(value)
  }

  if (column.toLowerCase().includes('price') || column.toLowerCase().includes('cost')) {
    return formatNumber(value)
  }

  if (typeof value === 'number') {
    return formatNumber(value)
  }

  return value
}

const getCellClass = (column) => {
  if (column === 'productName' || column.toLowerCase().includes('name')) {
    return 'product-name'
  }

  if (column.toLowerCase().includes('price') || column.toLowerCase().includes('cost')) {
    return 'price'
  }

  return ''
}

const fetchProducts = async () => {
  loading.value = true
  errorMessage.value = ''

  try {
    const response = await fetch(WEBHOOK_URL)

    if (!response.ok) {
      throw new Error('Failed to load products')
    }

    const result = await response.json()
    const normalizedProducts = normalizeProducts(result)

    products.value = normalizedProducts
    columns.value = buildColumns(normalizedProducts)
  } catch (error) {
    console.error(error)
    errorMessage.value = 'Unable to load data from n8n webhook. Please check the workflow and webhook URL.'
    products.value = []
    columns.value = []
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  fetchProducts()
})
</script>

<style scoped>
.product-show-page {
  min-height: calc(100vh - 90px);
  background:
    radial-gradient(circle at 12% 10%, rgba(98, 54, 255, 0.15), transparent 44%),
    radial-gradient(circle at 85% 20%, rgba(0, 214, 143, 0.16), transparent 46%),
    linear-gradient(150deg, #f6f7ff 0%, #f3fffa 100%);
}

.headline {
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
  gap: 1rem;
  flex-wrap: wrap;
}

.eyebrow {
  text-transform: uppercase;
  letter-spacing: 0.1em;
  font-size: 0.78rem;
  color: #5e6b84;
  font-weight: 700;
}

.title {
  color: #18263f;
  font-weight: 800;
}

.subtitle {
  color: #5e6b84;
}

.reload-btn {
  border: 0;
  border-radius: 12px;
  padding: 0.62rem 1rem;
  font-weight: 700;
  color: #fff;
  background: linear-gradient(120deg, #4f46e5, #14b8a6);
  box-shadow: 0 10px 22px rgba(79, 70, 229, 0.24);
}

.reload-btn:hover:not(:disabled) {
  transform: translateY(-1px);
}

.table-shell {
  background: #fff;
  border-radius: 22px;
  padding: 1.1rem;
  box-shadow: 0 20px 48px rgba(24, 38, 63, 0.12);
}

.product-table thead th {
  background: linear-gradient(120deg, #4f46e5, #14b8a6);
  color: #fff;
  border: none;
  font-weight: 700;
  position: sticky;
  top: 0;
  z-index: 1;
}

.product-table tbody td {
  border-color: #e9edf4;
  color: #22314f;
}

.product-table tbody tr:nth-child(even) {
  background-color: #f8fbff;
}

.row-index {
  font-weight: 700;
  color: #4f46e5;
}

.product-name {
  font-weight: 700;
}

.price {
  font-weight: 700;
  color: #0f9d86;
}

.state-block {
  text-align: center;
  color: #5e6b84;
  padding: 2.2rem 1rem;
}

@media (max-width: 576px) {
  .table-shell {
    padding: 0.8rem;
  }

  .product-table {
    font-size: 0.9rem;
  }
}
</style>

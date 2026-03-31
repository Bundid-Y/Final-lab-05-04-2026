<template>
  <section class="product-show-page">
    <div class="container py-5">
      <div class="headline mb-4">
        <div>
          <p class="eyebrow mb-2">Inventory Portal</p>
          <h2 class="title mb-1">Product Catalog</h2>
          <p class="subtitle mb-0">Data loaded from n8n webhook</p>
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
                <th>Product ID</th>
                <th>Product Name</th>
                <th>Quantity</th>
                <th>Price</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(item, index) in products" :key="index">
                <td class="row-index">{{ index + 1 }}</td>
                <td>{{ item.productId }}</td>
                <td class="product-name">{{ item.productName }}</td>
                <td>{{ item.quantity }}</td>
                <td class="price">{{ formatPrice(item.price) }}</td>
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

const products = ref([])
const loading = ref(false)
const errorMessage = ref('')

const normalizeProducts = (payload) => {
  let list = []

  if (Array.isArray(payload)) {
    list = payload
  } else if (Array.isArray(payload?.data)) {
    list = payload.data
  } else if (Array.isArray(payload?.items)) {
    list = payload.items
  }

  return list
    .map((item) => (item?.json && typeof item.json === 'object' ? item.json : item))
    .filter((item) => item && typeof item === 'object')
}

const getField = (item, keys) => {
  for (const key of keys) {
    if (item[key] !== undefined && item[key] !== null && item[key] !== '') {
      return item[key]
    }
  }
  return '-'
}

const mapProduct = (item) => {
  return {
    productId: getField(item, ['productId', 'product_id', 'id', 'productID']),
    productName: getField(item, ['productName', 'product_name', 'name', 'title', 'product']),
    quantity: getField(item, ['quantity', 'qty', 'amount']),
    price: getField(item, ['price', 'unitPrice', 'cost'])
  }
}

const formatPrice = (price) => {
  if (price === '-' || price === '' || price === null || price === undefined) {
    return '-'
  }

  const value = Number(price)
  if (Number.isNaN(value)) return price

  return new Intl.NumberFormat('en-US', {
    maximumFractionDigits: 2
  }).format(value)
}

const fetchProducts = async () => {
  loading.value = true
  errorMessage.value = ''

  try {
    // ตอนทดสอบใน n8n ให้เปลี่ยนเป็น `/webhook-test/products` และกด "Listen for test event"
    // หลัง Publish workflow แล้วค่อยใช้ `/webhook/products`
    const response = await fetch('http://localhost:5678/webhook/products')

    if (!response.ok) {
      throw new Error('Failed to load products')
    }

    const result = await response.json()
    products.value = normalizeProducts(result).map(mapProduct)
  } catch (error) {
    console.error(error)
    errorMessage.value = 'Unable to load product data. Please check the n8n flow.'
    products.value = []
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

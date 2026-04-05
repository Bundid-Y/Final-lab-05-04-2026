<template>
  <section class="product-input-page">
    <div class="container py-5">
      <div class="row justify-content-center">
        <div class="col-12 col-md-9 col-lg-7">
          <div class="form-shell">
            <div class="card-head">
              <p class="eyebrow mb-2">Inventory Portal</p>
              <h2 class="title mb-1">Webhook Form Sender</h2>
              <small class="subtitle">Send data to n8n webhook: /webhook/formdata</small>
            </div>

            <div class="card-body p-4 p-md-5">
              <div
                v-if="status.message"
                :class="`alert alert-${status.type} alert-dismissible fade show`"
              >
                {{ status.message }}
                <button type="button" class="btn-close" @click="status.message = ''"></button>
              </div>

              <form @submit.prevent="submitForm">
                <div class="mb-3">
                  <label class="form-label">Product ID *</label>
                  <input
                    v-model.trim="form.productId"
                    name="productId"
                    type="text"
                    class="form-control custom-input"
                    required
                  />
                </div>

                <div class="mb-3">
                  <label class="form-label">Product Name *</label>
                  <input
                    v-model.trim="form.productName"
                    name="productName"
                    type="text"
                    class="form-control custom-input"
                    required
                  />
                </div>

                <div class="row g-3">
                  <div class="col-12 col-md-6">
                    <label class="form-label">Quantity *</label>
                    <input
                      v-model.number="form.quantity"
                      name="quantity"
                      type="number"
                      min="0"
                      class="form-control custom-input"
                      required
                    />
                  </div>

                  <div class="col-12 col-md-6">
                    <label class="form-label">Price *</label>
                    <input
                      v-model.number="form.price"
                      name="price"
                      type="number"
                      min="0"
                      step="0.01"
                      class="form-control custom-input"
                      required
                    />
                  </div>
                </div>

                <button class="btn submit-btn w-100 fw-bold mt-4" :disabled="loading">
                  <span v-if="loading" class="spinner-border spinner-border-sm me-2"></span>
                  {{ loading ? 'Submitting...' : 'Save Product' }}
                </button>
              </form>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup>
import { reactive, ref } from 'vue'

const WEBHOOK_URL = 'http://localhost:5678/webhook/formdata'
const FIELD_NAMES = ['productId', 'productName', 'quantity', 'price']

const form = reactive({
  productId: '',
  productName: '',
  quantity: null,
  price: null
})

const loading = ref(false)

const status = reactive({
  message: '',
  type: ''
})

const resetForm = () => {
  form.productId = ''
  form.productName = ''
  form.quantity = null
  form.price = null
}

const submitForm = async () => {
  loading.value = true
  status.message = ''

  try {
    const formData = new FormData()

    FIELD_NAMES.forEach((fieldName) => {
      const value = form[fieldName]
      formData.append(fieldName, value === null || value === undefined ? '' : String(value))
    })

    const response = await fetch(WEBHOOK_URL, {
      method: 'POST',
      body: formData
    })

    if (!response.ok) {
      throw new Error('Failed to save product')
    }

    status.message = 'Webhook sent successfully.'
    status.type = 'success'
    resetForm()
  } catch (error) {
    console.error(error)
    status.message = 'Unable to save product. Please check the n8n flow and try again.'
    status.type = 'danger'
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
:root {
  --ink: #13233a;
  --muted: #4f5f78;
  --card: #ffffff;
  --accent-start: #ff6b6b;
  --accent-mid: #ff9f43;
  --accent-end: #feca57;
}

.product-input-page {
  min-height: calc(100vh - 90px);
  background:
    radial-gradient(circle at 10% 20%, rgba(255, 107, 107, 0.22), transparent 45%),
    radial-gradient(circle at 90% 10%, rgba(72, 219, 251, 0.2), transparent 50%),
    linear-gradient(145deg, #f7faff 0%, #fef9f3 100%);
}

.form-shell {
  border-radius: 24px;
  overflow: hidden;
  background: var(--card);
  box-shadow: 0 20px 45px rgba(19, 35, 58, 0.12);
  border: 1px solid rgba(255, 255, 255, 0.7);
}

.card-head {
  padding: 2rem 2.25rem;
  color: #fff;
  background: linear-gradient(120deg, var(--accent-start), var(--accent-mid), var(--accent-end));
}

.eyebrow {
  text-transform: uppercase;
  letter-spacing: 0.1em;
  font-size: 0.78rem;
  opacity: 0.9;
}

.title {
  font-weight: 800;
  line-height: 1.15;
}

.subtitle {
  opacity: 0.95;
}

.form-label {
  color: var(--ink);
  font-weight: 700;
  margin-bottom: 0.4rem;
}

.custom-input {
  border-radius: 12px;
  border: 1px solid #d7deea;
  padding: 0.72rem 0.9rem;
  transition: all 0.2s ease;
}

.custom-input:focus {
  box-shadow: 0 0 0 0.25rem rgba(255, 107, 107, 0.18);
  border-color: #ff6b6b;
}

.submit-btn {
  border: none;
  border-radius: 14px;
  padding: 0.8rem 1rem;
  color: #fff;
  background: linear-gradient(120deg, #ff6b6b 0%, #ff8f5a 55%, #ffb14a 100%);
  box-shadow: 0 10px 22px rgba(255, 107, 107, 0.25);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.submit-btn:hover:not(:disabled) {
  transform: translateY(-1px);
  box-shadow: 0 14px 28px rgba(255, 107, 107, 0.32);
}

.submit-btn:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}
</style>

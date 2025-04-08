<template>
  <form @submit.prevent="handleSubmit" class="space-y-4">
    <input
      v-model="url"
      type="url"
      placeholder="Enter website URL"
      class="w-full p-2 border rounded"
      required
    />
    <button type="submit" class="bg-blue-600 text-white px-4 py-2 rounded">
      {{ loading ? 'Auditing...' : 'Run Audit' }}
    </button>
    <p v-if="error" class="text-red-600 mt-2">{{ error }}</p>
  </form>
</template>

<script setup>
import { ref } from 'vue'

const emit = defineEmits(['results'])
const url = ref('')
const loading = ref(false)
const error = ref('')

async function handleSubmit() {
  error.value = ''
  loading.value = true
  try {
    const res = await fetch('http://localhost:3000/api/audit', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ url: url.value }),
    })

    const data = await res.json()
    if (res.ok) {
      emit('results', data)
    } else {
      error.value = data.error || 'Unknown error'
    }
  } catch (e) {
    error.value = 'Failed to reach backend'
  } finally {
    loading.value = false
  }
}
</script>

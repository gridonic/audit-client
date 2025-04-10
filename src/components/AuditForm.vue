<template>
  <form @submit.prevent="handleSubmit" class="space-y-6">
    <div class="relative">
      <input
        v-model="url"
        type="url"
        placeholder="Enter website URL (e.g., https://example.com)"
        class="w-full p-4 border-2 border-gray-200 rounded-xl focus:border-blue-500 focus:ring-2 focus:ring-blue-200 transition-all duration-200 text-gray-700 placeholder-gray-400"
        required
      />
      <button 
        type="submit" 
        class="absolute right-2 top-1/2 -translate-y-1/2 bg-gradient-to-r from-blue-600 to-indigo-600 text-white px-6 py-2 rounded-lg font-medium hover:from-blue-700 hover:to-indigo-700 transition-all duration-200 shadow-md hover:shadow-lg disabled:opacity-50 disabled:cursor-not-allowed"
        :disabled="loading"
      >
        {{ loading ? 'Auditing...' : 'Run Audit' }}
      </button>
    </div>
    <p v-if="error" class="text-red-600 mt-2 text-sm bg-red-50 p-3 rounded-lg">{{ error }}</p>
    <p class="text-sm text-gray-500 mt-2 text-left">Gridonic and OpenAI Website Expertise</p>
  </form>
</template>

<script setup>
import { ref } from 'vue'

const emit = defineEmits(['results', 'loading', 'error', 'url'])
const url = ref('')
const loading = ref(false)
const error = ref('')

async function handleSubmit() {
  error.value = ''
  loading.value = true
  emit('loading', true)
  emit('error', '')
  emit('url', url.value)
  
  try {
    const apiUrl = import.meta.env.VITE_LOCAL_API_URL
    console.log('Sending request to:', `${apiUrl}/api/audit`)
    const res = await fetch(`${apiUrl}/api/audit`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ url: url.value }),
    })

    const data = await res.json()
    console.log('Response data:', data)
    
    if (res.ok) {
      emit('results', data)
    } else {
      if (data.fallback) {
        console.warn('Using fallback data due to server error:', data.error)
        emit('results', data.fallback)
      } else {
        error.value = data.error || 'Unknown error'
        emit('error', error.value)
      }
    }
  } catch (e) {
    console.error('Error:', e)
    error.value = 'Failed to reach backend'
    emit('error', error.value)
  } finally {
    loading.value = false
    emit('loading', false)
  }
}
</script>

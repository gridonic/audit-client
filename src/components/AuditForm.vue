<template>
  <form @submit.prevent="handleSubmit" class="space-y-4">
    <div class="flex items-center justify-between mb-4">
      <label class="text-sm font-medium text-gray-700">API Endpoint:</label>
      <div class="flex items-center space-x-2">
        <span class="text-sm text-gray-600">Local</span>
        <label class="relative inline-flex items-center cursor-pointer">
          <input type="checkbox" v-model="useNetlifyApi" class="sr-only peer">
          <div class="w-11 h-6 bg-gray-200 peer-focus:outline-none peer-focus:ring-4 peer-focus:ring-blue-300 rounded-full peer peer-checked:after:translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[2px] after:left-[2px] after:bg-white after:border-gray-300 after:border after:rounded-full after:h-5 after:w-5 after:transition-all peer-checked:bg-blue-600"></div>
        </label>
        <span class="text-sm text-gray-600">Netlify</span>
      </div>
    </div>
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
    <p class="text-xs text-gray-500 mt-1">Using API: {{ apiUrl }}</p>
  </form>
</template>

<script setup>
import { ref, computed } from 'vue'

const emit = defineEmits(['results', 'loading', 'error', 'url'])
const url = ref('')
const loading = ref(false)
const error = ref('')
const useNetlifyApi = ref(false)

const apiUrl = computed(() => {
  return useNetlifyApi.value 
    ? import.meta.env.VITE_NETLIFY_API_URL 
    : import.meta.env.VITE_LOCAL_API_URL
})

async function handleSubmit() {
  error.value = ''
  loading.value = true
  emit('loading', true)
  emit('error', '')
  emit('url', url.value)
  
  try {
    console.log('Sending request to:', `${apiUrl.value}/api/audit`)
    const res = await fetch(`${apiUrl.value}/api/audit`, {
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

<template>
  <div class="min-h-screen bg-gray-100 p-6">
    <div class="max-w-3xl mx-auto bg-white shadow-xl rounded-2xl p-6">
      <h1 class="text-4xl font-bold mb-6 text-gray-800 text-center">
        Gridonic Website Audit Tool
      </h1>
      <AuditForm 
        @results="handleResults" 
        @loading="handleLoading" 
        @error="handleError" 
        @url="handleUrl" 
      />
      <div v-if="results" class="mt-8 space-y-6">
        <AuditResults :data="results" :url="url" :loading="loading" :error="error" :results="results" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import AuditForm from './components/AuditForm.vue'
import AuditResults from './components/ChatResults.vue'

const results = ref(null)
const url = ref('')
const loading = ref(false)
const error = ref('')

function handleResults(data) {
  console.log('Received results in App.vue:', data)
  results.value = data
}

function handleLoading(isLoading) {
  loading.value = isLoading
}

function handleError(err) {
  error.value = err
}

function handleUrl(newUrl) {
  url.value = newUrl
}
</script>

<template>
  <div class="min-h-screen bg-gradient-to-br from-gray-50 to-gray-100 p-6">
    <div class="max-w-4xl mx-auto bg-white shadow-2xl rounded-3xl p-8">
      <h1 class="text-4xl font-bold mb-8 text-gray-800 text-center bg-clip-text text-transparent bg-gradient-to-r from-blue-600 to-indigo-600">
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

<style>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');

:root {
  font-family: 'Inter', sans-serif;
}

body {
  margin: 0;
  min-height: 100vh;
  background-color: #f9fafb;
}
</style>

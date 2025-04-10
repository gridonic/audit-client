<template>
  <div class="min-h-screen bg-gray-100 p-6">
    <div class="max-w-3xl mx-auto bg-white shadow-xl rounded-2xl p-6">
      <h1 class="text-4xl font-bold mb-6 text-gray-800 text-center">
        Gridonic Website Audit Tool
      </h1>
      <AuditForm @results="handleResults" />
      <div v-if="results" class="mt-8 space-y-6">
        <AuditResults :data="results" />

        <!-- Screenshot Section -->
        <div v-if="results?.screenshot" class="bg-gray-50 border rounded-lg p-4">
          <h2 class="text-lg font-semibold mb-2 text-gray-700">Screenshot Preview</h2>
          <img
            :src="`data:image/png;base64,${results.screenshot}`"
            alt="Audit Screenshot"
            class="rounded shadow-md border border-gray-200"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import AuditForm from './components/AuditForm.vue'
import AuditResults from './components/AuditResults.vue'

const results = ref(null)
function handleResults(data) {
  console.log('Received results in App.vue:', data)
  results.value = data
}
</script>

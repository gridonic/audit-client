<template>
  <div class="chat-container bg-gray-50 rounded-2xl p-6">
    <!-- Chat messages -->
    <div class="chat-messages space-y-6" ref="messagesContainer">
      <!-- System message -->
      <div class="message system-message bg-white p-4 rounded-xl shadow-sm">
        <div class="flex items-center space-x-2 mb-2">
          <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 text-blue-600" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
          </svg>
          <span class="font-medium text-gray-700">System</span>
        </div>
        <p class="text-gray-600">Analyzing {{ url }}...</p>
      </div>

      <!-- Loading state -->
      <div v-if="loading" class="message system-message bg-white p-4 rounded-xl shadow-sm">
        <div class="flex items-center space-x-2">
          <div class="animate-spin rounded-full h-5 w-5 border-b-2 border-blue-600"></div>
          <span class="text-gray-600">Running analysis...</span>
        </div>
      </div>

      <!-- Error state -->
      <div v-if="error" class="message error-message bg-red-50 p-4 rounded-xl">
        <div class="flex items-center space-x-2 text-red-600">
          <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
          </svg>
          <span class="font-medium">Error</span>
        </div>
        <p class="mt-2 text-red-600">{{ error }}</p>
      </div>

      <!-- Results -->
      <div v-if="results && !loading && !error" class="space-y-6">
        <!-- Screenshots -->
        <div v-if="screenshots.length > 0" class="message bg-white p-4 rounded-xl shadow-sm">
          <h3 class="text-lg font-semibold text-gray-800 mb-4">Screenshots</h3>
          <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
            <div v-for="(screenshot, index) in screenshots" :key="index" class="relative group">
              <img 
                :src="screenshot" 
                :alt="`Screenshot ${index + 1}`"
                class="w-full h-auto rounded-lg shadow-md transition-transform duration-200 group-hover:scale-[1.02]"
              />
              <div class="absolute inset-0 bg-black bg-opacity-0 group-hover:bg-opacity-10 transition-all duration-200 rounded-lg"></div>
            </div>
          </div>
        </div>

        <!-- Scores -->
        <div class="message bg-white p-4 rounded-xl shadow-sm">
          <h3 class="text-lg font-semibold text-gray-800 mb-4">Performance Scores</h3>
          <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
            <div v-for="(score, category) in scores" :key="category" 
              class="bg-gray-50 p-4 rounded-lg text-center">
              <div class="text-2xl font-bold" :class="getScoreColor(score)">
                {{ score }}
              </div>
              <div class="text-sm text-gray-600 mt-1 capitalize">
                {{ category }}
              </div>
            </div>
          </div>
        </div>

        <!-- Analysis -->
        <div class="message bg-white p-4 rounded-xl shadow-sm">
          <h3 class="text-lg font-semibold text-gray-800 mb-4">Visual Analysis</h3>
          <div class="prose prose-blue max-w-none">
            <div v-html="analysisContent"></div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

const props = defineProps({
  data: Object,
  url: String,
  loading: Boolean,
  error: String,
  results: Object
})

const messagesContainer = ref(null)

const screenshots = computed(() => {
  if (!props.results?.screenshots) return [];
  return Array.isArray(props.results.screenshots) ? props.results.screenshots : [props.results.screenshots];
})

const scores = computed(() => ({
  performance: props.results?.performance || 0,
  accessibility: props.results?.accessibility || 0,
  bestPractices: props.results?.bestPractices || 0,
  seo: props.results?.seo || 0
}))

const analysisContent = computed(() => {
  if (!props.results?.visualAnalysis) return ''
  return props.results.visualAnalysis.replace(/\n/g, '<br>')
})

function getScoreColor(score) {
  if (score >= 90) return 'text-green-600'
  if (score >= 50) return 'text-yellow-600'
  return 'text-red-600'
}

watch(() => props.results, () => {
  setTimeout(() => {
    if (messagesContainer.value) {
      messagesContainer.value.scrollTop = messagesContainer.value.scrollHeight
    }
  }, 100)
}, { deep: true })
</script>

<style scoped>
.chat-container {
  max-height: 80vh;
  overflow-y: auto;
}

.chat-messages {
  display: flex;
  flex-direction: column;
}

.message {
  max-width: 100%;
  margin-bottom: 1rem;
}

.system-message {
  background-color: #f8fafc;
}

.error-message {
  background-color: #fef2f2;
}

.prose {
  line-height: 1.6;
}

.prose p {
  margin-bottom: 1rem;
}

.prose ul {
  list-style-type: disc;
  padding-left: 1.5rem;
  margin-bottom: 1rem;
}

.prose li {
  margin-bottom: 0.5rem;
}
</style> 
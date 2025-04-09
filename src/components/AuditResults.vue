<template>
  <div class="bg-white border rounded-lg p-4 shadow">
    <!-- Screenshot Section -->
    <div v-if="data.screenshot" class="mb-6">
      <h2 class="text-2xl font-semibold mb-4 text-gray-800">Website Screenshot</h2>
      <div class="border rounded-lg overflow-hidden">
        <img 
          :src="'data:image/png;base64,' + data.screenshot" 
          alt="Website screenshot" 
          class="w-full h-auto"
        />
      </div>
    </div>

    <h2 class="text-2xl font-semibold mb-4 text-gray-800">Audit Scores</h2>
    <ul class="grid grid-cols-2 gap-4 text-lg text-gray-700">
      <li><strong>Performance:</strong> {{ data.performance }}</li>
      <li><strong>SEO:</strong> {{ data.seo }}</li>
      <li><strong>Accessibility:</strong> {{ data.accessibility }}</li>
      <li><strong>Best Practices:</strong> {{ data.bestPractices }}</li>
    </ul>

    <div class="mt-6">
  <h3 class="text-xl font-semibold mb-2 text-gray-800">Code Quality Issues</h3>

  <div v-if="data.codeIssues?.length">
    <ul class="space-y-3 text-sm text-gray-700">
      <li
        v-for="(issue, index) in data.codeIssues"
        :key="index"
        class="bg-gray-50 border rounded p-3"
      >
        <p><strong>🔍 {{ issue.message }}</strong></p>
        <p class="text-xs text-gray-500 mt-1">
          Rule: <code>{{ issue.ruleId }}</code> |
          Severity:
          <span :class="{
            'text-red-600 font-semibold': issue.severity === 2,
            'text-yellow-500': issue.severity === 1
          }">
            {{ issue.severity === 2 ? 'Error' : 'Warning' }}
          </span>
        </p>
      </li>
    </ul>
  </div>

  <div v-else class="text-sm text-gray-500 italic">No HTML code issues detected 🎉</div>
</div>
<div class="mt-6">
  <h3 class="text-xl font-semibold mb-2 text-gray-800">Console Warnings & Errors</h3>

  <div v-if="data.consoleLogs?.length">
    <ul class="space-y-2 text-sm text-gray-700">
      <li
        v-for="(log, i) in data.consoleLogs"
        :key="i"
        class="bg-gray-100 p-2 rounded border"
      >
        <strong :class="{
          'text-red-600': log.type === 'error',
          'text-yellow-600': log.type === 'warning'
        }">
          [{{ log.type.toUpperCase() }}]
        </strong>
        {{ log.text }}
      </li>
    </ul>
  </div>

  <div v-else class="text-sm text-gray-500 italic">No console errors or warnings 🎉</div>
</div>
  </div>
  <div class="mt-6">
  <h3 class="text-xl font-semibold text-gray-800 mb-2">AI-Powered Visual Design Feedback</h3>
  <p class="text-sm whitespace-pre-line text-gray-700 bg-gray-50 p-4 rounded border">
    {{ data.designAnalysis || 'No visual feedback available' }}
  </p>
</div>
</template>

<script setup>
const { data } = defineProps(['data']);
</script>

<template>
  <div class="chat-container">
    <!-- Loading State -->
    <div v-if="loading && !hasResults" class="loading-state">
      <div class="spinner"></div>
      <p>Analyzing your website...</p>
    </div>

    <!-- Error State -->
    <div v-else-if="error" class="error-state">
      <p>{{ error }}</p>
    </div>

    <!-- Results -->
    <div v-else-if="results" class="results">
      <!-- Scores -->
      <div v-if="scores" class="scores">
        <div v-for="(score, key) in scores" :key="key" class="score">
          <h3>{{ key }}</h3>
          <div class="score-value" :class="getScoreClass(score)">
            {{ score }}%
          </div>
        </div>
      </div>

      <!-- Analysis -->
      <div v-if="analysis" class="analysis">
        <h2>Visual Analysis</h2>
        <div class="analysis-content" v-html="analysis"></div>
      </div>

      <!-- Screenshots -->
      <div v-if="screenshots.length > 0" class="screenshots">
        <h2>Screenshots</h2>
        <div class="screenshots-grid">
          <div v-for="(screenshot, index) in screenshots" :key="index" class="screenshot">
            <img :src="screenshot" :alt="`Website screenshot ${index + 1}`" />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, watch } from 'vue';

const props = defineProps({
  loading: Boolean,
  error: String,
  results: Object
});

// Add a watcher to log when props change
watch(() => props.loading, (newValue) => {
  console.log('Loading state changed:', newValue);
});

watch(() => props.results, (newValue) => {
  console.log('Results changed:', newValue ? 'Has data' : 'No data');
});

// Computed property to check if we have valid results
const hasResults = computed(() => {
  return props.results && (
    props.results.screenshots?.length > 0 || 
    props.results.performance || 
    props.results.visualAnalysis
  );
});

const screenshots = computed(() => {
  if (!props.results?.screenshots) return [];
  const screenshotsArray = Array.isArray(props.results.screenshots) 
    ? props.results.screenshots 
    : [props.results.screenshots];
  
  return screenshotsArray.map(screenshot => {
    if (typeof screenshot === 'string') {
      return screenshot.startsWith('data:image') 
        ? screenshot 
        : `data:image/png;base64,${screenshot}`;
    }
    return '';
  }).filter(Boolean);
});

const scores = computed(() => {
  if (!props.results) return null;
  return {
    Performance: Math.round(props.results.performance) || 0,
    Accessibility: Math.round(props.results.accessibility) || 0,
    'Best Practices': Math.round(props.results.bestPractices) || 0,
    SEO: Math.round(props.results.seo) || 0
  };
});

const analysis = computed(() => {
  if (!props.results?.visualAnalysis) return '';
  // Convert markdown-style line breaks to HTML
  return props.results.visualAnalysis.replace(/\n/g, '<br>');
});

const getScoreClass = (score) => {
  if (score >= 90) return 'score-good';
  if (score >= 50) return 'score-warning';
  return 'score-bad';
};
</script>

<style scoped>
.chat-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 2rem;
}

.loading-state {
  text-align: center;
  padding: 2rem;
}

.spinner {
  border: 4px solid #f3f3f3;
  border-top: 4px solid #3498db;
  border-radius: 50%;
  width: 40px;
  height: 40px;
  animation: spin 1s linear infinite;
  margin: 0 auto 1rem;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.error-state {
  color: #e74c3c;
  text-align: center;
  padding: 2rem;
}

.scores {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1rem;
  margin-bottom: 2rem;
}

.score {
  text-align: center;
  padding: 1rem;
  background: #f8f9fa;
  border-radius: 0.5rem;
}

.score h3 {
  margin: 0 0 0.5rem;
  color: #2c3e50;
}

.score-value {
  font-size: 2rem;
  font-weight: bold;
  padding: 0.5rem;
  border-radius: 0.25rem;
}

.score-good {
  color: #27ae60;
  background: #e8f5e9;
}

.score-warning {
  color: #f39c12;
  background: #fff3e0;
}

.score-bad {
  color: #e74c3c;
  background: #fde8e8;
}

.analysis {
  background: white;
  padding: 2rem;
  border-radius: 0.5rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.analysis h2 {
  margin: 0 0 1rem;
  color: #2c3e50;
}

.analysis-content {
  line-height: 1.6;
  color: #34495e;
}

.analysis-content :deep(h3) {
  color: #2c3e50;
  margin: 1.5rem 0 1rem;
}

.analysis-content :deep(ul) {
  padding-left: 1.5rem;
  margin: 1rem 0;
}

.analysis-content :deep(li) {
  margin: 0.5rem 0;
}

.screenshots {
  margin-top: 2rem;
  background: white;
  padding: 2rem;
  border-radius: 0.5rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.screenshots h2 {
  margin: 0 0 1rem;
  color: #2c3e50;
}

.screenshots-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1rem;
}

.screenshot img {
  width: 100%;
  height: auto;
  border-radius: 0.5rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}
</style> 
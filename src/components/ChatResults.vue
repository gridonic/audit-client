<template>
  <div class="chat-container">
    <!-- Chat messages -->
    <div class="chat-messages" ref="messagesContainer">
      <!-- Assistant message with audit results -->
      <div class="message assistant-message">
        <div class="message-content">
          <!-- Loading State with Skeleton -->
          <div v-if="loading" class="loading-state">
            <div class="skeleton-container">
              <div class="skeleton-header"></div>
              <div class="skeleton-scores"></div>
              <div class="skeleton-screenshot"></div>
              <div class="skeleton-analysis"></div>
            </div>
          </div>
          
          <!-- Error State -->
          <div v-else-if="error" class="error-state">
            <div class="error-icon">⚠️</div>
            <div class="error-message">{{ error }}</div>
          </div>
          
          <!-- Results State -->
          <div v-else-if="results" class="results-state">
            <!-- Screenshots Section -->
            <div v-if="screenshots.length > 0" class="screenshots-section">
              <h3>Screenshots ({{ screenshots.length }})</h3>
              <div class="screenshots-grid">
                <div v-for="(screenshot, index) in screenshots" :key="index" class="screenshot-container">
                  <img :src="'data:image/png;base64,' + screenshot.data" :alt="screenshot.label" />
                  <div class="screenshot-label">{{ screenshot.label }}</div>
                </div>
              </div>
            </div>

            <!-- Scores Section -->
            <div class="scores-section">
              <div class="score-card" v-for="(score, category) in scores" :key="category">
                <div class="score-label">{{ formatCategory(category) }}</div>
                <div class="score-value" :style="{ color: getScoreColor(score) }">
                  {{ Math.round(score * 100) }}
                </div>
              </div>
            </div>
            
            <!-- Analysis Section -->
            <div class="analysis-section">
              <h3>Detailed Analysis</h3>
              <div v-if="analysisContent.length === 0" class="no-analysis">
                <p>No analysis available.</p>
                <button @click="debugData" class="debug-button">Debug Data</button>
              </div>
              <div v-else class="analysis-content">
                <div v-for="(section, index) in analysisContent" :key="index" class="analysis-section-item">
                  <h4>{{ section.title }}</h4>
                  <div class="section-content" v-html="section.content"></div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch, computed } from 'vue';

const props = defineProps({
  data: {
    type: Object,
    required: true
  },
  url: {
    type: String,
    required: true
  },
  loading: {
    type: Boolean,
    default: false
  },
  error: {
    type: String,
    default: ''
  },
  results: {
    type: Object,
    default: null
  }
});

const messagesContainer = ref(null);

// Function to convert markdown to HTML
function convertMarkdownToHtml(text) {
  if (!text) return '';
  
  // Convert bold/strong
  text = text.replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>');
  text = text.replace(/__(.*?)__/g, '<strong>$1</strong>');
  
  // Convert italic/emphasis
  text = text.replace(/\*(.*?)\*/g, '<em>$1</em>');
  text = text.replace(/_(.*?)_/g, '<em>$1</em>');
  
  // Convert code
  text = text.replace(/`(.*?)`/g, '<code>$1</code>');
  
  // Convert blockquotes
  text = text.replace(/^>\s(.*)$/gm, '<blockquote>$1</blockquote>');
  
  // Convert lists
  text = text.replace(/^\s*[-*+]\s(.*)$/gm, '<li>$1</li>');
  text = text.replace(/(<li>.*<\/li>)/s, '<ul>$1</ul>');
  
  // Convert paragraphs
  text = text.replace(/^(?!<[a-z])(.*)$/gm, '<p>$1</p>');
  
  return text;
}

// Handle multiple screenshots
const screenshots = computed(() => {
  if (!props.results) return [];
  
  console.log('Screenshots from server:', props.results.screenshots);
  
  // If results.screenshots is an array, return it with labels
  if (Array.isArray(props.results.screenshots)) {
    const processedScreenshots = props.results.screenshots.map((screenshot, index) => ({
      data: screenshot,
      label: ['Desktop', 'Tablet', 'Mobile'][index] || `Screenshot ${index + 1}`
    }));
    console.log('Processed screenshots:', processedScreenshots);
    return processedScreenshots;
  }
  
  // If it's a single screenshot, wrap it in an array
  if (props.results.screenshot) {
    console.log('Single screenshot found:', props.results.screenshot);
    return [{
      data: props.results.screenshot,
      label: 'Desktop'
    }];
  }
  
  return [];
});

const scores = computed(() => {
  if (!props.results) return {};
  
  return {
    performance: props.results.performance || 0,
    accessibility: props.results.accessibility || 0,
    bestPractices: props.results.bestPractices || 0,
    seo: props.results.seo || 0,
    pwa: props.results.pwa || 0
  };
});

// Process analysis content
const analysisContent = computed(() => {
  if (!props.results?.visualAnalysis) {
    console.log('No visual analysis available');
    return [];
  }
  
  console.log('Visual analysis content:', props.results.visualAnalysis);
  
  // Split the analysis into sections based on markdown headings (#)
  const sections = props.results.visualAnalysis.split(/(?=#)/).filter(Boolean);
  console.log('Split sections:', sections);
  
  const processedSections = sections.map(section => {
    // Extract title (remove the # and trim)
    const title = section.split('\n')[0].replace(/^#+\s*/, '').trim();
    // Get the content (everything after the first line)
    const content = section.split('\n').slice(1).join('\n').trim();
    
    console.log('Section title:', title);
    console.log('Section content before conversion:', content);
    
    const htmlContent = convertMarkdownToHtml(content);
    console.log('Section content after conversion:', htmlContent);
    
    return {
      title,
      content: htmlContent
    };
  });
  
  console.log('Processed sections:', processedSections);
  return processedSections;
});

// Scroll to bottom when messages change
watch(() => props.data, () => {
  setTimeout(() => {
    if (messagesContainer.value) {
      messagesContainer.value.scrollTop = messagesContainer.value.scrollHeight;
    }
  }, 100);
}, { deep: true });

onMounted(() => {
  setTimeout(() => {
    if (messagesContainer.value) {
      messagesContainer.value.scrollTop = messagesContainer.value.scrollHeight;
    }
  }, 100);
});

function getScoreColor(score) {
  if (score >= 0.9) return 'var(--success-color)';
  if (score >= 0.7) return 'var(--warning-color)';
  return 'var(--error-color)';
}

function formatCategory(category) {
  const categories = {
    performance: 'Performance',
    accessibility: 'Accessibility',
    bestPractices: 'Best Practices',
    seo: 'SEO',
    pwa: 'PWA'
  };
  return categories[category] || category;
}

// Debug function
function debugData() {
  console.log('Results:', props.results);
  console.log('Visual Analysis:', props.results?.visualAnalysis);
  console.log('Analysis Content:', analysisContent.value);
  
  // Show an alert with the raw visual analysis
  alert('Visual Analysis:\n\n' + props.results?.visualAnalysis);
}
</script>

<style scoped>
.chat-container {
  display: flex;
  flex-direction: column;
  height: 100%;
  max-width: 100%;
}

.chat-messages {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
  overflow-y: auto;
  max-height: calc(100vh - 180px);
  padding: 1rem;
  margin-bottom: 80px;
}

.message {
  display: flex;
  flex-direction: column;
  max-width: 100%;
}

.message-content {
  padding: 2rem;
  border-radius: 1rem;
  max-width: 100%;
  background-color: white;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
}

.assistant-message .message-content {
  background-color: #ffffff;
  color: #333;
  border: 1px solid #e5e7eb;
}

.screenshots-section {
  margin-bottom: 2rem;
}

.screenshots-section h3 {
  font-size: 1.75rem;
  font-weight: 700;
  color: var(--text-primary);
  margin-bottom: 1.5rem;
  text-align: center;
}

.screenshots-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 1.5rem;
  margin-bottom: 2rem;
}

.screenshot-container {
  width: 100%;
  border-radius: 0.75rem;
  overflow: hidden;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
  background-color: #f8fafc;
  position: relative;
  transition: transform 0.2s ease-in-out;
  display: flex;
  flex-direction: column;
}

.screenshot-container:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 12px -1px rgba(0, 0, 0, 0.15);
}

.screenshot-container img {
  width: 100%;
  height: auto;
  display: block;
  object-fit: contain;
  max-height: 500px;
}

.screenshot-label {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  background: rgba(0, 0, 0, 0.7);
  color: white;
  padding: 0.75rem;
  font-size: 0.875rem;
  text-align: center;
  font-weight: 500;
}

.scores-section {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
  gap: 1.5rem;
  margin-bottom: 2rem;
  padding: 1.5rem;
  background-color: #f8fafc;
  border-radius: 1rem;
}

.score-card {
  background-color: white;
  border-radius: 1rem;
  padding: 1.5rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.75rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
  transition: transform 0.2s, box-shadow 0.2s;
}

.score-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.score-label {
  font-weight: 600;
  color: var(--text-secondary);
  font-size: 0.9rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  text-align: center;
}

.score-value {
  font-size: 2.5rem;
  font-weight: 700;
  line-height: 1;
}

.analysis-section {
  background-color: white;
  border-radius: 1rem;
  padding: 2rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
}

.analysis-section h3 {
  font-size: 1.75rem;
  font-weight: 700;
  color: var(--text-primary);
  margin-bottom: 2rem;
  text-align: center;
}

.analysis-content {
  line-height: 1.8;
  color: var(--text-primary);
  font-size: 1.05rem;
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

.analysis-section-item {
  background-color: #f8fafc;
  border-radius: 1rem;
  padding: 2rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
  transition: transform 0.2s ease;
}

.analysis-section-item:hover {
  transform: translateY(-2px);
}

.analysis-section-item h4 {
  font-size: 1.5rem;
  font-weight: 700;
  color: var(--primary-color);
  margin-bottom: 1.5rem;
  padding-bottom: 0.75rem;
  border-bottom: 2px solid #e5e7eb;
}

.analysis-section-item .section-content {
  color: var(--text-secondary);
  font-size: 1.1rem;
  line-height: 1.8;
}

.analysis-section-item .section-content p {
  margin-bottom: 1.5rem;
}

.analysis-section-item .section-content ul,
.analysis-section-item .section-content ol {
  margin: 1.5rem 0;
  padding-left: 1.5rem;
}

.analysis-section-item .section-content li {
  margin-bottom: 1rem;
  position: relative;
  padding-left: 1.5rem;
}

.analysis-section-item .section-content li::before {
  content: "•";
  color: var(--primary-color);
  position: absolute;
  left: 0;
  font-weight: bold;
}

/* Markdown formatting */
.analysis-section-item .section-content strong,
.analysis-section-item .section-content b {
  font-weight: 700;
  color: var(--text-primary);
}

.analysis-section-item .section-content em,
.analysis-section-item .section-content i {
  font-style: italic;
}

.analysis-section-item .section-content code {
  background-color: #f1f5f9;
  padding: 0.2rem 0.4rem;
  border-radius: 0.25rem;
  font-family: monospace;
  font-size: 0.9em;
}

.analysis-section-item .section-content blockquote {
  border-left: 4px solid var(--primary-color);
  padding-left: 1rem;
  margin: 1.5rem 0;
  font-style: italic;
  color: var(--text-secondary);
}

.loading-state {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 200px;
}

.skeleton-container {
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

.skeleton-header,
.skeleton-scores,
.skeleton-screenshot,
.skeleton-analysis {
  background: linear-gradient(90deg, #f0f0f0 25%, #e0e0e0 50%, #f0f0f0 75%);
  background-size: 200% 100%;
  animation: loading 1.5s infinite;
  border-radius: 0.5rem;
}

.skeleton-screenshot {
  height: 300px;
}

.skeleton-scores {
  height: 100px;
}

.skeleton-analysis {
  height: 200px;
}

@keyframes loading {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}

.error-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.5rem;
  padding: 3rem;
  text-align: center;
  background-color: white;
  border-radius: 1rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
}

.error-icon {
  font-size: 2.5rem;
}

.error-message {
  color: var(--error-color);
  font-size: 1.1rem;
  font-weight: 500;
}

.no-analysis {
  text-align: center;
  padding: 2rem;
  background-color: #f8fafc;
  border-radius: 1rem;
}

.debug-button {
  background-color: var(--primary-color);
  color: white;
  border: none;
  padding: 0.75rem 1rem;
  border-radius: 0.5rem;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: background-color 0.2s;
}

.debug-button:hover {
  background-color: var(--primary-color-hover);
}
</style> 
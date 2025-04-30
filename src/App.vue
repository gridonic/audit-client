<template>
  <div class="container">
    <div class="form-box">
      <h1 class="heading">🔍 Website Audit Tool</h1>
      <div class="input-group">
        <input v-model="url" placeholder="https://example.com" />
        <button @click="handleSubmit">🚀 Run Audit</button>
      </div>
    </div>

    <div v-if="loading" class="loading">
      <div class="pulse"></div>
      <div class="pulse large"></div>
      <div class="pulse xlarge"></div>
    </div>

    <div v-if="result" class="result-section">
      <img :src="result.screenshotDesktop" alt="Desktop Screenshot" class="screenshot" />
      <img :src="result.screenshotMobile" alt="Mobile Screenshot" class="screenshot" />

      <div class="score-box">
        <h2>📊 Lighthouse Scores</h2>
        <div class="score-grid">
          <div v-for="(score, key) in result.lighthouse.desktop" :key="'desktop-' + key" class="score-item">
            <svg viewBox="0 0 36 36">
              <circle class="bg" cx="18" cy="18" r="16" />
              <circle
                class="fg desktop"
                cx="18" cy="18" r="16"
                :stroke-dasharray="(score.score * 100 * 1.005) + ', 1000'"
                transform="rotate(-90 18 18)"
              />
              <text x="18" y="21">{{ Math.round(score.score * 100) }}%</text>
            </svg>
            <p>🖥️ {{ key }}</p>
          </div>
          <div v-for="(score, key) in result.lighthouse.mobile" :key="'mobile-' + key" class="score-item">
            <svg viewBox="0 0 36 36">
              <circle class="bg" cx="18" cy="18" r="16" />
              <circle
                class="fg mobile"
                cx="18" cy="18" r="16"
                :stroke-dasharray="(score.score * 100 * 1.005) + ', 1000'"
                transform="rotate(-90 18 18)"
              />
              <text x="18" y="21">{{ Math.round(score.score * 100) }}%</text>
            </svg>
            <p>📱 {{ key }}</p>
          </div>
        </div>
      </div>

      <div class="audit-text">
        <p v-for="(line, index) in animatedLines" :key="index">{{ line }}</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      url: '',
      loading: false,
      result: null,
      animatedLines: []
    };
  },
  methods: {
    async handleSubmit() {
      this.loading = true;
      this.result = null;
      this.animatedLines = [];
      try {
        const serverUrl = process.env.NODE_ENV === 'development'
          ? "http://localhost:3000"
          : "https://audit-server-7nw6.onrender.com";
        const response = await fetch(`${serverUrl}/audit`, {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ url: this.url }),
        });
        const data = await response.json();
        this.result = data;
        this.revealAudit(data.audit);
      } catch (error) {
        console.error("Error:", error);
        alert("Failed to connect to the audit server.");
      } finally {
        this.loading = false;
      }
    },
    async revealAudit(text) {
      const lines = text.split("\n");
      for (let i = 0; i < lines.length; i++) {
        await new Promise(resolve => setTimeout(resolve, 50));
        this.animatedLines.push(lines[i]);
      }
    }
  }
};
</script>

<style scoped>
body {
  font-family: 'Inter', 'Segoe UI', sans-serif;
}

.container {
  min-height: 100vh;
  background: #f4f4f5;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 2rem;
}

.form-box {
  background: white;
  padding: 2rem;
  border-radius: 1rem;
  box-shadow: 0 8px 20px rgba(0,0,0,0.1);
  text-align: center;
  width: 100%;
  max-width: 600px;
}

.input-group {
  display: flex;
  flex-direction: row;
  gap: 1rem;
  justify-content: center;
  margin-top: 1rem;
}

.input-group input {
  flex: 1;
  padding: 0.75rem 1rem;
  border: 1px solid #ccc;
  border-radius: 0.75rem;
  font-size: 1rem;
}

.input-group button {
  background: black;
  color: white;
  border: none;
  padding: 0.75rem 1.25rem;
  border-radius: 0.75rem;
  cursor: pointer;
  font-size: 1rem;
  font-weight: bold;
}

.loading .pulse {
  height: 2rem;
  background: #ccc;
  margin-top: 1rem;
  border-radius: 1rem;
  animation: pulse 1.5s infinite ease-in-out;
}

.loading .large { height: 6rem; }
.loading .xlarge { height: 10rem; }

@keyframes pulse {
  0% { opacity: 0.6; }
  50% { opacity: 1; }
  100% { opacity: 0.6; }
}

.result-section {
  margin-top: 2rem;
  width: 100%;
  max-width: 900px;
}

.screenshot {
  width: 100%;
  border-radius: 1rem;
  margin-bottom: 1.5rem;
  box-shadow: 0 4px 12px rgba(0,0,0,0.05);
}

.score-box {
  background: white;
  border-radius: 1rem;
  padding: 2rem;
  box-shadow: 0 4px 12px rgba(0,0,0,0.05);
}

.score-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
  gap: 2rem;
}

.score-item {
  text-align: center;
}

.score-item svg {
  width: 80px;
  height: 80px;
}

.bg {
  stroke: #eee;
  stroke-width: 4;
  fill: none;
}

.fg {
  stroke-width: 4;
  fill: none;
  stroke-linecap: round;
}

.fg.desktop {
  stroke: #10b981;
}

.fg.mobile {
  stroke: #3b82f6;
}

.score-item text {
  fill: #333;
  font-size: 0.6rem;
  font-weight: bold;
  text-anchor: middle;
  dominant-baseline: middle;
}

.audit-text {
  background: white;
  padding: 1.5rem;
  margin-top: 2rem;
  border-radius: 1rem;
  white-space: pre-wrap;
  box-shadow: inset 0 2px 4px rgba(0,0,0,0.05);
  font-size: 0.95rem;
  color: #333;
  font-family: 'Inter', 'Segoe UI', sans-serif;
}

.heading {
  font-family: 'Inter', sans-serif;
  font-size: 2rem;
  font-weight: 700;
  color: #111827;
  margin-bottom: 1rem;
}

.subheading {
  font-family: 'Inter', sans-serif;
  font-size: 1.5rem;
  font-weight: 600;
  color: #1f2937;
  margin-bottom: 1rem;
}

.body-text {
  font-family: 'Inter', sans-serif;
  font-size: 1rem;
  color: #333;
}

</style>

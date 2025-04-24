<template>
  <div class="min-h-screen flex flex-col justify-center items-center bg-gradient-to-br from-gray-100 to-gray-200 py-20 px-4">
    <div class="w-full max-w-xl bg-white p-8 rounded-2xl shadow-xl">
      <h1 class="text-3xl font-bold text-center mb-6 text-gray-800">🔍 Website Audit Tool</h1>
      <input
        v-model="url" 
        placeholder="https://example.com"
        class="border border-gray-300 rounded-xl p-4 w-full text-base shadow-inner focus:outline-none focus:ring-2 focus:ring-blue-400 transition duration-200" 
      />
      <button 
        @click="handleSubmit" 
        class="mt-4 w-full py-3 bg-blue-600 hover:bg-blue-700 text-white text-lg font-semibold rounded-xl transition duration-300 ease-in-out">
        🚀 Run Audit
      </button>
    </div>

    <div v-if="loading" class="w-full max-w-xl mt-10 space-y-4">
      <div class="h-8 bg-gray-300 animate-pulse rounded-xl"></div>
      <div class="h-48 bg-gray-300 animate-pulse rounded-xl"></div>
      <div class="h-64 bg-gray-300 animate-pulse rounded-xl"></div>
    </div>

    <div v-if="result" class="w-full max-w-4xl mt-12 space-y-6">
      <img :src="result.screenshotDesktop" alt="Desktop Screenshot" class="w-full border rounded-xl shadow" />
      <img :src="result.screenshotMobile" alt="Mobile Screenshot" class="w-full border rounded-xl shadow" />

      <div class="bg-white p-6 rounded-xl shadow-md">
        <h3 class="font-bold mb-4 text-lg text-gray-700">📊 Lighthouse Scores (Desktop)</h3>
        <div v-for="(score, key) in result.lighthouse.desktop" :key="'desktop-' + key" class="mb-4">
          <label class="block font-medium mb-1 text-sm text-gray-700">{{ key }}: {{ Math.round(score.score * 100) }}%</label>
          <div class="w-full bg-gray-200 rounded-full h-4 overflow-hidden">
            <div class="bg-green-500 h-4" :style="{ width: Math.round(score.score * 100) + '%' }"></div>
          </div>
        </div>

        <h3 class="font-bold mt-6 mb-4 text-lg text-gray-700">📱 Lighthouse Scores (Mobile)</h3>
        <div v-for="(score, key) in result.lighthouse.mobile" :key="'mobile-' + key" class="mb-4">
          <label class="block font-medium mb-1 text-sm text-gray-700">{{ key }}: {{ Math.round(score.score * 100) }}%</label>
          <div class="w-full bg-gray-200 rounded-full h-4 overflow-hidden">
            <div class="bg-blue-500 h-4" :style="{ width: Math.round(score.score * 100) + '%' }"></div>
          </div>
        </div>
      </div>

      <pre class="bg-white p-6 rounded-xl shadow-inner whitespace-pre-wrap text-sm text-gray-800">{{ result.audit }}</pre>
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
    };
  },
  methods: {
    async handleSubmit() {
      this.loading = true;
      this.result = null;
      try {
        const serverUrl = process.env.NODE_ENV === 'development' 
          ? "http://localhost:3000" 
          : "https://audit-server-7nw6.onrender.com";
        const response = await fetch(`${serverUrl}/audit`, {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({ url: this.url }),
        });
        const data = await response.json();
        this.result = data;
      } catch (error) {
        console.error("Error:", error);
        alert("Failed to connect to the audit server.");
      } finally {
        this.loading = false;
      }
    }
  }
};
</script>

<style scoped>
/* Add any global styles if needed */
</style>

// ==== Server Side (Node.js/Express) ====
// File: /server/index.js

const express = require("express");
const cors = require("cors");
const { auditWebsite } = require("./lib/audit");
const app = express();

const corsOptions = {
  origin: ["http://localhost:5173", "https://your-frontend-name.netlify.app"],
  methods: ["GET", "POST", "OPTIONS"],
  allowedHeaders: ["Content-Type"],
};

app.use(cors(corsOptions));
app.use(express.json());

// Optional: explicitly respond to OPTIONS preflight
app.options("/audit", (req, res) => {
  res.set("Access-Control-Allow-Origin", req.headers.origin || "*");
  res.set("Access-Control-Allow-Methods", "GET,POST,OPTIONS");
  res.set("Access-Control-Allow-Headers", "Content-Type");
  res.sendStatus(200);
});

app.post("/audit", async (req, res) => {
  const { url } = req.body;
  try {
    const result = await auditWebsite(url);
    res.json(result);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.get("/audit", (req, res) => {
  res.send("GET requests are not supported for this endpoint.");
});

app.listen(3000, () => console.log("Server running on port 3000"));

// ==== Audit Logic (/server/lib/audit.js) ====

const { chromium, devices } = require("playwright");
const lighthouse = require("lighthouse");
const { launch } = require("chrome-launcher");
const { Configuration, OpenAIApi } = require("openai");
const fs = require("fs/promises");
const path = require("path");
const sharp = require("sharp");

const configuration = new Configuration({ apiKey: process.env.OPENAI_API_KEY });
const openai = new OpenAIApi(configuration);

async function auditWebsite(url) {
  const chrome = await launch({ chromeFlags: ["--headless"] });
  const options = { logLevel: "info", output: "json", onlyCategories: ["performance", "accessibility", "seo", "best-practices"], port: chrome.port };

  const desktopResult = await lighthouse(url, options);

  options.emulatedFormFactor = "mobile";
  const mobileResult = await lighthouse(url, options);

  const browser = await chromium.launch();
  const contextDesktop = await browser.newContext();
  const pageDesktop = await contextDesktop.newPage();
  await pageDesktop.goto(url);
  await pageDesktop.setViewportSize({ width: 1280, height: 720 });
  const screenshotDesktop = await pageDesktop.screenshot();

  const contextMobile = await browser.newContext({ ...devices["iPhone 13"] });
  const pageMobile = await contextMobile.newPage();
  await pageMobile.goto(url);
  const screenshotMobile = await pageMobile.screenshot();

  await browser.close();
  await chrome.kill();

  const analysisPrompt = `Act as a senior UX/UI designer and strategist. Analyze the design of the website and give a brutally honest, detailed critique...`; // (Insert full prompt here)

  const completion = await openai.createChatCompletion({
    model: "gpt-4-vision-preview",
    messages: [
      { role: "system", content: "You are a helpful assistant." },
      { role: "user", content: analysisPrompt },
      { role: "user", name: "image", content: screenshotDesktop.toString("base64") }
    ],
    max_tokens: 2000
  });

  return {
    screenshotDesktop: `data:image/png;base64,${screenshotDesktop.toString("base64")}`,
    screenshotMobile: `data:image/png;base64,${screenshotMobile.toString("base64")}`,
    audit: completion.data.choices[0].message.content,
    lighthouse: {
      desktop: desktopResult.lhr.categories,
      mobile: mobileResult.lhr.categories
    }
  };
}

module.exports = { auditWebsite };
<template>
  <div class="p-6 max-w-xl mx-auto">
    <input v-model="url" placeholder="Enter website URL" class="border rounded p-2 w-full" />
    <button @click="handleSubmit" class="mt-4 px-4 py-2 bg-blue-500 text-white rounded">Run Audit</button>

    <div v-if="loading" class="space-y-4 mt-8">
      <div class="h-8 bg-gray-200 animate-pulse"></div>
      <div class="h-40 bg-gray-200 animate-pulse"></div>
      <div class="h-60 bg-gray-200 animate-pulse"></div>
    </div>

    <div v-if="result" class="mt-8 space-y-6">
      <img :src="result.screenshotDesktop" alt="Desktop Screenshot" class="w-full border rounded" />
      <img :src="result.screenshotMobile" alt="Mobile Screenshot" class="w-full border rounded" />
      <pre class="bg-gray-100 p-4 rounded whitespace-pre-wrap text-sm">{{ result.audit }}</pre>
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
      const res = await fetch('https://your-backend-url.onrender.com/audit', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ url: this.url })
      });
      const data = await res.json();
      this.result = data;
      this.loading = false;
    }
  }
};
</script>

<style scoped>
/* Add any global styles if needed */
</style>

<template>
  <div class="flex flex-col min-h-screen bg-gray-50">
    <header class="bg-white shadow-sm py-4 px-6">
      <div class="max-w-5xl mx-auto">
        <h1 class="text-2xl font-bold text-gray-800">Website Audit Tool</h1>
      </div>
    </header>
    
    <main class="flex-grow py-8 px-6">
      <div class="max-w-5xl mx-auto space-y-10">
        <div class="bg-white rounded-2xl shadow-lg p-8">
          <AuditForm 
            @submit="handleSubmit" 
            :loading="loading" 
          />
        </div>
        
        <ChatResults 
          :loading="loading"
          :error="error"
          :results="results"
        />
      </div>
    </main>
    
    <footer class="bg-gray-800 text-white py-4 px-6">
      <div class="max-w-5xl mx-auto text-center text-sm">
        &copy; {{ new Date().getFullYear() }} Website Audit Tool. All rights reserved.
      </div>
    </footer>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import AuditForm from './components/AuditForm.vue';
import ChatResults from './components/ChatResults.vue';

const url = ref('');
const loading = ref(false);
const error = ref('');
const results = ref(null);

// Use the API URL from environment variables
const apiUrl = import.meta.env.VITE_API_URL + '/api/audit';

onMounted(() => {
  console.log('App mounted, API URL:', apiUrl);
});

const handleSubmit = async (submittedUrl) => {
  url.value = submittedUrl;
  loading.value = true;
  error.value = '';
  results.value = null;
  
  console.log('Submitting URL:', url.value);
  console.log('Using API URL:', apiUrl);
  
  try {
    const response = await fetch(apiUrl, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ url: url.value }),
    });
    
    if (!response.ok) {
      const errorData = await response.text();
      throw new Error(`Server responded with status ${response.status}: ${errorData}`);
    }
    
    const data = await response.json();
    console.log('Response data:', data);
    
    if (data.error) {
      error.value = data.error;
    } else {
      results.value = data;
    }
  } catch (err) {
    console.error('Error during fetch:', err);
    error.value = `Failed to connect to the audit server: ${err.message}`;
  } finally {
    loading.value = false;
  }
};
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');

body {
  font-family: 'Inter', sans-serif;
}
</style>

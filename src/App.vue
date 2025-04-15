<template>
  <div class="min-h-screen bg-gradient-to-br from-gray-50 to-gray-100 p-6">
    <div class="max-w-4xl mx-auto bg-white shadow-2xl rounded-3xl p-8">
      <h1 class="text-4xl font-bold mb-8 text-gray-800 text-center bg-clip-text text-transparent bg-gradient-to-r from-blue-600 to-indigo-600">
        Gridonic Website Audit Tool
      </h1>
      <AuditForm 
        @submit="handleSubmit" 
      />
      <div class="mt-8 space-y-6">
        <ChatResults 
          :url="url" 
          :loading="loading" 
          :error="error" 
          :results="results" 
        />
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import AuditForm from './components/AuditForm.vue';
import ChatResults from './components/ChatResults.vue';

const url = ref('');
const loading = ref(false);
const error = ref('');
const results = ref(null);

const handleSubmit = async (formUrl) => {
  url.value = formUrl;
  loading.value = true;
  error.value = '';
  results.value = null;

  try {
    console.log('Sending request to server...');
    const apiUrl = import.meta.env.VITE_API_URL;
    console.log('Using API URL:', apiUrl);
    const response = await fetch(`${apiUrl}/api/audit`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ url: formUrl }),
    });

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    console.log('Received response from server');
    const data = await response.json();
    console.log('Received data:', data);
    
    if (data) {
      if (data.screenshots || data.performance || data.visualAnalysis) {
        results.value = data;
        console.log('Results set successfully');
      } else {
        console.warn('Unexpected response structure:', data);
        throw new Error('Invalid response data received from server');
      }
    } else {
      throw new Error('No data received from server');
    }
  } catch (e) {
    console.error('Error during audit:', e);
    error.value = e.message;
  } finally {
    console.log('Setting loading to false');
    loading.value = false;
  }
};
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

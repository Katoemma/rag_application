<template>
    <div class="min-h-screen bg-gradient-to-br from-gray-50 to-gray-200 flex flex-col items-center py-12 px-4">
      <!-- Header -->
      <h1 class="text-4xl md:text-5xl font-extrabold text-gray-900 mb-8 tracking-tight">
        RAG SQL Query App
      </h1>
  
      <!-- Card Container -->
      <div class="w-full max-w-4xl bg-white rounded-xl shadow-2xl p-8 transform transition-all duration-300 hover:shadow-3xl">
        <!-- Query Input -->
        <div class="mb-6">
          <label class="block text-gray-700 font-medium mb-2">Enter Your Query</label>
          <div class="relative">
            <input
              v-model="query"
              @keyup.enter="submitQuery"
              type="text"
              placeholder="e.g., List products under $50"
              class="w-full px-5 py-3 border border-gray-300 rounded-lg bg-gray-50 text-gray-800 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-transparent transition duration-200 placeholder-gray-400"
              @input="console.log('Query input changed:', query)"
            />
            <span class="absolute inset-y-0 right-3 flex items-center text-gray-400">
              <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
              </svg>
            </span>
          </div>
        </div>
  
        <!-- Submit Button -->
        <button
          @click="submitQuery"
          :disabled="loading"
          class="w-full bg-indigo-600 text-white py-3 rounded-lg font-semibold hover:bg-indigo-700 disabled:bg-gray-400 disabled:cursor-not-allowed transition duration-200 flex items-center justify-center space-x-2"
        >
          <span>{{ loading ? 'Processing...' : 'Submit Query' }}</span>
          <svg v-if="loading" class="animate-spin w-5 h-5" fill="none" viewBox="0 0 24 24">
            <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" />
            <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z" />
          </svg>
        </button>
  
        <!-- Error Message -->
        <div v-if="error" class="mt-6 p-4 bg-red-100 text-red-700 rounded-lg flex items-center space-x-2 animate-fade-in">
          <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
          </svg>
          <span>{{ error }}</span>
        </div>
  
        <!-- Generated SQL (Commented Out) -->
        <!-- <div v-if="sqlQuery" class="mt-6">
          <h3 class="text-xl font-semibold text-gray-800 mb-3">Generated SQL</h3>
          <pre class="bg-gray-900 text-gray-100 p-4 rounded-lg font-mono text-sm overflow-x-auto">{{ sqlQuery }}</pre>
        </div> -->
  
        <!-- Results Table -->
        <div v-if="results.length" class="mt-6 animate-fade-in">
          <h3 class="text-xl font-semibold text-gray-800 mb-4">Query Results</h3>
          <div class="overflow-x-auto">
            <table class="w-full border-collapse bg-white rounded-lg shadow-sm">
              <thead>
                <tr class="bg-indigo-50 text-indigo-900">
                  <th
                    v-for="col in Object.keys(results[0] || {})"
                    :key="col"
                    class="px-6 py-3 text-left font-medium uppercase tracking-wider border-b border-gray-200"
                  >
                    {{ col }}
                  </th>
                </tr>
              </thead>
              <tbody>
                <tr
                  v-for="row in results"
                  :key="row.id"
                  class="hover:bg-gray-50 transition duration-150"
                >
                  <td
                    v-for="col in Object.keys(row)"
                    :key="col"
                    class="px-6 py-4 border-b border-gray-200 text-gray-700"
                  >
                    {{ row[col] }}
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref } from 'vue';
  
  const query = ref('');
  const sqlQuery = ref('');
  const results = ref([]);
  const loading = ref(false);
  const error = ref('');
  
  async function submitQuery() {
    console.log('submitQuery called, query:', query.value);
    if (!query.value.trim()) {
      console.log('Empty query, aborting');
      return;
    }
    loading.value = true;
    console.log('Loading state set to true');
    error.value = '';
    try {
      console.log('Sending POST to /api/rag with body:', { query: query.value });
      const response = await $fetch('/api/rag', {
        method: 'POST',
        body: { query: query.value },
      });
      console.log('API response:', response);
      sqlQuery.value = response.sqlQuery || '';
      console.log('Set sqlQuery:', sqlQuery.value);
      results.value = response.results || [];
      console.log('Set results:', results.value);
    } catch (err) {
      console.error('API error:', err);
      error.value = 'Error processing query: ' + (err.message || err);
      console.log('Set error:', error.value);
    } finally {
      loading.value = false;
      console.log('Loading state reset to false');
    }
  }
  </script>
  
  <style scoped>
  /* Fade-in animation for dynamic content */
  .animate-fade-in {
    animation: fadeIn 0.5s ease-in;
  }
  
  @keyframes fadeIn {
    from {
      opacity: 0;
      transform: translateY(10px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }
  
  /* Custom shadow for hover effect */
  .shadow-3xl {
    box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.25);
  }
  </style>
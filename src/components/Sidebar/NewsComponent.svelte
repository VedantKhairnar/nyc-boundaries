<script lang="ts">
  import { onMount } from 'svelte';
  import axios from 'axios';

  export let districtName: string;
  
  let articles = [];
  let isLoading = true;
  let error = null;
  
  // Function to format date
  function formatDate(dateString: string): string {
    const date = new Date(dateString);
    return date.toLocaleDateString('en-US', {
      year: 'numeric',
      month: 'short',
      day: 'numeric'
    });
  }
  
  // Function to fetch news articles
  async function fetchNews() {
    isLoading = true;
    error = null;
    
    try {
      // Using NewsAPI.org - you'll need to replace 'YOUR_API_KEY' with an actual API key
      const API_KEY = import.meta.env.VITE_NEWS_API_KEY;
      const searchQuery = encodeURIComponent(`New York ${districtName}`);
      const response = await axios.get(
        `https://newsapi.org/v2/everything?q=${searchQuery}&sortBy=publishedAt&language=en&pageSize=5&apiKey=${API_KEY}`
      );
      
      if (response.data.status === 'ok') {
        articles = response.data.articles;
      } else {
        error = 'Failed to fetch news articles';
      }
    } catch (err) {
      error = err.message || 'An error occurred while fetching news';
    } finally {
      isLoading = false;
    }
  }
  
  $: if (districtName) {
    fetchNews();
  }
  
  onMount(() => {
    if (districtName) {
      fetchNews();
    }
  });
</script>

<div class="px-4 py-4 border-t border-gray-200">
  <h3 class="text-lg font-medium text-gray-900 mb-3">Related News</h3>
  
  {#if isLoading}
    <div class="flex justify-center items-center py-4">
      <div class="animate-spin rounded-full h-6 w-6 border-b-2 border-blue-500"></div>
    </div>
  {:else if error}
    <div class="bg-red-50 p-3 rounded-md">
      <p class="text-red-600 text-sm">{error}</p>
    </div>
  {:else if articles.length === 0}
    <p class="text-gray-500 text-sm">No news articles found for this district.</p>
  {:else}
    <ul class="space-y-4">
      {#each articles as article}
        <li class="border-b border-gray-100 pb-3 last:border-0">
          <a href={article.url} target="_blank" rel="noreferrer" class="block hover:bg-gray-50 rounded-md transition-colors p-2 -mx-2">
            <h4 class="text-sm font-medium text-gray-900 hover:text-blue-600">{article.title}</h4>
            <div class="flex items-center mt-1 text-xs text-gray-500">
              <span class="font-medium">{article.source.name}</span>
              <span class="mx-1">•</span>
              <span>{formatDate(article.publishedAt)}</span>
            </div>
          </a>
        </li>
      {/each}
    </ul>
  {/if}
</div>
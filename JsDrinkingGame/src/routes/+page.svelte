<script lang="ts">
    import confetti from 'canvas-confetti';
    import { Beer, Package, Shuffle, Search, ExternalLink, Italic } from 'lucide-svelte';
    import { writable } from 'svelte/store';
  
    let word = '';
    let gameStarted = false;
    const searchResult = writable<{ found: boolean, name?: string, description?: string, version?: string, keywords?: string[] } | null>(null);
    let isLoading = false;
    let randomWords = [];

    async function loadWords() {
        try {
            const response = await fetch('/words.txt');
            if (!response.ok) {
                throw new Error('Failed to fetch the words');
            }
            const text = await response.text();
            randomWords = text.split('\n').map(word => word.trim()).filter(word => word);
        } catch (error) {
            console.error(error);
        } finally {
            isLoading = false;
        }
    }

    loadWords();
  
    function handleRandomWord() {
      const randomIndex = Math.floor(Math.random() * randomWords.length);
      word = randomWords[randomIndex];
    }
  
    async function searchNpm() {
      if (!word) return;
  
      gameStarted = true;
      isLoading = true;
      try {
        const response = await fetch(`https://registry.npmjs.org/${encodeURIComponent(word.toLowerCase())}`);
        if (response.ok) {
          const data = await response.json();
          searchResult.set({
            found: true,
            name: data.name,
            description: data.description,
            version: data['dist-tags'].latest,
            keywords: data.keywords || []
          });
          confetti({
            particleCount: 100,
            spread: 70,
            origin: { y: 0.6 }
          });
        } else {
          searchResult.set({ found: false });
        }
      } catch (error) {
        console.error('Error searching npm:', error);
        searchResult.set({ found: false });
      } finally {
        isLoading = false;
      }
    }
  
    function openNpmPage(packageName: string) {
      window.open(`https://www.npmjs.com/package/${packageName}`, '_blank');
    }
  </script>
  
  <div class="min-h-screen bg-gradient-to-br from-purple-700 to-indigo-900 p-6 flex items-center justify-center">
    <div class="max-w-md w-full bg-white bg-opacity-10 backdrop-blur-lg rounded-xl shadow-xl overflow-hidden">
      <div class="p-6">
        <h1 class="text-3xl font-bold mb-4 text-center text-white flex items-center justify-center">
            JS Drinking Game
        </h1>
        <p class="text-white text-opacity-80 text-center mb-4">
          Enter a JS package name and see if it exists on npm. If it does, take a drink! 🍻
        </p>

        {#if !gameStarted}
        <div class="bg-white bg-opacity-20 p-3 text-center rounded mb-4 mt-4">
            <h2 class="text-xl text-white mb-2">🚨 Warning: Drink Responsibly 🍻</h2>
            <p class="text-left text-white">You may think that after 8 beers you can just write another SQL script on Prod, but you can't. You may forget what happened, but the DROP TABLE won't.</p>
            <p class="text-left text-white">If you have any doubts about what you are doing, stop coding. (Stopping drinking is not an option.)</p>
        </div>
        {/if}
        
        <div class="flex space-x-2 mb-4">
          <input
            type="text"
            bind:value={word}
            placeholder="Enter a JS package name"
            class="flex-grow bg-white bg-opacity-20 text-white placeholder-gray-300 p-2 rounded"
          />
          <button on:click={handleRandomWord} class="bg-yellow-500 hover:bg-yellow-600 p-2 rounded">
            <Shuffle class="h-4 w-4" />
          </button>
        </div>
        <button on:click={searchNpm} disabled={!word || isLoading} class="w-full mb-4 bg-blue-500 hover:bg-blue-600 p-2 rounded">
          {#if isLoading}
            Searching...
          {:else}
            Search for package
          {/if}
        </button>
        {#if $searchResult !== null}
        <div class="{($searchResult.found ? 'bg-gradient-to-r from-green-400 to-blue-500' : 'bg-gradient-to-r from-red-400 to-yellow-500')} text-white p-4 rounded">
            <div class="flex items-center justify-between">
              <div class="flex items-center">
                <Package class="mr-2" />
                {#if $searchResult.found}
                  {$searchResult.name}
                {:else}
                  Not Found
                {/if}
              </div>
              {#if $searchResult.found}
                <button on:click={() => openNpmPage($searchResult.name)} class="text-white hover:text-blue-200">
                  <ExternalLink class="h-5 w-5" />
                </button>
              {/if}
            </div>
            {#if $searchResult.found}
              <p class="text-white text-opacity-80">{ $searchResult.version }</p>
            {/if}
            <div>
              {#if $searchResult.found}
                <p class="mb-2">{ $searchResult.description }</p>
                <div class="flex flex-wrap gap-2">
                  {#each $searchResult.keywords as keyword}
                    <span class="px-2 py-1 bg-white bg-opacity-30 rounded-full text-xs">{keyword}</span>
                  {/each}
                </div>
              {:else}
                <p>No npm package found. How about you create a new package with this name?</p>
              {/if}
            </div>
          </div>
        {/if}
      </div>
    </div>
  </div>
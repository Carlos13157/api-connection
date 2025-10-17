<script setup>
import { ref } from 'vue';

const searchTerm = ref('');
const games = ref([]);
const loading = ref(false);
const error = ref(null);

async function searchGame() {
  if (!searchTerm.value.trim()) return;

  loading.value = true;
  error.value = null;
  games.value = [];

  try {
    const query = encodeURIComponent(searchTerm.value.trim());
    const apiKey = '1e0ef13f35044a7faa1938824b267a7f'; 

    const response = await fetch(`https://api.gamebrain.co/v1/games?query=${query}`, {
      method: 'GET',
      headers: {
        'x-api-key': apiKey,
        'Accept': 'application/json'
      }
    });
    
    if (!response.ok) throw new Error(`Error ${response.status}`);
    
    const data = await response.json();
    games.value = data.results; // ¡Aquí está el cambio!
    console.log('Respuesta de GameBrain:', data.results); // También podrías loguear directamente results
  } catch (err) {
    error.value = 'Error: ' + err.message;
  } finally {
    loading.value = false;
  }
}
</script>

<template>
  <div class="bg-blue-400 p-5 rounded-lg shadow-md max-w-3xl mx-auto my-8">
    <h1>Buscador de Juegos (GameBrain API)</h1>

    <div class="mb-5">
      <input class="p-2.5 text-base border-2 border-black rounded-md mr-2.5"
        v-model="searchTerm"
        @keyup.enter="searchGame"
        placeholder="Escribe el nombre del juego..."
        :disabled="loading"
      />
      <button class="px-5 py-2.5 text-base bg-green-500 text-gray-100 border-none rounded-md cursor-pointer hover:bg-green-600 disabled:bg-gray-300 disabled:cursor-not-allowed" @click="searchGame" :disabled="loading || !searchTerm.trim()">
        {{ loading ? 'Buscando...' : 'Buscar' }}
      </button>
    </div>

    <div v-if="loading" class="text-center text-lg text-gray-100">Cargando resultados...</div>
    <div v-if="error" class="text-red-600 bg-red-100 p-2.5 rounded-md mt-2.5">{{ error }}</div>

    <div v-if="games.length > 0">
      <div v-for="game in games" :key="game.id" class="border-2 border-black rounded-lg p-5 text-center mt-5">
        <h2>{{ game.name }}</h2>
        <p><strong>Género:</strong> {{ game.genre || 'N/A' }}</p>
        <p><strong>Año:</strong> {{ game.year || 'Desconocida' }}</p>
        <img
          v-if="game.image"
          :src="game.image"
          :alt="game.name"
          class="max-w-[200px] rounded-lg mt-2.5"
        />
      </div>
    </div>
  </div>
</template>
<template>
  <div class="grid mt-10">
    <!-- Campo de búsqueda -->
    <div class="flex justify-center mb-6">
      <div class="w-1/2">
        <label for="searchSong"
          class="relative block rounded-md border border-gray-200 shadow-sm focus-within:border-blue-600 focus-within:ring-1 focus-within:ring-blue-600">
          <input type="text" id="searchSong"
            class="peer border-none bg-transparent placeholder-transparent focus:border-transparent focus:outline-none focus:ring-0 w-full p-2"
            placeholder="What music are you thinking about?" v-model="inputValue" @keydown="onKeyDown" />
          <span
            class="pointer-events-none absolute start-2.5 top-0 -translate-y-1/2 bg-white p-0.5 text-xs text-gray-700 transition-all peer-placeholder-shown:top-1/2 peer-placeholder-shown:text-sm peer-focus:top-0 peer-focus:text-xs">
            Song
          </span>
        </label>
      </div>
    </div>

    <!-- Información de la canción y letra -->
    <div v-if="lyrics" class="flex w-full max-w-7xl mx-auto">
      <!-- Información de la canción -->
      <div class="w-1/3 p-6 bg-white rounded-lg shadow-lg">
        <div class="flex items-center space-x-4">
          <img :src="currentSong.image.src" :alt="currentSong.image.alt" class="w-32 h-32 object-cover rounded-lg" />
          <div>
            <h2 class="text-2xl font-bold text-gray-800">{{ currentSong.title }}</h2>
            <p class="text-gray-600"><span class="font-medium">Artista:</span> {{ currentSong.description }}</p>
            <p class="text-gray-600"><span class="font-medium">Álbum:</span> {{ currentSong.album.title }}</p>
            <p class="text-gray-600"><span class="font-medium">Duración:</span> {{ currentSong.duration }}</p>
          </div>
        </div>
      </div>

      <!-- Letra de la canción -->
      <div class="w-2/3 p-6 bg-gray-50 rounded-lg shadow-lg">
        <!-- Opciones de tamaño de letra -->
        <div class="mb-4 text-center">
          <p class="text-gray-600 mb-2 font-medium">Tamaño de letra:</p>
          <label class="inline-flex items-center mx-2">
            <input type="radio" name="fontSize" value="small" v-model="selectedFontSize" class="mr-2" />
            Pequeña
          </label>
          <label class="inline-flex items-center mx-2">
            <input type="radio" name="fontSize" value="medium" v-model="selectedFontSize" class="mr-2" />
            Mediana
          </label>
          <label class="inline-flex items-center mx-2">
            <input type="radio" name="fontSize" value="large" v-model="selectedFontSize" class="mr-2" />
            Grande
          </label>
        </div>

        <pre class="whitespace-pre-wrap bg-gray-100 p-6 rounded-lg shadow text-left" :class="fontSizeClass">
          {{ lyrics }}
        </pre>
      </div>
    </div>

    <!-- Botón para volver -->
    <div v-if="lyrics" class="flex justify-center mt-6">
      <button @click="resetSearch"
        class="inline-block w-1/2 text-center rounded-lg border-2 border-red-600 bg-red-600 text-white py-2 text-sm font-semibold transition-all hover:bg-transparent hover:text-red-600 focus:outline-none focus:ring focus:ring-red-300">
        Volver a buscar
      </button>
    </div>

    <!-- Resultados de canciones -->
    <div v-else class="grid grid-cols-1 gap-4 lg:grid-cols-4 md:grid-cols-2 lg:gap-8 w-7/12 mx-auto">
      <cardSongVue v-for="(data, index) in results" :key="index" :id="data.id" :title="data.title"
        :description="data.description" :duration="data.duration" :explicit="data.explicit" :preview="data.preview"
        :album="data.album" :image="data.image" @fetch-lyrics="fetchLyrics(index)"></cardSongVue>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import axios from 'axios';
import cardSongVue from './components/cardSong.vue';

const results = ref([]);
const inputValue = ref('');
const lyrics = ref(null);
const currentSong = ref(null);
const selectedFontSize = ref('medium'); // Tamaño de letra seleccionado

let debounceTimer = null;

function onKeyDown() {
  clearTimeout(debounceTimer);
  debounceTimer = setTimeout(() => {
    searchSong(inputValue.value);
  }, 300);
}

async function searchSong(searchQuery) {
  lyrics.value = null;
  currentSong.value = null;
  if (!searchQuery) {
    results.value = [];
    return;
  }

  try {
    const response = await axios.get(
      `https://api.lyrics.ovh/suggest/${searchQuery}&limit=0`
    );
    const data = response.data.data;

    results.value = data.map((song) => ({
      id: song.id,
      title: song.title,
      description: song.artist.name,
      duration: `${Math.floor(song.duration / 60)}:${String(song.duration % 60).padStart(2, '0')}`,
      explicit: song.explicit_lyrics,
      preview: song.preview,
      album: {
        title: song.album.title,
        cover: song.album.cover_medium,
        link: song.album.tracklist,
      },
      image: {
        src: song.album.cover_medium,
        alt: `Cover of ${song.album.title}`,
      },
    }));
  } catch (error) {
    console.error('Error al obtener canciones:', error.message);
    results.value = [];
  }
}

async function fetchLyrics(index) {
  const song = results.value[index];
  try {
    const response = await axios.get(
      `https://api.lyrics.ovh/v1/${song.description}/${song.title}`
    );
    lyrics.value = response.data.lyrics;
    currentSong.value = song;
    results.value = []; // Limpia los resultados para mostrar solo la letra
  } catch (error) {
    console.error('Error al obtener las letras:', error.message);
    lyrics.value = 'No se encontraron letras para esta canción.';
  }
}

function resetSearch() {
  lyrics.value = null;
  currentSong.value = null;
}

const fontSizeClass = computed(() => {
  switch (selectedFontSize.value) {
    case 'small':
      return 'text-sm';
    case 'medium':
      return 'text-lg';
    case 'large':
      return 'text-2xl';
    default:
      return 'text-base';
  }
});
</script>

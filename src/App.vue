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

    <!-- Alerta: Sin resultados -->
    <div v-if="showNoResultsAlert" class="mb-6 text-center text-red-600 font-medium">
      No se encontraron resultados para tu búsqueda.
    </div>

    <!-- Alerta: Sin letras -->
    <div v-if="showNoLyricsAlert" class="mb-6 text-center text-yellow-600 font-medium">
      La canción seleccionada no tiene letras disponibles.
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

        <pre class="whitespace-pre-wrap bg-gray-100 p-6 rounded-lg shadow text-center"
          :class="fontSizeClass">{{ lyrics }}</pre>
      </div>
    </div>

    <!-- Resultados de canciones -->
    <div v-else class="grid grid-cols-1 gap-4 lg:grid-cols-4 md:grid-cols-2 lg:gap-8 w-7/12 mx-auto">
      <!-- Mostrar resultados de búsqueda -->
      <cardSongVue v-for="(data, index) in results" :key="index" :id="data.id" :title="data.title"
        :description="data.description" :duration="data.duration" :explicit="data.explicit" :preview="data.preview"
        :album="data.album" :image="data.image" @fetch-lyrics="fetchLyrics(data)" />
    </div>
    <!-- Mostrar canciones recientes -->
    <div v-if="!results.length && recentSongs.length">
      <h2 class="text-xl font-bold text-gray-800 mb-4 flex justify-center">Últimas canciones vistas:</h2>
      <div class="grid grid-cols-1 gap-4 lg:grid-cols-4 md:grid-cols-2 lg:gap-8 w-7/12 mx-auto">
        <cardSongVue v-for="(data, index) in recentSongs" :key="'recent-' + index" :id="data.id" :title="data.title"
          :description="data.description" :duration="data.duration" :explicit="data.explicit" :preview="data.preview"
          :album="data.album" :image="data.image" @fetch-lyrics="fetchLyrics(data)" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import axios from 'axios';
import { debounce } from 'lodash';
import cardSongVue from './components/cardSong.vue';

// Variables reactivas
const results = ref([]);
const recentSongs = ref(getRecentSongsFromStorage());
const inputValue = ref('');
const lyrics = ref(null);
const currentSong = ref(null);
const selectedFontSize = ref('medium');

// Alertas
const showNoResultsAlert = ref(false);
const showNoLyricsAlert = ref(false);

// Configuración
const MAX_RECENT_SONGS = 5;

// Búsqueda con debounce
const debouncedSearch = debounce(searchSong, 300);

function onKeyDown() {
  showNoResultsAlert.value = false;
  showNoLyricsAlert.value = false;
  debouncedSearch(inputValue.value);
}

async function searchSong(searchQuery) {
  if (!searchQuery) {
    results.value = [];
    lyrics.value = null;
    return;
  }

  try {
    const response = await axios.get(`https://api.lyrics.ovh/suggest/${searchQuery}&limit=0`);
    if (response.data.total === 0) {
      showNoResultsAlert.value = true;
      results.value = [];
    } else {
      results.value = response.data.data.map(mapSongData);
      showNoResultsAlert.value = false;
    }
  } catch (error) {
    console.error('Error al buscar canciones:', error.message);
    results.value = [];
    showNoResultsAlert.value = true;
  }
}

async function fetchLyrics(song) {
  try {
    const response = await axios.get(`https://api.lyrics.ovh/v1/${song.description}/${song.title}`);
    lyrics.value = response.data.lyrics || null;

    if (!lyrics.value) {
      showNoLyricsAlert.value = true;
    } else {
      showNoLyricsAlert.value = false;
    }

    currentSong.value = song;
    results.value = [];
    addToRecentSongs(song);
  } catch (error) {
    if (error.response?.status === 404) {
      console.warn('Letra no encontrada:', error.response.data.error);
      showNoLyricsAlert.value = true;
    } else {
      console.error('Error al obtener letras:', error.message);
    }

    lyrics.value = null;
  }
}

function mapSongData(song) {
  return {
    id: song.id,
    title: song.title,
    description: song.artist.name,
    duration: formatDuration(song.duration),
    explicit: song.explicit_lyrics,
    preview: song.preview,
    album: { title: song.album.title, cover: song.album.cover_medium, link: song.album.tracklist },
    image: { src: song.album.cover_medium, alt: `Cover of ${song.album.title}` },
  };
}

function getRecentSongsFromStorage() {
  return JSON.parse(localStorage.getItem('recentSongs')) || [];
}

function saveRecentSongsToStorage(songs) {
  localStorage.setItem('recentSongs', JSON.stringify(songs));
}

function addToRecentSongs(song) {
  if (!recentSongs.value.some((s) => s.id === song.id)) {
    recentSongs.value.unshift(song);
    if (recentSongs.value.length > MAX_RECENT_SONGS) {
      recentSongs.value.pop();
    }
    saveRecentSongsToStorage(recentSongs.value);
  }
}

function formatDuration(seconds) {
  return `${Math.floor(seconds / 60)}:${String(seconds % 60).padStart(2, '0')}`;
}

const fontSizeClass = computed(() => {
  const sizes = { small: 'text-sm', medium: 'text-lg', large: 'text-2xl' };
  return sizes[selectedFontSize.value] || 'text-base';
});
</script>

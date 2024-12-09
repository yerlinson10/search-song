<template>
    <div class="max-w-sm mx-auto rounded-lg shadow-lg bg-gradient-to-br from-white to-gray-100 p-6">
        <!-- Imagen de la canción -->
        <img :src="image?.src || 'https://via.placeholder.com/150'" :alt="image?.alt || 'Sin descripción'"
            class="w-full h-48 object-cover rounded-lg mb-4 shadow-md" />
        <!-- Título de la canción -->
        <h2 class="text-2xl font-bold text-gray-800 mb-1">{{ title }}</h2>
        <!-- Descripción del artista -->
        <p class="text-gray-600 text-sm mb-2"><span class="font-medium">Artista:</span> {{ description }}</p>
        <!-- Información del álbum -->
        <p class="text-gray-600 text-sm mb-2"><span class="font-medium">Álbum:</span> {{ album.title }}</p>
        <!-- Duración de la canción -->
        <p class="text-gray-600 text-sm mb-2"><span class="font-medium">Duración:</span> {{ duration }}</p>
        <!-- Contenido explícito -->
        <p class="text-sm font-medium mb-4" :class="{ 'text-red-500': explicit, 'text-green-500': !explicit }">
            Contenido explícito: {{ explicit ? 'Sí' : 'No' }}
        </p>
        <!-- Botón de reproducción -->
        <audio v-if="preview" controls :src="preview" class="w-full mt-2"></audio>
        <!-- Botón de letras -->
        <button
            class="inline-block mt-4 w-full text-center rounded-lg border-2 border-indigo-600 bg-indigo-600 text-white py-2 text-sm font-semibold transition-all hover:bg-transparent hover:text-indigo-600 focus:outline-none focus:ring focus:ring-indigo-300"
            @click="onFetchLyrics">
            Ver letras
        </button>
    </div>
</template>

<script>
export default {
    props: {
        id: {
            type: Number,
            required: true,
        },
        title: {
            type: String,
            required: true,
        },
        description: {
            type: String,
            required: true,
        },
        duration: {
            type: String,
            required: true,
        },
        explicit: {
            type: Boolean,
            required: true,
        },
        preview: {
            type: String,
            required: false,
        },
        album: {
            type: Object,
            required: true,
        },
        image: {
            type: Object,
            required: false,
        },
    },
    emits: ['fetch-lyrics'],
    methods: {
        onFetchLyrics() {
            this.$emit('fetch-lyrics');
        },
    },
};
</script>

<style scoped>
img {
    transition: transform 0.3s ease, box-shadow 0.3s ease;
}

img:hover {
    transform: scale(1.05);
    box-shadow: 0 8px 16px rgba(0, 0, 0, 0.2);
}

button:hover {
    transform: scale(1.02);
}
</style>
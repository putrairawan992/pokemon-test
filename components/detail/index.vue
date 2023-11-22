<!-- PokemonDetail.vue -->

<template>
  <div class="container mx-auto">
    <div v-if="pokemon" class="border p-10 rounded-xl">
      <h2 class="text-3xl font-bold mb-4">{{ pokemon.name }}</h2>
      <img :src="pokemon.image" :alt="pokemon.name" class="w-48 h-48 mb-4" />

      <div v-if="pokemon.types">
        <p class="mb-2"><strong>Types:</strong> {{ pokemon.types }}</p>
      </div>

      <div v-if="pokemon.height">
        <p class="mb-2"><strong>Height:</strong> {{ pokemon.height }}</p>
      </div>

      <div v-if="pokemon.weight">
        <p class="mb-2"><strong>Weight:</strong> {{ pokemon.weight }}</p>
      </div>

      <div v-if="pokemon.abilities">
        <p class="mb-2"><strong>Abilities:</strong> {{ pokemon.abilities }}</p>
      </div>

      <div v-if="pokemon.base_experience">
        <p class="mb-2"><strong>Base Experience:</strong> {{ pokemon.base_experience }}</p>
      </div>

      <div v-if="localCatchDateTime">
        <p class="mb-2"><strong>Captured at:</strong> {{ localCatchDateTime }}</p>
      </div>

      <div v-if="localIsFavorite">
        <p class="mb-2"><strong>Favorited at:</strong> {{ localFavoriteDate }}</p>
      </div>

      <button
        @click="toggleCatch"
        class="px-3 py-1 rounded bg-blue-500 text-white mb-2"
      >
        Catch
      </button>
      <button
        @click="toggleFavorite"
        class="px-3 py-1 ml-3 rounded bg-yellow-500 text-white mb-2"
      >
        Favorite
      </button>
    </div>
  </div>
</template>

<script>
import { ref, watch } from 'vue';

export default {
  props: ["pokemon"],
  setup(props, { emit }) {
    const localCatchDateTime = ref(props.pokemon.catchDateTime || null);
    const localFavoriteDate = ref(props.pokemon.favoriteDate || null);
    const localIsFavorite = ref(props.pokemon.isFavorite || false);

    const toggleCatch = () => {
      if (!localCatchDateTime.value) {
        localCatchDateTime.value = new Date().toLocaleString();
        emit("pokemonCaught", {
          name: props.pokemon.name,
          catchDateTime: localCatchDateTime.value,
        });
      }
    };

    const toggleFavorite = () => {
      localIsFavorite.value = !localIsFavorite.value;
      localFavoriteDate.value = localIsFavorite.value ? new Date().toLocaleString() : null;
      emit("pokemonFavorited", {
        name: props.pokemon.name,
        favoriteDate: localFavoriteDate.value,
        isFavorite: localIsFavorite.value,
      });
    };

    watch(() => props.pokemon, () => {
      localCatchDateTime.value = props.pokemon.catchDateTime || null;
      localFavoriteDate.value = props.pokemon.favoriteDate || null;
      localIsFavorite.value = props.pokemon.isFavorite || false;
    });

    return {
      localCatchDateTime,
      localFavoriteDate,
      localIsFavorite,
      toggleCatch,
      toggleFavorite,
    };
  },
};
</script>

<style>
/* TailwindCSS classes or custom styling */
</style>

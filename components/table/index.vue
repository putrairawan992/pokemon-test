<!-- PokemonList.vue -->

<template>
  <div class="container mx-auto p-4">
    <h1 class="text-3xl font-bold mb-4">List of Pokemon</h1>
    <div v-if="selectedPokemon">
      <button @click="selectedPokemon = null; pokemonDetails = null" class="mb-4 px-3 py-1 rounded bg-blue-500 text-white">Back to List</button>
      <Detail :pokemon="selectedPokemon" @pokemonCaught="handlePokemonCaught" @pokemonFavorited="handlePokemonFavorited" />
    </div>
    <div v-else>
      <div class="flex">
        <div class="flex items-center mb-4">
          <label for="sortSelect" class="mr-2">Sort by:</label>
          <select id="sortSelect" v-model="sortOrder" @change="sortPokemon" class="px-2 py-1 rounded bg-gray-200">
            <option value="name_asc">Name (Ascending)</option>
            <option value="name_desc">Name (Descending)</option>
            <option value="type_name_asc">Type & Name (Ascending)</option>
            <option value="type_name_desc">Type & Name (Descending)</option>
          </select>
        </div>
        <div class="flex items-center mb-4 ml-4">
          <label for="typeSearch" class="mr-2">Search by Type:</label>
          <input v-model="typeFilter" id="typeSearch" type="text" class="px-2 py-1 rounded bg-gray-200" @input="filterByType" />
        </div>
      </div>
      <div class="grid grid-cols-2 gap-4">
        <div v-for="(pokemon, index) in filteredPokemon" :key="index" @click="showPokemonDetail(pokemon.url)" class="border p-4 cursor-pointer">
          <img :src="pokemon.image" :alt="pokemon.name" class="w-24 h-24 mb-2" />
          <p class="font-semibold">{{ pokemon.name }}</p>
          <p v-if="pokemon.types?.length" class="text-sm text-gray-500">{{ pokemon.types.join(', ') }}</p>
          <p v-if="pokemon.catchDateTime" class="text-sm text-green-500">Caught at: {{ pokemon.catchDateTime }}</p>
          <p v-if="pokemon.isFavorite" class="text-sm text-yellow-500">Favorited at {{ pokemon.favoriteDate }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, computed } from 'vue';
import Detail from "../detail";

export default {
  components: {
    Detail,
  },
  setup() {
    const pokemonList = ref([]);
    const currentPage = ref(1);
    const itemsPerPage = ref(10);
    const selectedPokemon = ref(null);
    const pokemonDetails = ref(null);
    const sortOrder = ref('name_asc');
    const typeFilter = ref('');

    const fetchPokemonData = async () => {
      try {
        const response = await fetch('https://pokeapi.co/api/v2/pokemon');
        const data = await response.json();
        const results = data.results;

        for (const pokemon of results) {
          const urlParts = pokemon.url.split('/');
          const pokemonId = urlParts[urlParts.length - 2];
          const pokemonImageUrl = `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${pokemonId}.png`;

          const typesResponse = await fetch(`https://pokeapi.co/api/v2/pokemon/${pokemonId}`);
          const typesData = await typesResponse.json();

          pokemonList.value.push({
            name: pokemon.name,
            image: pokemonImageUrl,
            url: pokemon.url,
            types: typesData.types.map(type => type.type.name), 
          });
        }
      } catch (error) {
        console.error('Error fetching Pokemon data:', error);
      }
    };

    onMounted(fetchPokemonData);

    const totalPages = computed(() => Math.ceil(pokemonList.value.length / itemsPerPage.value));

    const displayedPokemon = computed(() => {
      const start = (currentPage.value - 1) * itemsPerPage.value;
      const end = currentPage.value * itemsPerPage.value;
      return pokemonList.value.slice(start, end).map(pokemon => ({
        ...pokemon,
        types: pokemon.types || [],
        catchDateTime: pokemon.catchDateTime || null,
        isFavorite: pokemon.isFavorite || false,
        favoriteDate: pokemon.favoriteDate || null,
      }));
    });

    const sortedPokemon = computed(() => {
      const sorted = [...pokemonList.value];
      if (sortOrder.value === 'name_asc') {
        sorted.sort((a, b) => (a.name > b.name ? 1 : -1));
      } else if (sortOrder.value === 'name_desc') {
        sorted.sort((a, b) => (a.name < b.name ? 1 : -1));
      } else if (sortOrder.value === 'type_name_asc') {
        sorted.sort((a, b) => {
          if (a.types.join(', ') === b.types.join(', ')) {
            return a.name > b.name ? 1 : -1;
          }
          return a.types.join(', ') > b.types.join(', ') ? 1 : -1;
        });
      } else if (sortOrder.value === 'type_name_desc') {
        sorted.sort((a, b) => {
          if (a.types.join(', ') === b.types.join(', ')) {
            return a.name < b.name ? 1 : -1;
          }
          return a.types.join(', ') < b.types.join(', ') ? 1 : -1;
        });
      }
      return sorted;
    });

    const filteredPokemon = computed(() => {
      if (!typeFilter.value) {
        return sortedPokemon.value;
      }
      const searchTerm = typeFilter.value.toLowerCase();
      return sortedPokemon.value.filter(pokemon => pokemon.types.some(type => type.toLowerCase().includes(searchTerm)));
    });

    const showPokemonDetail = async (url) => {
      try {
        const response = await fetch(url);
        const data = await response.json();

        const selected = pokemonList.value.filter((item) => item.url === url);

        selectedPokemon.value = {
          name: data.name,
          image: `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${data.id}.png`,
          height: data.height,
          weight: data.weight,
          abilities: getAbilities(data.abilities),
          base_experience: data.base_experience,
          type: getTypes(data.types),
          catchDateTime: selected[0]?.catchDateTime || null,
          isFavorite: selected[0]?.isFavorite || false,
          favoriteDate: selected[0]?.favoriteDate || null,
        };
      } catch (error) {
        console.error('Error fetching Pokemon details:', error);
      }
    };

    const handlePokemonCaught = (data) => {
      const index = pokemonList.value.findIndex(pokemon => pokemon.name === data.name);
      if (index !== -1) {
        pokemonList.value[index].catchDateTime = data.catchDateTime;
      }
    };

    const handlePokemonFavorited = (data) => {
      const index = pokemonList.value.findIndex(pokemon => pokemon.name === data.name);
      if (index !== -1) {
        pokemonList.value[index].isFavorite = data.isFavorite;
        pokemonList.value[index].favoriteDate = data.favoriteDate;
      }
    };

    const getAbilities = (abilities) => {
      return abilities.map(ability => ability.ability.name).join(', ');
    };

    const getTypes = (types) => {
      return types.map(type => type.type.name).join(', ');
    };

    const sortPokemon = () => {
      // Method to trigger sort changes
      // sortedPokemon will be automatically updated by computed property
    };

    const filterByType = () => {
      // Method to trigger type search changes
      // filteredPokemon will be automatically updated by computed property
    };

    return {
      pokemonList,
      currentPage,
      itemsPerPage,
      selectedPokemon,
      pokemonDetails,
      sortOrder,
      typeFilter,
      totalPages,
      displayedPokemon,
      sortedPokemon,
      filteredPokemon,
      showPokemonDetail,
      handlePokemonCaught,
      handlePokemonFavorited,
      sortPokemon,
      filterByType,
    };
  },
};
</script>

<style>
/* TailwindCSS classes or custom styling */
</style>

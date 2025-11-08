<script setup>
import { ref, onMounted } from "vue";
import logo from "./fondo/entrenador-pokemon.png";

const searchQuery = ref("");
const pokemon = ref(null);
const loading = ref(false);
const error = ref(null);

const searchPokemon = async () => {
  if (!searchQuery.value.trim()) return;

  loading.value = true;
  error.value = null;

  try {
    const response = await fetch(
      `https://pokeapi.co/api/v2/pokemon/${searchQuery.value.toLowerCase()}`
    );
    if (!response.ok) {
      throw new Error("Pokemon not found");
    }
    const data = await response.json();
    pokemon.value = data;
  } catch (err) {
    error.value = err.message;
    pokemon.value = null;
  } finally {
    loading.value = false;
  }
};

onMounted(() => {
  // Optionally load a default pokemon
});
</script>

<template>
  <div class="app">
    <header>
      <div class="header-left">Pokemón</div>
      <div class="header-center">
        <div class="search-input-container">
          <input
            v-model="searchQuery"
            @keyup.enter="searchPokemon"
            placeholder="Buscar Pokémon"
            type="text"
          />
          <button @click="searchPokemon" :disabled="loading">Buscar</button>
        </div>
      </div>
      <div class="header-right">
        <img :src="logo" alt="Logo" />
      </div>
    </header>
    <main>
      <div v-if="loading" class="loading">Loading...</div>
      <div v-if="error" class="error">{{ error }}</div>

      <div v-if="pokemon" class="pokemon-card">
        <div class="column image-column">
          <img :src="pokemon.sprites.front_default" :alt="pokemon.name" />
        </div>
        <div class="column info-column">
          <h2>
            {{ pokemon.name.charAt(0).toUpperCase() + pokemon.name.slice(1) }}
          </h2>
          <p>
            <strong>Types:</strong>
            {{ pokemon.types.map((t) => t.type.name).join(", ") }}
          </p>
          <p><strong>Height:</strong> {{ pokemon.height / 10 }} m</p>
          <p><strong>Weight:</strong> {{ pokemon.weight / 10 }} kg</p>
          <p>
            <strong>Abilities:</strong>
            {{ pokemon.abilities.map((a) => a.ability.name).join(", ") }}
          </p>
          <!-- Weaknesses would require additional API calls to type data -->
        </div>
        <div class="column stats-column">
          <h3>Base Stats</h3>
          <div v-for="stat in pokemon.stats" :key="stat.stat.name" class="stat">
            <span>{{ stat.stat.name.toUpperCase() }}:</span>
            <span>{{ stat.base_stat }}</span>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<style scoped>
.app {
  min-height: 100vh;
  padding: 20px;
  font-family: Arial, sans-serif;
}

header {
  position: absolute;
  top: 20px;
  left: 0;
  right: 0;
  display: flex;
  align-items: center;
  padding: 0 20px;
}

.header-left {
  flex: 0 0 200px;
  font-size: 24px;
  font-weight: bold;
  color: #c5c2c2;
}

.header-center {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
}

.search-input-container {
  position: relative;
  display: flex;
  align-items: center;
  margin-left: -80px;
}

.search-input-container input {
  padding: 10px 40px 10px 10px;
  width: 250px;
  border-radius: 30px;
  background-color: #07070765;
  border: none;
  color: #ffffff;
}

.search-input-container input:focus {
  border: none;
  outline: none;
}

.search-input-container button {
  position: absolute;
  right: 5px;
  padding: 5px;
  font-size: 15px;
  background-color: #022846b7;
  color: rgba(255, 255, 255, 0.513);
  border: none;
  border-radius: 30px;
  cursor: pointer;
}

.search-input-container button:focus {
  outline: none;
}

.search-input-container button:active {
  background-color: #0424448d;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.5);
}

.search-input-container button:disabled {
  color: #cccccc;
}

.header-right img {
  height: 50px;
  width: auto;
  animation: pulse 3s infinite;
}

@keyframes pulse {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.1);
  }
  100% {
    transform: scale(1);
  }
}

.search-container {
  position: absolute;
  top: 20px;
  right: 20px;
  display: flex;
  gap: 10px;
  align-items: center;
}

.search-container input {
  padding: 10px;
  width: 200px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.search-container button {
  padding: 10px;
  background-color: #4caf50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.search-container button:disabled {
  background-color: #cccccc;
}

input {
  padding: 10px;
  width: 300px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

button {
  padding: 10px 20px;
  background-color: #4caf50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:disabled {
  background-color: #cccccc;
}

.pokemon-card {
  display: grid;
  grid-template-columns: 1fr;
  gap: 20px;
  background-color: rgba(255, 255, 255, 0.9);
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  margin-top: 20px;
}

.column {
  padding: 10px;
}

.image-column {
  text-align: center;
}

.image-column img {
  max-width: 200px;
  height: auto;
}

.info-column h2 {
  margin-top: 0;
  color: #333;
}

.stat {
  display: flex;
  justify-content: space-between;
  margin-bottom: 5px;
}

.loading,
.error {
  text-align: center;
  margin-top: 20px;
  font-size: 18px;
}

.error {
  color: red;
}
</style>

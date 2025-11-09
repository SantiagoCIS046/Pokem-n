<script setup>
import { ref, onMounted, computed } from "vue";
import logo from "./fondo/entrenador-pokemon.png";

const searchQuery = ref("");
const pokemon = ref(null);
const weaknesses = ref([]);
const loading = ref(false);
const error = ref(null);

const typeColor = computed(() => {
  if (!pokemon.value || !pokemon.value.types.length) return "#333";
  const typeName = pokemon.value.types[0].type.name;
  const typeColors = {
    normal: "#A8A878",
    fire: "#F08030",
    water: "#6890F0",
    electric: "#F8D030",
    grass: "#78C850",
    ice: "#98D8D8",
    fighting: "#C03028",
    poison: "#A040A0",
    ground: "#E0C068",
    flying: "#A890F0",
    psychic: "#F85888",
    bug: "#A8B820",
    rock: "#B8A038",
    ghost: "#705898",
    dragon: "#7038F8",
    dark: "#705848",
    steel: "#B8B8D0",
    fairy: "#EE99AC",
  };
  return typeColors[typeName.toLowerCase()] || "#333";
});

const getTypeColor = (typeName) => {
  const typeColors = {
    normal: "#A8A878",
    fire: "#F08030",
    water: "#6890F0",
    electric: "#F8D030",
    grass: "#78C850",
    ice: "#98D8D8",
    fighting: "#C03028",
    poison: "#A040A0",
    ground: "#E0C068",
    flying: "#A890F0",
    psychic: "#F85888",
    bug: "#A8B820",
    rock: "#B8A038",
    ghost: "#705898",
    dragon: "#7038F8",
    dark: "#705848",
    steel: "#B8B8D0",
    fairy: "#EE99AC",
  };
  return typeColors[typeName.toLowerCase()] || "#333";
};

const getStatBarColor = (index) => {
  if (!pokemon.value || !pokemon.value.types.length) return "#4caf50";
  const types = pokemon.value.types;
  const typeIndex = index % types.length;
  const typeName = types[typeIndex].type.name;
  return getTypeColor(typeName);
};

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

    // Play Pokemon cry - use high-quality cry from pokemoncries.com, fallback to PokeAPI
    const cryUrl = `https://pokemoncries.com/cries/${data.id}.mp3`;
    const audio = new Audio(cryUrl);
    audio.volume = 0.5; // Set volume to 50%
    audio.play().catch((err) => {
      // Fallback to PokeAPI cry if pokemoncries.com fails
      if (data.cries && data.cries.latest) {
        const fallbackAudio = new Audio(data.cries.latest);
        fallbackAudio.volume = 0.5;
        fallbackAudio
          .play()
          .catch((fallbackErr) =>
            console.log("Fallback audio play failed:", fallbackErr)
          );
      } else {
        console.log("No cry available for this Pokemon");
      }
    });

    // Fetch weaknesses
    const typePromises = data.types.map(async (typeInfo) => {
      const typeResponse = await fetch(typeInfo.type.url);
      return await typeResponse.json();
    });
    const typeData = await Promise.all(typePromises);
    const allWeaknesses = new Set();
    typeData.forEach((type) => {
      type.damage_relations.double_damage_from.forEach((weak) => {
        allWeaknesses.add(weak.name);
      });
    });
    weaknesses.value = Array.from(allWeaknesses);
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
        <div
          class="column image-column"
          :style="{
            borderColor: typeColor,
            backgroundColor: typeColor,
          }"
        >
          <img
            :src="pokemon.sprites.other['official-artwork'].front_default"
            :alt="pokemon.name"
          />
          <h2
            class="pokemon-name"
            :style="{
              color: 'white',
              textShadow: `2px 2px 4px ${typeColor}`,
            }"
          >
            {{ pokemon.name.charAt(0).toUpperCase() + pokemon.name.slice(1) }}
          </h2>
          <p class="height">
            <strong>Height:</strong> {{ pokemon.height / 10 }} m
          </p>
          <p class="weight">
            <strong>Weight:</strong> {{ pokemon.weight / 10 }} kg
          </p>
        </div>
        <div class="column info-column">
          <p class="info-item">{{ pokemon.id }}</p>
          <div class="types-section">
            <strong>Types:</strong>
            <div class="types-list">
              <span
                v-for="type in pokemon.types"
                :key="type.type.name"
                class="type-badge"
                :style="{ backgroundColor: getTypeColor(type.type.name) }"
              >
                {{
                  type.type.name.charAt(0).toUpperCase() +
                  type.type.name.slice(1)
                }}
              </span>
            </div>
          </div>
          <div class="weaknesses-section">
            <strong>Weaknesses:</strong>
            <div class="weaknesses-list">
              <span
                v-for="weakness in weaknesses"
                :key="weakness"
                class="weakness-badge"
                :style="{ backgroundColor: getTypeColor(weakness) }"
              >
                {{ weakness.charAt(0).toUpperCase() + weakness.slice(1) }}
              </span>
            </div>
          </div>
        </div>
        <div class="column stats-column">
          <h3>Stats</h3>
          <div
            v-for="(stat, index) in pokemon.stats"
            :key="stat.stat.name"
            class="stat"
          >
            <div class="stat-name">{{ stat.stat.name.toUpperCase() }}</div>
            <div class="stat-bar-container">
              <div
                class="stat-bar"
                :style="{
                  width: (stat.base_stat / 255) * 100 + '%',
                  backgroundImage: `linear-gradient(to right, ${getStatBarColor(
                    index
                  )}, rgba(255, 255, 255, 0.3))`,
                }"
              ></div>
            </div>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap");

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
  font-family: "Press Start 2P", cursive;
  color: #ffd700;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.8);
  letter-spacing: 2px;
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
  grid-template-columns: 1fr 1fr 1fr;
  gap: 50px;
  margin-top: 50px;
}

.column {
  height: 100%;
  padding: 20px;
  border: 1px solid rgba(0, 0, 0, 0.28);
  border-radius: 1cqb;
  background-color: rgba(0, 0, 0, 0.53);
}

.image-column {
  text-align: center;
  position: relative;
}

.height {
  position: absolute;
  bottom: 10px;
  left: 10px;
  margin: 0;
  background-color: rgba(0, 0, 0, 0.8);
  color: #ffffff;
  padding: 5px 10px;
  border-radius: 10px;
  font-size: 14px;
  font-weight: bold;
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.2),
    0 4px 8px rgba(0, 0, 0, 0.3);
}

.weight {
  position: absolute;
  bottom: 10px;
  right: 10px;
  margin: 0;
  background-color: rgba(0, 0, 0, 0.8);
  color: #ffffff;
  padding: 5px 10px;
  border-radius: 10px;
  font-size: 14px;
  font-weight: bold;
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.2),
    0 4px 8px rgba(0, 0, 0, 0.3);
}

.image-column img {
  width: 250px;
  height: auto;
  filter: drop-shadow(0 0 15px rgba(88, 37, 37, 0.968));
  animation: pulse 2.5s infinite;
}

.info-column {
  text-align: center;
  background-color: #0000009a;
}

.info-column h2 {
  margin-top: 0;
  color: #333;
}

.stat {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 15px;
}

.stat-name {
  font-size: 13px;
  font-weight: bold;
  color: #fff;
  margin-bottom: 5px;
}

.stat-bar-container {
  width: 90%;
  height: 10px;
  background-color: rgba(255, 255, 255, 0.2);
  border-radius: 25px;
  overflow: hidden;
  margin-bottom: 5px;
}

.stat-bar {
  height: 100%;
  background-color: #4caf50;
  border-radius: 5px;
  transition: width 0.3s ease;
}

.stat-value {
  font-size: 12px;
  color: #fff;
  font-weight: bold;
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

.pokemon-name {
  margin: 10px 0;
  font-size: 28px;
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
  color: #ffd700;
  text-shadow: 3px 3px 6px rgba(0, 0, 0, 0.8);
  letter-spacing: 1px;
}

.info-item {
  margin-bottom: 15px;
  font-size: 72px;
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
  color: #fff;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.8);
}

.types-section,
.weaknesses-section {
  margin-bottom: 15px;
}

.types-section strong,
.weaknesses-section strong {
  display: block;
  margin-bottom: 8px;
  color: #fff;
  font-family: Arial, Helvetica, sans-serif;
  font-size: 16px;
  font-weight: bold;
}

.types-list,
.weaknesses-list {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  justify-content: center;
}

.type-badge,
.weakness-badge {
  padding: 6px 12px;
  border-radius: 20px;
  color: #fff;
  font-size: 14px;
  font-weight: bold;
  text-transform: capitalize;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
  cursor: pointer;
}

.type-badge:hover {
  transform: scale(1.1);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.5);
  color: black;
}

.weakness-badge:hover {
  transform: scale(1.1);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.5);
  color: black;
}

/* Mobile responsiveness */
@media (max-width: 768px) {
  .app {
    padding: 5px;
  }

  header {
    position: relative;
    top: 0;
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 5px;
  }

  .header-left {
    font-size: 16px;
    text-align: center;
    margin-bottom: 0.5px;
    order: -1;
  }

  .header-center {
    width: 100%;
    order: 1;
    margin-bottom: 5px;
  }

  .search-input-container {
    margin-left: 0;
    width: 100%;
  }

  .search-input-container input {
    width: 100%;
    padding: 10px 50px 10px 10px;
  }

  .header-right {
    display: none;
  }

  .pokemon-card {
    grid-template-columns: 1fr;
    gap: 0px;
    margin-top: 0;
  }

  .column {
    padding: 0px;
    top: 10px;
    height: auto;
    min-height: auto;
    margin-top: 10%;
  }

  .image-column {
    order: -1;
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .image-column img {
    width: 100%;
    max-width: 180px;
  }

  .height,
  .weight {
    position: static;
    margin: 5px 10px;
    display: inline-block;
  }

  .pokemon-name {
    font-size: 22px;
  }

  .info-item {
    font-size: 40px;
  }

  .stat-name {
    font-size: 11px;
  }

  .type-badge,
  .weakness-badge {
    font-size: 11px;
    padding: 3px 6px;
  }

  .height,
  .weight {
    font-size: 12px;
    padding: 3px 6px;
  }
}

/* Extra small screens (e.g., 400px width) */
@media (max-width: 480px) {
  .header-left {
    font-size: 14px;
  }

  .search-input-container input {
    font-size: 14px;
  }

  .search-input-container button {
    font-size: 12px;
  }
}
</style>

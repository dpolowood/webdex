<script setup>
import { ref, reactive } from "vue";

var query = ref("bulbasaur");
const state = reactive({
  pokemon: {
    name: {},
    height: {},
    weight: {},
    id: {},
  },
  species: {},
  genera: { genus: {} },
  flavor_text_entry: { flavor_text: {} },
});

var requestOptions = {
  method: "GET",
  redirect: "follow",
};

fetchPokemon();
function fetchPokemon() {
  if (!query.value?.trim()) return;
  query.value = query.value.toLowerCase();
  fetch(`https://pokeapi.co/api/v2/pokemon/${query.value}`, requestOptions)
    .then((response) => {
      return response.json();
    })
    .then(setResults)
    .catch((error) => console.log("error", error));
  query.value = "";
}

function setResults(results) {
  state.pokemon = results;
  state.sprite = state.pokemon.sprites.other["official-artwork"].front_default;
  state.pokemon.weight = (state.pokemon.weight * 0.1).toFixed(1);
  state.pokemon.height = (state.pokemon.height * 0.1).toFixed(1);
  fetch(`${state.pokemon.species.url}`, requestOptions)
    .then((response) => {
      return response.json();
    })
    .then(setSpeciesInfo)
    .catch((error) => console.log("error", error));
}

function setSpeciesInfo(speciesInfo) {
  state.species = speciesInfo;
  state.flavor_text_entry = state.species.flavor_text_entries.find(
    checkLanguageFlavorText
  );
  state.flavor_text_entry.flavor_text =
    state.flavor_text_entry.flavor_text.replace("\f", " ");
  state.genera = state.species.genera.find(checkLanguageGenera);
}

function checkLanguageGenera(language) {
  if (language.language.name == "en") return language;
}

function checkLanguageFlavorText(language) {
  if (language.language.name == "en") return language;
}
function goToLast() {
  query = ref(pokemon.id - 1);
  fetchPokemon();
}
function goToNext() {
  query = ref(pokemon.id + 1);
  fetchPokemon();
}
</script>

<template>
  <div class="pokedex">
    <div class="topbar">
      <div class="title">
        <div class="number">
          <p>No. {{ state.pokemon.id }}</p>
        </div>
        <header>{{ state.pokemon.name }}</header>
      </div>
      <div class="species">
        <div class="genera">{{ state.genera.genus }}</div>
        <ul id="types">
          <li v-for="type in state.pokemon.types" :key="type.type.name">
            {{ type.type.name.toUpperCase() }}
          </li>
        </ul>
      </div>
    </div>
    <div class="main">
      <div class="image"><img :src="state.sprite" /></div>
      <div class="attributes">
        <p class="stats">Weight: {{ state.pokemon.weight }} kg</p>
        <p class="stats">Height: {{ state.pokemon.height }} m</p>
        <p class="description">
          {{ state.flavor_text_entry.flavor_text }}
        </p>
      </div>
    </div>
    <div class="bottom">
      <button  v-on:click="goToLast"> last </button>
      <input
        v-model="query"
        placeholder="Enter pokemon name or number"
        @keyup.enter="fetchPokemon"
      />
      <button  v-on:click="goToNext" > next </button>
    </div>
  </div>
</template>

<style>
@import "./assets/base.css";

#app {
  max-width: 1280px;
  min-width: 768px;
  margin: 0 auto;
  padding: 2rem;
  font-weight: normal;
}

.pokedex {
  display: flex;
  background-color: #e8e6cf;
  flex-direction: column;
}

.topbar {
  display: flex;
  flex-direction: row;
  flex-wrap: nowrap;
  background-color: #324357;
  height: 50px;
  margin: 20px;
  color: #c3c9cf;
  border-radius: 10px;
  align-items: center;
  justify-content: space-around;
}

.main {
  display: flex;
  flex-direction: row;
  flex-wrap: nowrap;
  justify-content: space-around;
  background-color: #e8e6cf;
  margin: 10px 0px 20px 0px;
}

.attributes {
  display: flex;
  flex-direction: column;
  flex: 0 1 25%;
  background-color: #e8e6cf;
  color: #8a91a0;
  justify-content: center;
}

.stats {
  padding: 0px 0px 0px 15px;
}

.description {
  background-color: #faf6e3;
  margin: 10px;
  padding: 10px;
  color: #8a91a0;
  border-radius: 10px;
  flex-grow: 6;
}

.title {
  display: flex;
  flex-direction: row;
  align-items: center;
  padding: 5px 5px 5px 5px;
}

.species {
  display: flex;
  flex-direction: row;
  flex-wrap: nowrap;
  align-items: center;
}
/* top right bottom left*/
.genera {
  border-radius: 5px;
  padding: 5px 30px 5px 30px;
  margin: 0px 10px 0px 10px;
  background-color: #485663;
  font-size: 1.2rem;
}

li {
  background-color: #343e3d;
  padding: 5px 5px;
  margin: 0px 5px 0px 5px;
  font-weight: 500;
}

.image {
  flex: 0 2 40%;
  background-color: white;
}

header {
  font-size: 2rem;
  margin: 5px;
  font-weight: bold;
}

header::first-letter {
  text-transform: capitalize;
}

.number {
  font-size: 1.2rem;
  border: 2px solid #c3c9cf;
  border-radius: 10px;
  margin: 5px;
}

ul {
  list-style: none;
  display: flex;
  flex-direction: row;
}

.bottom {
  display: flex;
  justify-content: center;
  margin: 10px 0px 20px 0px;
}

input {
  padding: 10px 10px 10px 10px;
  font-size: 1rem;
  background-color: #485663;
  color: white;
}

button {
  margin: 0px 10px 0px 10px;
  padding: 10px 10px 10px 10px;
  background-clip: #485663;
}

::placeholder {
  color: #c3c9cf;
}
</style>

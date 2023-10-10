<template>
  <div class="barrasupe">
    <img
      style="background-image: url(../src/images/banner.png); background-size: cover; width: 100%; height: 200px; background-position: center; " />
    <div class="cen"><input placeholder="buscar pokemon" type="text" v-model="filtroNombre" />
      <select v-model="filtroTipo">
        <option value="">Todos los tipos</option>
        <option v-for="tipo in tiposPokemon" :key="tipo" :value="tipo">{{ tipo }}</option>
      </select>
    </div>


  </div>

  <div>
    <div class="target">
      <div class="center" v-for="pokemon in pokemonsFiltrados" :key="pokemon.id">
        <p>#{{ pokemon.num }} {{ pokemon.nombre }}</p>
        <img :src="pokemon.img" style="width: 330px; height: 330px;" />
        <button class="button" data-bs-toggle="modal" data-bs-target="#pokemonModal" @click="mostrarModal(pokemon)">
          Información
        </button>
        <div class="types">
          <span v-for="tipo in pokemon.tipos" :key="tipo" :class="tipo.toLowerCase()">
            {{ tipo }}
          </span>
        </div>
      </div>
    </div>

    <div class="modal" id="pokemonModal" tabindex="-1" aria-labelledby="pokemonModalLabel" aria-hidden="true">
      <div class="modal-dialog modal-dialog-centered">
        <div class="modal-content">
          <div class="modal-header">
            <div class="modti">
              <h5 class="modal-title" id="pokemonModalLabel">
                #{{ num }} {{ nombre }}
              </h5>
            </div>
          </div>
          <div class="modal-body">
            <div id="part1" :style="c ? 'display:grid' : 'display: none'">
              <div class="centmo">
                <h3>peso: {{ peso }} Kg</h3>
                <h3>altura: {{ altura }}M</h3>
                <img :src="img" style="width: 280px; height: 280px;" />
                <div class="stats">
                  <h4 style="display: flex; gap: 10px;">
                    HP <progress :value="hp" max="300"></progress> {{ hp }}
                  </h4>
                  <h4 style="display: flex; gap: 10px;">
                    ATTACK<progress :value="ataque" max="300"></progress> {{ ataque }}
                  </h4>
                  <h4 style="display: flex; gap: 10px;">
                    DEFENSE<progress :value="d" max="300"></progress> {{ d }}
                  </h4>
                  <h4 style="display: flex; gap: 10px;">
                    SPECIAL ATTACK<progress :value="s" max="300"></progress> {{ s }}
                  </h4>
                  <h4 style="display: flex; gap: 10px;">
                    SPECIAL DEFENSE<progress :value="sd" max="300"></progress> {{ sd }}
                  </h4>
                  <h4 style="display: flex; gap: 10px;">
                    SPEED<progress :value="velocidad" max="300"></progress> {{ velocidad }}
                  </h4>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import axios from "axios";
import { ref, onMounted, computed } from "vue";

const c = ref(false);
const filtroNombre = ref("");
const filtroTipo = ref("");

const img = ref("");
const nombre = ref("");
const num = ref("");
const peso = ref("");
const altura = ref("");
const hp = ref(0);
const ataque = ref(0);
const d = ref(0);
const s = ref(0);
const sd = ref(0);
const velocidad = ref(0);

const pokemons = ref([]);

async function obtenerUrlPokemon(pokemonId) {
  try {
    let r = await axios.get(`https://pokeapi.co/api/v2/pokemon/${pokemonId}/`);
    let species = await axios.get(r.data.species.url);
    c.value = true;
    const nuevoPokemon = {
      num: r.data.id,
      nombre: species.data.names.find((name) => name.language.name === "es").name,
      img: r.data.sprites.other["official-artwork"].front_default,
      tipos: r.data.types.map((e) => e.type.name),
      peso: r.data.weight / 10,
      altura: r.data.height / 10,
      hp: r.data.stats[0].base_stat,
      ataque: r.data.stats[1].base_stat,
      d: r.data.stats[2].base_stat,
      s: r.data.stats[3].base_stat,
      sd: r.data.stats[4].base_stat,
      velocidad: r.data.stats[5].base_stat,
    };
    pokemons.value.push(nuevoPokemon);
  } catch (error) {
    console.error("Error al obtener información del Pokémon:", error);
    throw error;
  }
}

onMounted(async () => {
  for (let i = 1; i <= 50; i++) {
    await obtenerUrlPokemon(i);
  }
});

const mostrarModal = (pokemon) => {
  nombre.value = pokemon.nombre;
  num.value = pokemon.num;
  img.value = pokemon.img;
  peso.value = pokemon.peso;
  altura.value = pokemon.altura;
  hp.value = pokemon.hp;
  ataque.value = pokemon.ataque;
  d.value = pokemon.d;
  s.value = pokemon.s;
  sd.value = pokemon.sd;
  velocidad.value = pokemon.velocidad;

  $('#pokemonModal').modal('show');
};

const tiposPokemon = computed(() => {
  const tipos = new Set();
  pokemons.value.forEach((pokemon) => {
    pokemon.tipos.forEach((tipo) => {
      tipos.add(tipo);
    });
  });
  return Array.from(tipos);
});

const pokemonsFiltrados = computed(() => {
  let filtrados = pokemons.value;
  if (filtroNombre.value) {
    filtrados = filtrados.filter((pokemon) =>
      pokemon.nombre.toLowerCase().includes(filtroNombre.value.toLowerCase())
    );
  }
  if (filtroTipo.value) {
    filtrados = filtrados.filter((pokemon) =>
      pokemon.tipos.includes(filtroTipo.value)
    );
  }
  return filtrados;
});
</script>

<style scoped>

.cen{
    display: flex;
    justify-content: center;
    padding: 2%;
    gap: 80px;
}

.types {
  display: flex;
  gap: 25px;
}

span {

  display: flex;
  border-radius: 5px;
  width: 60px;
  height: 30px;
  justify-content: center;
  font-size: 16px;
  font-weight: bold;
  box-shadow: 0px 0px 15px 0px rgba(0, 0, 0, 0.729);
}

span:hover {
  background-color: #c9c6c6;
}


p {
  font-weight: bold;
  font-size: 30px;
}

.grass {
  background-color: #4CAF50;
}

.poison {
  background-color: #dc02dc;
}

.fire {
  background-color: #FF0000;
}

.flying {
  background-color: #00BFFF;
}

.water {
  background-color: #2196F3;
}

.bug {
  background-color: #6B8E23;
}

.normal {
  background-color: #808080;
}

.electric {
  background-color: #FFFF00;
}

.ground {
  background-color: #d5722c;
}

.fairy {
  background-color: #FF69B4;
}

.fighting {
  background-color: #d84343;
}

.psychic {
  background-color: #fe891c;
}

.rock {
  background-color: #c8c8c8;
}

.ice {
  background-color: #9bdfff;
}

.ghost {
  background-color: #9e029e;
}

.steel {
  background-color: #d1d1d1;
}

.dragon {
  background-color: #a96b00;
}

.dark {
  background-color: #746e64;
}

progress {
  vertical-align: baseline;
  height: 40px;
}

.target {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 35px;
  margin: 25px;
}

.centmo {
  display: flex;
  text-align: center;
  flex-direction: column;
  align-items: center;
  align-content: center;
}



.barra {
  height: 20px;
  background-color: green;
  width: 300px;
  border-radius: 10px;
}


.grid2 {
  display: grid;
  grid-template-columns: 50% 50%;
  justify-items: center;

}

.stats {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
}

.center {
  display: flex;
  flex-direction: column;
  align-items: center;
  background-color: #f0f0f0;
  padding: 10px;
  border-radius: 10px;
  box-shadow: 0px 0px 10px 0px rgba(0, 0, 0, 0.2);
}

.center:hover {
  background-color: rgb(196, 251, 251);
}

.button {
  border-radius: 8px;
  border: 1px solid transparent;
  padding: 0.6em 1.2em;
  font-size: 1em;
  font-weight: 500;
  font-family: inherit;
  background-color: #dff0a6;
  cursor: pointer;
  transition: background-color 0.25s, border-color 0.25s;
  margin: 20px;
}

.button:hover {
  background-color: #c3c6f4;
}

.modal-dialog {
  max-width: 800px;
}

.modal-content {
  border-radius: 15px;
  box-shadow: 0px 0px 10px 0px rgba(0, 0, 0, 0.2);

}


.modal-body {
  padding: 20px;
}

.modti {
  display: grid;
  justify-items: center;
}

.modal-header {
  display: grid;
  justify-content: center;
}



.h5,
h5 {
  font-size: 40px;
  font-family: Verdana, Geneva, Tahoma, sans-serif;
  color: green;

}


h2 {
  text-align: center;
  font-size: 20px;
  font-family: Verdana, Geneva, Tahoma, sans-serif;
  margin-top: -90px
}


h4 {
  display: flex;
  gap: 20px;
  align-items: center;
  font-size: 20px;
  font-family: Verdana, Geneva, Tahoma, sans-serif;
  margin-right: 90px;
}
</style>

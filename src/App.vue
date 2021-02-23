<template>
  <div id="app">
    <SearchBar 
      v-model="search" 
      @chose="onPokemonChose"
      :pokemonList="pokemonList"
    />
  </div>
</template>

<script>
import SearchBar from './components/SearchBar.vue'

export default {
  name: 'App',
  components: {
    SearchBar,
  },
  data() {
    return {
      search: '',
      pokemonList: [],
    }
  },
  methods: {
    onPokemonChose(value) {
      this.pokemonList.push(value)
    },
  },
}
</script>

<style>
#app {
  font-family: GT America, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

@font-face {
  font-family: "GT America";
  src: local("GT America"), url(./assets/fonts/GT-America-Regular.otf) format("truetype");
}

</style>

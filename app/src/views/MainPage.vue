<template>
    <div>

    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import PokemonCard from '@/components/PokemonCard.vue'

const pokemon = ref([])
async function getPokemon() {
  try {
    const response = await fetch(`https://pokeapi.co/api/v2/pokemon?limit=151&offset=0`)
    const data = await response.json()
    pokemon.value = data.results
  } catch (error) {
    console.log(error)
  }
}
onMounted(() => {
  getPokemon()
})
</script>

<style scoped>

</style>
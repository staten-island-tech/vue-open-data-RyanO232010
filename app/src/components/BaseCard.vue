<template>
  <RouterLink :to="pkmnPath">
    <h2>{{ props.data.sex }}</h2>
    <h3>{{ props.data.leading_cause }}</h3>
  </RouterLink>
</template>

<script setup>
import { computed } from 'vue';

const props = defineProps({
    pokemon:{
        type: Object,
        required: true
    },
    id: {
        type:Number,
        required:true,
    }
})
const pkmnPath = computed(()=> {
  return `/pkmn/${props.pokemon.name}`
})
</script>

<style scoped>

div {
  padding: 16px;
}

h1 {
  text-align: center;
  margin-bottom: 20px;
}
.container {
  display: flex;
  width: 80vw;
  margin: 20px auto;
  justify-content: space-around;
  align-items: center;
  flex-wrap: wrap;
}
.card {
  border: 1px solid #ccc;
  padding: 12px;
  margin-bottom: 12px;
  border-radius: 6px;
  width: 300px;
  display: flex;
  flex-direction: column;
  justify-content: space-around;
  align-items: center;
}
</style>

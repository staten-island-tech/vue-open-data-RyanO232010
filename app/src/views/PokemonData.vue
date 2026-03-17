<template>
    <div>
        <h2>{{ pokemon.species.id }}</h2>
    </div>
</template>

<script setup>
import { onMounted,ref,watch } from 'vue';
import { useRoute } from 'vue-router';
const route = useRoute()
const pokemon = ref()
async function getData() {
  try {
    const response = await fetch(`https://data.cityofnewyork.us/resource/jb7j-dtam.json${id}`)
    const data = await response.json()
    data.value = data.results
  } catch (error) {
    console.log(error)
  }
}
watch(
    ()=> route.params.id,
    function(id){
        getData(id)
    }
)
onMounted(function(){
  getData(route.params.id)
})

</script>

<style scoped>

</style>
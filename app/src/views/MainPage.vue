<template>
    <div>
      <button @click="getData()">Click</button>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'


const data = ref([])
async function getData() {
  try {
    const response = await fetch(`https://data.cityofnewyork.us/resource/jb7j-dtam.json`)
    const data = await response.json()
    data.value = data.results
  } catch (error) {
    console.log(error)
  }


if(data.value.){
  console.log(data)
}
}
</script>

<style scoped>

</style>
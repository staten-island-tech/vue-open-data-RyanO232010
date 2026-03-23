<template>
  <div>
    <h1>Precinct Incident Data</h1>
    <BarChart :data="chartData" />
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import BarChart from '../components/BarChart.vue'

const chartData = ref([])

async function getData() {
  try {
    const response = await fetch('https://data.cityofnewyork.us/resource/5ucz-vwe8.json')
    const result = await response.json()

    const precinctCounts = result
      .map(item => item.precinct)
      .filter(v => v !== undefined && !isNaN(Number(v)))
      .reduce((acc, precinct) => {
        acc[precinct] = (acc[precinct] || 0) + 1
        return acc
      }, {})

    chartData.value = Object.entries(precinctCounts).map(([precinct, count]) => ({
      precinct,
      count
    }))

  } catch (error) {
    console.error('Error fetching data:', error)
  }
}

onMounted(() => {
  getData()
})
</script>

<style scoped>
h1 {
  text-align: center;
  margin-bottom: 20px;
}

div {
  padding: 20px;
}
</style>
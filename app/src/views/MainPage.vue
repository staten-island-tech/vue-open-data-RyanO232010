<template>
  <div>
    <button @click="getData()">click</button>
    <Bar
      v-if="chartData.labels.length > 0"
      :data="chartData"
      :options="chartOptions"
      class="h-[30rem]"
    />
  </div>
</template>

<script setup>
import { shallowRef, onMounted } from 'vue'
import { Bar } from 'vue-chartjs'

const chartData = shallowRef({
  labels: [],
  datasets: []
})

const chartOptions = shallowRef({
  responsive: true,
  plugins: {
    legend: { position: 'top' },
    title: {
      display: true,
      text: 'NYC Shooting Incidents by Borough'
    }
  }
})

async function getData() {
  try {
    const response = await fetch('https://data.cityofnewyork.us/resource/5ucz-vwe8.json?$limit=1000')
    const fetchedData = await response.json()

    const boroCounts = fetchedData.reduce((acc, item) => {
      const boro = item.boro || 'UNKNOWN'
      acc[boro] = (acc[boro] || 0) + 1
      return acc
    }, {})

    chartData.value = {
      labels: Object.keys(boroCounts),
      datasets: [
        {
          label: 'Shooting Incidents',
          data: Object.values(boroCounts),
          backgroundColor: 'rgba(75, 192, 192, 0.2)',
          borderColor: 'rgba(75, 192, 192, 1)',
          borderWidth: 1
        }
      ]
    }
  } catch (error) {
    console.error(error)
  }
}

onMounted(() => {
  getData()
})
</script>
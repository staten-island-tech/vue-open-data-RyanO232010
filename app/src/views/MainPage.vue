<template>
  <div>
    <h1>Precinct Incident Data</h1>
    <svg ref="svg"></svg>
    <button @click="change('count')">Update Counts</button>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'
import * as d3 from 'd3'

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

const svg = ref(null)

const drawChart = () => {
  const width = 500;
  const height = Math.min(500, width / 2);
  const outerRadius = height / 2 - 10;
  const innerRadius = outerRadius * 0.75;
  const tau = 2 * Math.PI;
  const color = d3.scaleOrdinal(d3.schemeCategory10);

  const svgEl = d3.select(svg.value)
    .attr("viewBox", [-width / 2, -height / 2, width, height]);

  const arc = d3.arc()
    .innerRadius(innerRadius)
    .outerRadius(outerRadius);

  const pie = d3.pie().sort(null).value(d => d.count);  // Using count as the value for the pie chart
  
  const path = svgEl.datum(chartData.value).selectAll("path")
    .data(pie)
    .join("path")
    .attr("fill", (d, i) => color(i))
    .attr("d", arc)
    .each(function(d) { this._current = d; });

  function change(value) {
    pie.value(d => d[value]); // change the value function dynamically (e.g., switch between count, or other values if present)
    path.data(pie);  // recompute the new angles
    path.transition().duration(750).attrTween("d", arcTween);  // apply the arc tween transition
  }

  function arcTween(a) {
    const i = d3.interpolate(this._current, a);
    this._current = i(0);
    return t => arc(i(t));  // interpolate the arc during the transition
  }

  return Object.assign(svgEl.node(), { change });
}

watch(() => chartData.value, drawChart)
</script>

<style scoped>
h1 {
  text-align: center;
  margin-bottom: 20px;
}

div {
  padding: 20px;
  text-align: center;
}
</style>
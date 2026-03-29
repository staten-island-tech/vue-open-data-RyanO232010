<template>
  <div>
     <RouterLink to="/boroughs"><button>Borough Data</button></RouterLink>
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
      .map((item) => item.precinct)
      .filter((v) => v !== undefined && !isNaN(Number(v)))
      .reduce((acc, precinct) => {
        acc[precinct] = (acc[precinct] || 0) + 1
        return acc
      }, {})
    chartData.value = Object.entries(precinctCounts).map(([precinct, count]) => ({
      precinct,
      count,
    }))
  } catch (error) {
    console.error('Error fetching data:', error)
  }
}

onMounted(() => {
  getData()
})

const svg = ref(null)
let chart

const drawChart = () => {
  const width = 500
  const height = Math.min(500, width / 2)
  const outerRadius = height / 2 - 10
  const innerRadius = outerRadius * 0.75
  const color = d3.scaleOrdinal(d3.schemeCategory10)
  const svgEl = d3.select(svg.value).attr('viewBox', [-width / 2, -height / 2, width, height])

  svgEl.selectAll('*').remove()

  const arc = d3.arc().innerRadius(innerRadius).outerRadius(outerRadius)

  const pie = d3
    .pie()
    .sort(null)
    .value((d) => d.count)

  const slices = svgEl
    .selectAll('path')
    .data(pie(chartData.value), (d) => d.data.precinct)
    .join('path')
    .attr('fill', (d, i) => color(i))
    .each(function (d) {
      this._current = { startAngle: 0, endAngle: 0 }
    })

  slices
    .transition()
    .duration(1000)
    .attrTween('d', function (d) {
      const interpolate = d3.interpolate(this._current, d)
      this._current = d
      return (t) => arc(interpolate(t))
    })

  const labels = svgEl
    .selectAll('text')
    .data(pie(chartData.value), (d) => d.data.precinct)
    .join('text')
    .attr('text-anchor', 'middle')
    .attr('alignment-baseline', 'middle')
    .style('font-size', '12px')
    .style('fill', '#fff')

  labels
    .transition()
    .duration(1000)
    .attr('transform', (d) => {
      const centroid = arc.centroid(d)
      return `translate(${centroid[0]}, ${centroid[1]})`
    })

  const tooltip = d3
    .select('body')
    .append('div')
    .attr('class', 'tooltip')
    .style('position', 'absolute')
    .style('padding', '6px')
    .style('background', 'rgba(0,0,0,0.7)')
    .style('color', '#fff')
    .style('border-radius', '4px')
    .style('pointer-events', 'none')
    .style('opacity', 0)

  slices
    .on('mouseover', (event, d) => {
      tooltip.transition().duration(200).style('opacity', 1)
      tooltip
        .html(`Precinct: ${d.data.precinct}<br>Count: ${d.data.count}`)
        .style('left', event.pageX + 10 + 'px')
        .style('top', event.pageY - 28 + 'px')
    })
    .on('mousemove', (event) => {
      tooltip.style('left', event.pageX + 10 + 'px').style('top', event.pageY - 28 + 'px')
    })
    .on('mouseout', () => {
      tooltip.transition().duration(200).style('opacity', 0)
    })

  function change(value) {
    pie.value((d) => d[value])
    const updated = pie(chartData.value)
    slices
      .data(updated)
      .transition()
      .duration(750)
      .attrTween('d', function (d) {
        const interpolate = d3.interpolate(this._current, d)
        this._current = d
        return (t) => arc(interpolate(t))
      })

    labels
      .data(updated)
      .transition()
      .duration(750)
      .attr('transform', (d) => {
        const centroid = arc.centroid(d)
        return `translate(${centroid[0]}, ${centroid[1]})`
      })
  }

  chart = { change }
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

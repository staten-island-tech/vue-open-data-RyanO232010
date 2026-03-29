<template>
  <div>
    <RouterLink to="/"><button>District Data</button></RouterLink>
  </div>

  <div>
    <label>Borough of Interest:</label>

    <select v-model="selectedBorough">
      <option value="">--Please choose a Borough</option>
      <option value="BROOKLYN">Brooklyn</option>
      <option value="STATEN ISLAND">Staten Island</option>
      <option value="BRONX">Bronx</option>
      <option value="QUEENS">Queens</option>
      <option value="MANHATTAN">Manhattan</option>
    </select>

    <button @click="applyFilter">Confirm</button>

    <h1>Location Classification Race</h1>

    <svg ref="svg" width="700" height="400"></svg>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import * as d3 from 'd3'

const selectedBorough = ref('')
const svg = ref(null)

async function applyFilter() {
  const response = await fetch('https://data.cityofnewyork.us/resource/5ucz-vwe8.json?$limit=10000')
  let raw = await response.json()

  if (selectedBorough.value) {
    raw = raw.filter(d =>
      (d.boro && d.boro.toUpperCase() === selectedBorough.value) ||
      (d.boro_nm && d.boro_nm.toUpperCase() === selectedBorough.value)
    )
  }

  const svgEl = d3.select(svg.value)
  svgEl.selectAll('*').remove()

  if (raw.length === 0) {
    svgEl.append('text')
      .attr('x', 350)
      .attr('y', 200)
      .attr('text-anchor', 'middle')
      .attr('font-size', 24)
      .text('NO DATA AVAILABLE')
    return
  }

  const allCounts = d3.rollup(
    raw,
    v => v.length,
    d => d.loc_classfctn_desc || 'UNKNOWN'
  )

  const topCategories = Array.from(allCounts, ([name, value]) => ({
    name,
    value: +value
  }))
    .sort((a, b) => b.value - a.value)
    .slice(0, 10)
    .map(d => d.name)

  const chunkSize = Math.max(20, Math.floor(raw.length / 50))
  let frames = []

  for (let i = chunkSize; i <= raw.length; i += chunkSize) {
    const slice = raw.slice(0, i)

    const counts = d3.rollup(
      slice,
      v => v.length,
      d => d.loc_classfctn_desc || 'UNKNOWN'
    )

    let dataset = topCategories.map(name => ({
      name,
      value: +(counts.get(name) || 0)
    }))
      .sort((a, b) => b.value - a.value)

    frames.push(dataset)
  }

  runRace(frames)
}

async function runRace(frames) {
  const width = 700
  const height = 400
  const margin = { top: 20, right: 20, bottom: 40, left: 150 }

  const svgEl = d3.select(svg.value)
    .attr('viewBox', [0, 0, width, height])

  svgEl.selectAll('*').remove()

  for (const data of frames) {
    const maxValue = d3.max(data, d => d.value)

    const x = d3.scaleLinear()
      .domain([0, maxValue * 1.15])
      .range([0, width - margin.left - margin.right])

    const y = d3.scaleBand()
      .domain(data.map(d => d.name))
      .range([margin.top, height - margin.bottom])
      .padding(0.1)

    const t = svgEl.transition().duration(700).ease(d3.easeLinear)

    const bars = svgEl.selectAll('rect')
      .data(data, d => d.name)

    bars.join(
      enter => enter.append('rect')
        .attr('x', margin.left)
        .attr('y', d => y(d.name))
        .attr('height', y.bandwidth())
        .attr('width', 0)
        .attr('fill', 'black')
        .call(e => e.transition(t).attr('width', d => x(d.value))),
      update => update
        .call(u => u.transition(t)
          .attr('y', d => y(d.name))
          .attr('width', d => x(d.value))
        ),
      exit => exit.remove()
    )

    const labels = svgEl.selectAll('.label')
      .data(data, d => d.name)

    labels.join(
      enter => enter.append('text')
        .attr('class', 'label')
        .attr('x', margin.left - 10)
        .attr('y', d => y(d.name) + y.bandwidth() / 2)
        .attr('dy', '0.35em')
        .attr('text-anchor', 'end')
        .text(d => d.name),
      update => update
        .call(u => u.transition(t)
          .attr('y', d => y(d.name) + y.bandwidth() / 2)
        ),
      exit => exit.remove()
    )

    const values = svgEl.selectAll('.value')
      .data(data, d => d.name)

    values.join(
      enter => enter.append('text')
        .attr('class', 'value')
        .attr('x', d => Math.min(
          margin.left + x(d.value) + 10,
          width - margin.right - 40
        ))
        .attr('y', d => y(d.name) + y.bandwidth() / 2)
        .attr('dy', '0.35em')
        .text(d => d.value),
      update => update
        .call(u => u.transition(t)
          .attr('x', d => Math.min(
            margin.left + x(d.value) + 10,
            width - margin.right - 40
          ))
          .attr('y', d => y(d.name) + y.bandwidth() / 2)
          .tween('text', function(d) {
            const i = d3.interpolateNumber(+this.textContent || 0, d.value)
            return function(t) {
              this.textContent = Math.round(i(t))
            }
          })
        ),
      exit => exit.remove()
    )

    await t.end()
  }
}
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
<template>
  <svg ref="svg"></svg>
</template>

<script setup>
import { onMounted, ref, watch } from 'vue'
import * as d3 from 'd3'

const props = defineProps({
  data: {
    type: Array,
    required: true
  },
  width: {
    type: Number,
    default: 400
  },
  height: {
    type: Number,
    default: 300
  }
})

const svg = ref(null)

const drawChart = () => {
  const svgEl = d3.select(svg.value)
  svgEl.selectAll('*').remove()

  svgEl.attr('width', props.width).attr('height', props.height)

  const precincts = props.data.map(d => d.precinct)

  const x = d3
    .scaleBand()
    .domain(precincts)
    .range([0, props.width])
    .padding(0.1)

  const y = d3.scaleLinear()
    .domain([0, d3.max(props.data, d => d.count) || 0])
    .nice()
    .range([props.height, 0])

  svgEl
    .selectAll('rect')
    .data(props.data)
    .join('rect')
    .attr('x', (d) => x(d.precinct))
    .attr('y', (d) => y(d.count))
    .attr('width', x.bandwidth())
    .attr('height', (d) => props.height - y(d.count))
    .attr('fill', 'steelblue')

  svgEl
    .selectAll('text.precinct')
    .data(props.data)
    .join('text')
    .attr('x', (d) => x(d.precinct) + x.bandwidth() / 2)
    .attr('y', (d) => y(d.count) - 5)
    .attr('text-anchor', 'middle')
    .attr('fill', 'black')
    .style('font-size', '20px')

  const yAxis = d3.axisLeft(y)
  svgEl
    .append('g')
    .call(yAxis)
    .attr('transform', `translate(0, 0)`)

  const xAxis = d3.axisBottom(x)
  svgEl
    .append('g')
    .call(xAxis)
    .attr('transform', `translate(0, ${props.height - 20})`)
}

onMounted(drawChart)

watch(() => props.data, drawChart)
</script>
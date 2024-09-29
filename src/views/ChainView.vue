<template>
  <div class="min-h-screen bg-gray-100">
    <div class="max-w-7xl mx-auto py-6 sm:px-6 lg:px-8">
      <div class="px-4 py-6 sm:px-0">
        <div class="bg-white overflow-hidden shadow-xl sm:rounded-lg">
          <div ref="graphContainer" class="h-[600px]"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import * as d3 from 'd3';
import axios from 'axios';

const graphContainer = ref(null);
const rawData = ref([]);

const nodes = ref([]);
const links = ref([]);

const fetchData = async () => {
  try {
    const response = await axios.get('/api/your-endpoint'); // Replace with your actual API endpoint
    rawData.value = response.data;

    rawData.value.forEach(item => {
      nodes.value.push({ id: item.caller_name, color: '#1f77b4' });
      nodes.value.push({ id: item.table_name, color: '#ff7f0e' });
      links.value.push({ source: item.caller_name, target: item.table_name });
    });

    // Remove duplicate nodes
    nodes.value = Array.from(new Set(nodes.value.map(node => JSON.stringify(node)))).map(str => JSON.parse(str));

    initGraph();
  } catch (error) {
    console.error('Error fetching data:', error);
  }
};

const initGraph = () => {
  const svg = d3.select(graphContainer.value).append('svg')
    .attr('width', '100%')
    .attr('height', '100%');

  const g = svg.append('g');

  const link = g.selectAll('.link')
    .data(links.value)
    .enter().append('line')
    .attr('class', 'link')
    .style('stroke', '#999')
    .style('stroke-opacity', 0.6)
    .style('stroke-width', 1);

  const node = g.selectAll('.node')
    .data(nodes.value)
    .enter().append('g')
    .attr('class', 'node');

  node.append('circle')
    .attr('r', 5)
    .style('fill', d => d.color);

  node.append('text')
    .attr('dx', d => d.id === rawData.value[0].caller_name ? -12 : 12) // Adjust text position
    .attr('dy', '.35em')
    .attr('text-anchor', d => d.id === rawData.value[0].caller_name ? 'end' : 'start') // Align text to the end for caller_name
    .text(d => d.id)
    .style('font-size', '10px');

  d3.forceSimulation(nodes.value)
    .force('link', d3.forceLink(links.value).id(d => d.id).distance(100))
    .force('charge', d3.forceManyBody().strength(-50))
    .force('center', d3.forceCenter(graphContainer.value.clientWidth / 2, graphContainer.value.clientHeight / 2))
    .on('tick', () => {
      link
        .attr('x1', d => d.source.x)
        .attr('y1', d => d.source.y)
        .attr('x2', d => d.target.x)
        .attr('y2', d => d.target.y);

      node
        .attr('transform', d => `translate(${d.x},${d.y})`);
    });
};

onMounted(() => {
  fetchData();
});
</script>

<style scoped>
.link {
  stroke: #999;
  stroke-opacity: 0.6;
}

.node circle {
  stroke: #fff;
  stroke-width: 1.5px;
}

.node text {
  pointer-events: none;
  font: 10px sans-serif;
}
</style>
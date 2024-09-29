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
import { ref, onMounted, watch } from 'vue';
import * as d3 from 'd3';

const searchQuery = ref('');
const graphContainer = ref(null);

const nodes = ref([
  { id: 'test', color: '#ff0000' },
  { id: 'aom12345', color: '#1f77b4' },
  { id: 'dujc', color: '#1f77b4' },
  { id: 'test2', color: '#1f77b4' },
  { id: 'csscss', color: '#1f77b4' },
  { id: 'test555', color: '#1f77b4' },
  { id: 'hhkk', color: '#1f77b4' },
  { id: 'test00222', color: '#1f77b4' },
  { id: 'test1111', color: '#1f77b4' },
  { id: 'dfgxdfgxfg', color: '#1f77b4' },
  { id: 'fwarfarf', color: '#1f77b4' },
  { id: 'qqqq', color: '#1f77b4' },
  { id: 'wwww', color: '#1f77b4' },
  { id: 'test003', color: '#1f77b4' },
  { id: '12345', color: '#ff0000' },
  { id: 'uupp', color: '#ff0000' },
]);

const links = ref(
  nodes.value.filter(node => node.id !== 'test').map(node => ({
    source: 'test',
    target: node.id,
  }))
);

let simulation;

const initGraph = () => {
  const svg = d3.select(graphContainer.value).append('svg')
    .attr('width', '100%')
    .attr('height', '100%');

  const g = svg.append('g');

  const zoom = d3.zoom()
    .scaleExtent([0.1, 4])
    .on('zoom', (event) => {
      g.attr('transform', event.transform);
    });

  svg.call(zoom);

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
    .attr('dx', 12)
    .attr('dy', '.35em')
    .text(d => d.id)
    .style('font-size', '10px');

  simulation = d3.forceSimulation(nodes.value)
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
  initGraph();
});

watch(searchQuery, (newValue) => {
  // Update graph based on search query
  console.log('Search query changed:', newValue);
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

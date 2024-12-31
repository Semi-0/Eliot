<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue';
import * as d3 from 'd3';

const props = defineProps({
  nodes: {
    type: Array,
    default: () => []
  },
  links: {
    type: Array,
    default: () => []
  }
});

const svgRef = ref(null);
let simulation = null;
let svg = null;
let g = null;
const width = 800;
const height = 600;

// Move the simulation creation and update logic to a separate function
const updateSimulation = () => {
  if (!svg) return;
  
  // Clear previous simulation
  if (simulation) {
    simulation.stop();
  }
  
  // Clear previous nodes and links
  g.selectAll('.nodes').remove();
  g.selectAll('.links').remove();

  // Create the force simulation with optimized settings
  simulation = d3.forceSimulation(props.nodes)
    .force('link', d3.forceLink(props.links)
      .id(d => d.id)
      .distance(150)
      .strength(1))
    .force('charge', d3.forceManyBody()
      .strength(-1000)
      .distanceMin(100)
      .distanceMax(500))
    .force('center', d3.forceCenter(width / 2, height / 2))
    .force('collision', d3.forceCollide().radius(50))
    .force('x', d3.forceX(width / 2).strength(0.1))
    .force('y', d3.forceY(height / 2).strength(0.1));

  // Draw the links first (so they appear under nodes)
  const links = g.append('g')
    .attr('class', 'links')
    .selectAll('line')
    .data(props.links)
    .join('line')
    .attr('class', d => `link link-${getLinkType(d)}`)
    .attr('stroke', '#999')
    .attr('marker-end', d => `url(#arrow-${getLinkType(d)})`);

  // Draw the nodes
  const nodes = g.append('g')
    .attr('class', 'nodes')
    .selectAll('g')
    .data(props.nodes)
    .join('g')
    .attr('class', 'node')
    .call(d3.drag()
      .on('start', dragstarted)
      .on('drag', dragged)
      .on('end', dragended));

  // First add rectangles
  nodes.append('rect')
    .attr('rx', 10) // rounded corners
    .attr('ry', 10) // rounded corners
    .attr('fill', 'white')
    .attr('stroke', 'black')
    .attr('stroke-width', '2px');

  // Then add labels
  nodes.append('text')
    .text(d => d.name)
    .attr('text-anchor', 'middle')
    .attr('dy', '.35em')
    .attr('fill', '#000')
    .attr('font-weight', 'bold');

  // Size the rectangles based on text size
  nodes.each(function() {
    const text = d3.select(this).select('text');
    const bbox = text.node().getBBox();
    const padding = { x: 20, y: 10 }; // Horizontal and vertical padding
    
    d3.select(this).select('rect')
      .attr('width', bbox.width + (padding.x * 2))
      .attr('height', bbox.height + (padding.y * 2))
      .attr('x', -(bbox.width + padding.x * 2) / 2)
      .attr('y', -(bbox.height + padding.y * 2) / 2);
  });

  // Update positions on each tick
  simulation.on('tick', () => {
    links
      .attr('x1', d => {
        const dx = d.target.x - d.source.x;
        const dy = d.target.y - d.source.y;
        const angle = Math.atan2(dy, dx);
        return d.source.x + 20 * Math.cos(angle); // Start from node edge
      })
      .attr('y1', d => {
        const dx = d.target.x - d.source.x;
        const dy = d.target.y - d.source.y;
        const angle = Math.atan2(dy, dx);
        return d.source.y + 20 * Math.sin(angle); // Start from node edge
      })
      .attr('x2', d => {
        const dx = d.target.x - d.source.x;
        const dy = d.target.y - d.source.y;
        const angle = Math.atan2(dy, dx);
        return d.target.x - 25 * Math.cos(angle); // Stop before arrow (adjusted distance)
      })
      .attr('y2', d => {
        const dx = d.target.x - d.source.x;
        const dy = d.target.y - d.source.y;
        const angle = Math.atan2(dy, dx);
        return d.target.y - 25 * Math.sin(angle); // Stop before arrow (adjusted distance)
      });

    nodes.attr('transform', d => `translate(${d.x},${d.y})`);
  });

  function dragstarted(event) {
    if (!event.active) simulation.alphaTarget(0.3).restart();
    event.subject.fx = event.subject.x;
    event.subject.fy = event.subject.y;
  }

  function dragged(event) {
    event.subject.fx = event.x;
    event.subject.fy = event.y;
  }

  function dragended(event) {
    if (!event.active) simulation.alphaTarget(0);
    event.subject.fx = null;
    event.subject.fy = null;
  }
};

onMounted(() => {
  svg = d3.select(svgRef.value)
    .attr('width', width)
    .attr('height', height)
    .attr('viewBox', [0, 0, width, height]);

  // Create different arrow markers for different connection types
  const defs = svg.append('defs');
  
  // Single arrow marker (for mono-directional)
  defs.append('marker')
    .attr('id', 'arrow-mono')
    .attr('viewBox', '-5 -5 10 10')
    .attr('refX', 15)
    .attr('refY', 0)
    .attr('markerWidth', 6)
    .attr('markerHeight', 6)
    .attr('orient', 'auto')
    .append('path')
    .attr('d', 'M -4 -4 L 0 0 L -4 4')
    .attr('fill', 'none')
    .attr('stroke', '#999')
    .attr('stroke-width', '1px');

  // Double arrow marker (for bi-directional)
  defs.append('marker')
    .attr('id', 'arrow-bi')
    .attr('viewBox', '-5 -5 10 10')
    .attr('refX', 15)
    .attr('refY', 0)
    .attr('markerWidth', 6)
    .attr('markerHeight', 6)
    .attr('orient', 'auto')
    .append('path')
    .attr('d', 'M -4 -4 L 0 0 L -4 4 M -2 -4 L 2 0 L -2 4')
    .attr('fill', 'none')
    .attr('stroke', '#999')
    .attr('stroke-width', '1px');

  // Multi-directional marker (for complex connections)
  defs.append('marker')
    .attr('id', 'arrow-multi')
    .attr('viewBox', '-5 -5 10 10')
    .attr('refX', 15)
    .attr('refY', 0)
    .attr('markerWidth', 6)
    .attr('markerHeight', 6)
    .attr('orient', 'auto')
    .append('path')
    .attr('d', 'M -4 -4 L 0 0 L -4 4 L -2 0 Z')
    .attr('fill', '#999')
    .attr('stroke', '#999')
    .attr('stroke-width', '1px');

  g = svg.append('g');

  // Zoom behavior
  const zoom = d3.zoom()
    .scaleExtent([0.1, 4])
    .on('zoom', (event) => {
      g.attr('transform', event.transform);
    });

  svg.call(zoom);

  // Initial simulation setup
  updateSimulation();
});

// Add watch effect for nodes and links
watch(
  () => [props.nodes, props.links],
  () => {
    updateSimulation();
  },
  { deep: true }
);

onUnmounted(() => {
  if (simulation) {
    simulation.stop();
  }
});
</script>

<template>
  <svg ref="svgRef"></svg>
</template>

<style scoped>
.link {
  stroke: #999;
  stroke-width: 2px;
  fill: none;
  pointer-events: none;
}

.link-mono {
  stroke-dasharray: none;
}

.link-bi {
  stroke-dasharray: 5,5;
}

.link-multi {
  stroke-width: 3px;
}

.node rect {
  stroke: black;
  stroke-width: 2px;
  cursor: pointer;
  fill: white;
}

.node text {
  font-size: 12px;
  pointer-events: none;
  fill: #000;
  font-weight: bold;
}

.nodes {
  z-index: 2;
}

.links {
  z-index: 1;
}
</style> 
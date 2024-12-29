<script setup>
import ForceGraph from './components/ForceGraph.vue';

// Sample data for the graph
const nodes = [
  { id: 1, name: 'Node A', color: '#4CAF50' },
  { id: 2, name: 'Node B', color: '#2196F3' },
  { id: 3, name: 'Node C', color: '#FFC107' },
  { id: 4, name: 'Node D', color: '#9C27B0' },
  { id: 5, name: 'Node E', color: '#F44336' }
];

const links = [
  { source: 1, target: 2 },           // Mono-directional A -> B
  { source: 2, target: 1 },           // Bi-directional B -> A (with above)
  { source: 2, target: 3 },           // Mono-directional B -> C
  { source: 3, target: 4 },           // Start of multi-directional
  { source: 4, target: 5 },           // Multi-directional
  { source: 5, target: 3 },           // Multi-directional (closes the loop)
  { source: 1, target: 5 }            // Additional connection
];
</script>

<template>
  <div class="container">
    <h1>Force-Directed Graph</h1>
    <div class="legend">
      <div class="legend-item">
        <span class="line mono"></span>
        <span>Mono-directional</span>
      </div>
      <div class="legend-item">
        <span class="line bi"></span>
        <span>Bi-directional</span>
      </div>
      <div class="legend-item">
        <span class="line multi"></span>
        <span>Multi-directional</span>
      </div>
    </div>
    <ForceGraph :nodes="nodes" :links="links" />
  </div>
</template>

<style>
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

h1 {
  text-align: center;
  color: #333;
  margin-bottom: 20px;
}

.legend {
  display: flex;
  justify-content: center;
  gap: 20px;
  margin-bottom: 20px;
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 8px;
}

.line {
  display: inline-block;
  width: 30px;
  height: 2px;
  background: #000000;
}

.line.bi {
  background: linear-gradient(to right, #666 50%, transparent 50%);
  background-size: 10px 100%;
  background-repeat: repeat-x;
}

.line.multi {
  height: 3px;
}

body {
  margin: 0;
  padding: 0;
  font-family: Arial, sans-serif;
  background-color: #f5f5f5;
}
</style>

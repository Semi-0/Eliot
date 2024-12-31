<script setup>
import { ref, onMounted, watch } from 'vue';
import ForceGraph from './components/ForceGraph.vue';
import SchemeEditor from './components/SchemeEditor.vue';

const nodes = ref([
  { id: 1, name: "Node 1" },
  { id: 2, name: "Node 2" },
  { id: 3, name: "Node 3" }
]);

const links = ref([
  { source: 1, target: 2 },
  { source: 2, target: 3 },
  { source: 3, target: 1 }
]);

const showEditor = ref(true);
const schemeCode = ref('');



// Add a watch if you need to respond to code changes
watch(schemeCode, (newCode) => {
  
  // Add your code processing logic here
});
</script>

<template>
  <div class="app-container" style="background-color: #f0f0f0;">
    <div class="editor-overlay">
      <SchemeEditor
        v-model="schemeCode"
        v-model:visible="showEditor"
      />
    </div>
    <div class="graph-container">
      <ForceGraph :nodes="nodes" :links="links" />
    </div>
  </div>
</template>

<style>
/* Reset CSS */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.app-container {
  /* width: 100%;
  height: 100vh; */
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  position: fixed;
  overflow: hidden;
}

.editor-overlay {
  position: absolute;
  top: 20px;
  left: 20px;
  z-index: 2;
  width: 40%;
}

</style>

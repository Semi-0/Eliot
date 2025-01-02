<script setup>
import { ref, onMounted, watch, shallowRef } from 'vue';
import ForceGraph from './components/ForceGraph.vue';
import SchemeEditor from './components/SchemeEditor.vue';
import { main, env, clear_env } from "seminteresting/Main"
import CodeMirrorEditor from './components/CodeMirrorEditor.vue';
import { define_generic_matcher } from 'seminteresting/tools/ExpressionHandler';
import { evaluate } from 'seminteresting/Evaluator';
import { nextTick } from 'vue';
import { create_default_scoped_primtive_func, make_primitive_package } from 'seminteresting/definition/PackageSystem';
import { getCurrentInstance } from 'vue';
import { ssrDynamicImportKey } from 'vite/module-runner';
import { watchEffect } from 'vue';
import { DefaultEnvironment } from 'seminteresting/definition/Environment';
const nodes = shallowRef([
  // { id: 1, name: "Node 1" },
  // { id: 2, name: "Node 2" },
  // { id: 3, name: "Node 3" }
]);

const links = shallowRef([
  // { source: 1, target: 2 },
  // { source: 2, target: 3 },
  // { source: 3, target: 1 }
]);

const showEditor = ref(true);
const schemeCode = ref('');
const execution_result = ref('');

function reference_store() {
  let id = 0
  return () => {
    id++
    return id
  }
}

const get_new_id = reference_store()

function add_node(id, name) {
  nodes.value = [...nodes.value, { id: id, name: name }];
  console.log("add_node", nodes.value)
  // nextTick(() => {
  //   console.log("Node added, DOM updated");
  // });
  return id;
}

function add_link(source, target) {
  console.log("add_link", source, target)
  links.value = [...links.value, { source: source, target: target }];
}


watchEffect(() => {
  console.log("watch effect", nodes.value)
})

function connect(source_id, target_id) {
  add_link(source_id, target_id);
}

function make_graph_wrapper_package() {
  const procedures = {
    "<>":  (name) => {
      console.log("try add", name)
      const id = add_node(get_new_id(), name)
      return id
    },
    "->":  (source_id, target_id) => {
      console.log("try connect", source_id, target_id)
       connect(source_id, target_id)
    }
  }

  const dict = Object.fromEntries(Object
                            .entries(procedures)
                            .map(([key, value]) => 
                                [key, create_default_scoped_primtive_func(value)]))

  return {
    name: "graph_wrapper",
    dict: dict,
    ref: 0,
    has: (key) => dict[key] !== undefined,
    not_has: (key) => dict[key] === undefined,
    summarize: () => summarize_dict(dict, summarize_scoped_value),
    copy: () => make_graph_wrapper_package(),
  }
}

function clear_graph(){
  nodes.value = []
  links.value = []
}




// Add a watch if you need to respond to code changes
watch(schemeCode, (newCode) => {
   const full_text = newCode

  try{
    clear_graph()
    setTimeout(() => {
      clear_env()
      env.load(make_primitive_package())
      env.load(make_graph_wrapper_package())
      const result = main(full_text)
      console.log("result", result)
      execution_result.value = result.value
    }, 10)
    
  }
  catch(e){
    console.error(e)
  }
  
   console.log(execution_result.value)
  // Add your code processing logic here
});


</script>

<template>
  <div class="app-container" style="background-color: #f0f0f0;">

    <div class="editor-overlay">
      <!-- <SchemeEditor
        v-model="schemeCode"
        v-model:visible="showEditor"
      /> -->
      <CodeMirrorEditor
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
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  gap: 20px;
  padding: 20px;
  overflow: visible;

}

.editor-overlay {
  position: relative;
  width: 40%;
  z-index: 2;
}


.graph-container {
  position: relative;
  flex: 1;
  min-height: 100%;
  z-index: 1;
  overflow: visible;

}

.append-button {
  margin-top: 10px;
  padding: 8px 16px;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.append-button:hover {
  background-color: #45a049;
}
</style>

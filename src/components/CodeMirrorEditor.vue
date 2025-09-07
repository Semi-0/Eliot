<script setup lang="ts">
import { defineProps, reactive, shallowRef, computed, watch, onMounted } from 'vue'
import { EditorView, ViewUpdate } from '@codemirror/view'
import { redo, undo } from '@codemirror/commands'
import { Codemirror } from 'vue-codemirror'
import { javascript } from '@codemirror/lang-javascript'
import { oneDark } from '@codemirror/theme-one-dark'

const props = defineProps({
  modelValue: {
    type: String,
    default: ''
  },
  visible: {
    type: Boolean,
    default: true
  }
});

const emit = defineEmits(['update:modelValue', 'update:visible']);

const code = shallowRef(props.modelValue)
const preview = shallowRef(false)
const cmView = shallowRef<EditorView>()

const config = reactive({
  disabled: false,
  indentWithTab: true,
  tabSize: 2,
  autofocus: true,
  height: '400px'
})

const state = reactive({
  lines: 0,
  cursor: 0,
  selected: 0,
  length: 0
})

// Configure extensions with JavaScript-like syntax highlighting for Scheme
const extensions = [
  javascript({
    jsx: false,
    typescript: false
  }),
  oneDark
]

const handleReady = ({ view }: { view: EditorView }) => {
  cmView.value = view
}

const handleStateUpdate = (viewUpdate: ViewUpdate) => {
  const ranges = viewUpdate.state.selection.ranges
  state.selected = ranges.reduce((plus, range) => plus + range.to - range.from, 0)
  state.cursor = ranges[0].anchor
  state.length = viewUpdate.state.doc.length
  state.lines = viewUpdate.state.doc.lines
  emit('update:modelValue', viewUpdate.state.doc.toString())
}

const handleUndo = () => {
  if (cmView.value) {
    undo({ state: cmView.value.state, dispatch: cmView.value.dispatch })
  }
}

const handleRedo = () => {
  if (cmView.value) {
    redo({ state: cmView.value.state, dispatch: cmView.value.dispatch })
  }
}

const togglePreview = () => {
  preview.value = !preview.value
}

watch(() => props.modelValue, (newValue) => {
  if (code.value !== newValue) {
    code.value = newValue
  }
})
</script>

<template>
  <div class="editor">
    <div class="main">
      <Codemirror
        v-model="code"
        :style="{
          width: preview ? '50%' : '100%',
          height: config.height,
          backgroundColor: 'transparent',
          color: '#e6e6e6'
        }"
        placeholder=""
        :extensions="extensions"
        :autofocus="config.autofocus"
        :disabled="config.disabled"
        :indent-with-tab="config.indentWithTab"
        :tab-size="config.tabSize"
        @update="handleStateUpdate"
        @ready="handleReady"
      />
      <pre v-if="preview" class="code" :style="{ height: config.height }">{{ code }}</pre>
    </div>

  </div>
</template>

<style scoped>
.editor {
  width: 100%;
  height: 100%;
}

.main {
  display: flex;
  width: 100%;
}

.code {
  width: 50%;
  margin: 0;
  padding: 0.4em;
  overflow: auto;
  border-left: 1px solid #ddd;
  font-family: monospace;
}

.divider {
  height: 1px;
  background-color: #ddd;
}

.footer {
  height: 3rem;
  padding: 0 1em;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 90%;
}

.buttons .item {
  margin-right: 1em;
  padding: 4px 8px;
  background-color: transparent;
  border: 1px dashed #ddd;
  font-size: 0.9em;
  color: #666;
  cursor: pointer;
}

.buttons .item:hover {
  color: #333;
  border-color: #333;
}

.infos .item {
  margin-left: 2em;
  display: inline-block;
  font-feature-settings: 'tnum';
}

/* Ensure the CodeMirror editor itself is transparent */
:deep(.cm-editor) {
  background-color: transparent !important;
  border: none !important;
  box-shadow: none !important;
  outline: none !important;
}

/* Transparent scroll area */
:deep(.cm-scroller) {
  background-color: transparent !important;
  border: none !important;
}

/* Transparent gutters (line numbers) */
:deep(.cm-gutters) {
  background-color: transparent !important;
  border: none !important;
}

/* Remove any theme-provided borders/outlines */
:deep(.cm-gutters .cm-gutter) { border: none !important; box-shadow: none !important; }
:deep(.cm-editor.cm-focused) { outline: none !important; }
:deep(.cm-content) { border: none !important; box-shadow: none !important; }
:deep(.cm-panels) { border: none !important; box-shadow: none !important; }
:deep(.vue-codemirror) { border: none !important; box-shadow: none !important; background: transparent !important; }
</style>
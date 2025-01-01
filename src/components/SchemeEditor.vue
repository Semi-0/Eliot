<script setup>
import { onMounted } from 'vue';
import { PrismEditor } from "vue-prism-editor";
import { highlight, languages } from 'prismjs';
import 'prismjs';
import 'prismjs/themes/prism.css';
import 'prismjs/components/prism-scheme';

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

const highlighter = (code) => {
  if (code) {
    return highlight(code, languages.scheme, 'scheme');
  }
  return '';
};

const updateCode = (code) => {
  emit('update:modelValue', code);
};
</script>

<template>
  <div class="scheme-editor-container">
    <div v-if="visible" class="editor-container">
      <PrismEditor
        :modelValue="modelValue"
        @update:modelValue="updateCode"
        :highlight="highlighter"
        :line-numbers="true"
        class="prism-editor"
      />
    </div>
  </div>
</template>

<style>
.scheme-editor-container {
  width: 100%;
  height: 100%;
}

.editor-container {
  height: 100%;
}

.prism-editor {
  /* Basic editor styling */
  background: #f5f5f5 !important;
  font-family: 'Fira code', 'Fira Mono', Consolas, Menlo, Courier, monospace;
  font-size: 14px;
  line-height: 1.5;
  padding: 5px;
  height: 100%;
}

.prism-editor__textarea {
  /* Ensure proper text visibility */
  color: #000 !important;
  background: transparent !important;
}

.prism-editor__editor {
  /* Ensure proper content layout */
  white-space: pre;
  overflow: auto;
  padding: 0.5em;
}

.prism-editor__line-numbers {
  /* Fix line numbers positioning */
  position: absolute;
  left: 0;
  top: 0;
  padding: 0.5em;
  background: #f0f0f0;
  border-right: 1px solid #ddd;
}
</style> 
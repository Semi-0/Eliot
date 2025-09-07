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
        class="prism-editor"
        :modelValue="modelValue"
        @update:modelValue="updateCode"
        :highlight="highlighter"
        :line-numbers="true"
  
      />
    </div>
  </div>
</template>

<style>
/* .scheme-editor-container {
  width: 100%;
  height: 100%;
} */
/* 
.editor-container {
  height: 100%;
} */

/* .prism-editor {
  background: #f5f5f5 !important;
  font-family: 'Fira code', 'Fira Mono', Consolas, Menlo, Courier, monospace;
  font-size: 14px;
  line-height: 1.5;
  padding: 5px;

} */

/* .prism-editor__textarea {

  color: #000 !important;
  background: transparent !important;
}

.prism-editor__editor {
 
  white-space: pre;
  overflow: auto;
  padding: 0.5em;
} */

.prism-editor__line-numbers {
  /* Fix line numbers positioning */
  position: absolute;
  left: 0;
  top: 0;
  padding: 0.5em;
  background: transparent;
  border-right: 1px solid #ddd;
}

/* Make Prism editor background transparent when used */
.prism-editor,
.prism-editor__editor,
.prism-editor__textarea {
  background: transparent !important;
  border: none !important;
  box-shadow: none !important;
  outline: none !important;
}

/* Remove any residual border from container */
.editor-container, .scheme-editor-container { border: none !important; box-shadow: none !important; }
</style> 
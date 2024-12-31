<script setup>
import { onMounted } from 'vue';
import { PrismEditor } from  "vue-prism-editor"
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
  if (code ) {
    try{
      const highlighted = Prism.highlight(code, Prism.languages.scheme,  "scheme") 
      console.log("highlighted", highlighted)
      return highlighted
    }
    catch(e){
      console.error(e)
    }
  }
  return ""
};

const toggleEditor = () => {
  console.log('Toggle clicked, current visible:', props.visible);
  emit('update:visible', !props.visible);
};

const updateCode = (newCode) => {

  const code = newCode.target.value
  emit('update:modelValue', code);
};
</script>

<template>
  <div class="scheme-editor-container">
    <div v-if="visible" class="editor-container">
      <PrismEditor
        :modelValue="modelValue"
        @input="updateCode"
        :highlight="highlighter"
        line-numbers
        style="height: 800px; width: 100%; background: transparent; color: black;"
        placeholder="Enter Scheme code here..."
      />
    </div>
  </div>
</template>

<style>
.scheme-editor-container {
  border-radius: 8px;
  padding: 10px;
}

/* Ensure text is visible against any background */
.prism-editor__textarea {
  color: black !important;
  background: transparent !important;
  text-shadow: 
    -1px -1px 0 white,
    1px -1px 0 white,
    -1px 1px 0 white,
    1px 1px 0 white;  /* This creates a white outline around text */
}

.prism-editor__container {
  background: transparent !important;
}

/* Make line numbers more visible */
.prism-editor__line-numbers {
  background: rgba(255, 255, 255, 0.5) !important;
}
</style> 
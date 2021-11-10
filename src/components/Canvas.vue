<template>
  <component :is="editor.findResolver(component)" v-bind="$attrs">
    <slot></slot>
    <template #drag-handle><slot name="drag-handle" /></template>
  </component>
</template>

<script>
export default {
  props: {
    component: String,
  },
  inject: [
    'editor',
  ],
};
</script>

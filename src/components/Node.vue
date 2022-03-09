<template>
  <component
    :is="editor.findResolver(node.componentName)"
    v-bind="node.props"
    :class="{ 'cf-node-selected': isSelected, 'cf-node-draggable': isDraggable }"
    :draggable="dragEnabled && isDraggable"
    @dragstart.native="handleDragStart"
    @dragover.native="handleDragOver"
    @dragleave.native="handleDragLeave"
    @drop.native="handleDrop"
    @dragend.native="handleDragEnd"
    @click.native="selectNode"
    :id="`editor-node-${node.uuid}`"
  >
    <Node
      v-for="node in node.children" :key="node.uuid"
      :node="node"
    />
    <template #drag-handle><span v-if="editor.enabled && isSelected" class="cf-node-drag-handle" @mousedown="dragEnabled = true" @mouseup="dragEnabled = false" /></template>
  </component>
</template>

<script>
import Node from '../core/Node';
import NodeService from '../core/services/NodeService';

export default {
  name: 'Node',
  props: {
    node: Node,
  },
  inject: [
    'editor',
  ],
  data() {
    return {
      nodeService: new NodeService(this),
      dragEnabled: false,
    };
  },
  computed: {
    isSelected() {
      return this.node === this.editor.selectedNode;
    },
    isDraggable() {
      return this.editor.enabled && this.node.isDraggable();
    }
  },
  provide() {
    return {
      node: this.node,
    };
  },
  methods: {
    cancelDefault(event) {
      event.stopPropagation();
      event.preventDefault();
    },
    handleDragStart(event) {
      event.stopPropagation();

      if (!this.editor.enabled) {
        // it is used to cancel dragging
        event.preventDefault();
        return;
      }

      this.editor.dragNode(this.node);
    },
    handleDragOver(event) {
      this.cancelDefault(event);

      this.nodeService.handleDragOver({
        clientX: event.clientX,
        clientY: event.clientY,
      });
    },
    handleDrop(event) {
      this.cancelDefault(event);

      this.nodeService.handleDrop({
        clientX: event.clientX,
        clientY: event.clientY,
      });
    },
    handleDragLeave(event) {
      // Marks as invalid target when dragging off a section
      this.cancelDefault(event);
      this.editor.indicator.setIsForbidden(true);
    },
    handleDragEnd(event) {
      event.stopPropagation();

      this.editor.dragNode(null);
      this.editor.indicator.hide();
      this.dragEnabled = false;
    },
    selectNode(event) {
      event.stopPropagation();

      if (!this.editor.enabled) {
        return;
      }

      this.editor.selectNode(this.node);
    },
  },
};
</script>

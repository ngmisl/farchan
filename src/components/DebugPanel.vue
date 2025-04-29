<script setup lang="ts">
import { ref } from 'vue';

defineProps<{
  context: Record<string, unknown> | undefined;
}>();

const isOpen = ref(false);

const togglePanel = () => {
  isOpen.value = !isOpen.value;
};
</script>

<template>
  <div class="debug-panel">
    <h3>Frame Context</h3>
    <button @click="togglePanel" class="debug-button">
      <span>{{ isOpen ? '▼' : '►' }}</span>
      {{ isOpen ? 'Hide' : 'Show' }} Frame Context
    </button>
    
    <div v-if="isOpen" class="context-content">
      <pre>{{ JSON.stringify(context, null, 2) }}</pre>
    </div>
  </div>
</template>

<style scoped>
.debug-panel {
  margin-top: 1rem;
  padding: 1rem;
  background-color: #f5f5f5;
  border-radius: 8px;
  width: 100%;
}

.debug-button {
  padding: 0.6rem;
  background-color: #e0e0e0;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 0.95rem;
}

.context-content {
  margin-top: 1rem;
  padding: 1rem;
  background-color: #ffffff;
  border-radius: 4px;
  overflow-x: auto;
  font-size: 0.8rem;
  white-space: pre-wrap;
}
</style>

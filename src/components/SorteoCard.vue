<template>
  <div class="sorteos-card">
    <div class="card-header">
      <SorteosTabs :activeTab="activeTab" @change-tab="activeTab = $event" />
      <button class="new-draw" @click="showForm = true">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="24"
          height="24"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2"
          stroke-linecap="round"
          stroke-linejoin="round"
          class="lucide"
        >
          <path d="M5 12h14"></path>
          <path d="M12 5v14"></path>
        </svg>
        <span>Nuevo Sorteo</span>
      </button>
    </div>

    <hr />

    <SorteosTable v-if="!showForm" :activeTab="activeTab" />
    <SorteosForm v-else @close-form="showForm = false" />
  </div>
</template>

<script setup>
import { ref } from 'vue';
import SorteosTabs from './SorteosTabs.vue';
import SorteosTable from './SorteosTable.vue';
import SorteosForm from './SorteosForm.vue';

const activeTab = ref('recientes');
const showForm = ref(false);
</script>

<style scoped>
.sorteos-card {
  background: var(--color-background);
  border-radius: 10px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  padding: 15px;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
}

/* Botón Nuevo Sorteo */
.new-draw {
  background: var(--color-primary);
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 6px;
  display: flex;
  align-items: center;
  gap: 8px; /* Espacio entre el ícono y el texto */
  cursor: pointer;
  transition: 0.3s;
}

/* Estilos del ícono */
.new-draw svg {
  stroke: white; /* Color del ícono */
  width: 18px;
  height: 18px;
}

.new-draw:hover {
  background: #7c3aed;
}
</style>

<template>
  <div class="tabs">
    <button 
      :class="{ active: activeTab === 'recientes' }" 
      @click="$emit('change-tab', 'recientes')"
    >
      Sorteos Recientes
    </button>
    <button 
      :class="{ active: activeTab === 'historial' }" 
      @click="$emit('change-tab', 'historial')"
    >
      Historial
    </button>
  </div>
</template>

<script setup>
defineProps({
  activeTab: String
});
defineEmits(['change-tab']);
</script>

<style scoped>
/* Contenedor de las pestañas */
.tabs {
  display: flex;
  background: var(--color-tabs-bg); /* Color de fondo */
  padding: 6px; /* Espaciado interno */
  border-radius: 8px;
  align-items: center;
  justify-content: flex-start; /* Alinear los botones a la izquierda */
  gap: 10px; /* Espacio entre botones */
}

/* Botones */
.tabs button {
  padding: 10px 20px;
  border: none;
  background: transparent;
  font-size: 14px;
  font-weight: 600;
  color: var(--color-tabs-text); /* Color del texto */
  cursor: pointer;
  transition: background 0.3s, color 0.3s;
  border-radius: 6px;
}

/* Botón activo */
.tabs button.active {
  background: white;
  color: black;
  font-weight: 600;
  border-radius: 6px;
}
</style>

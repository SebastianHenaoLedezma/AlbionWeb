<template>
    <div class="table-container">
        <!-- Sección de Filtros -->
        <div class="filters">
            <!-- Botón "Volver" cuando un usuario/personaje está seleccionado -->
            <button v-if="selectedFilter" class="back-btn" @click="resetFilter">
                ⬅ Volver
            </button>

            <!-- Título de Historial del Usuario/Personaje -->
            <h3 v-if="selectedFilter">
                Historial de
                {{
                    filterType === "usuario"
                        ? "@" + selectedFilter
                        : selectedFilter
                }}
            </h3>

            <!-- Contenedor del input y el select -->
            <div v-else class="filters-container">
                <input
                    type="text"
                    v-model="searchQuery"
                    placeholder="Buscar por usuario o personaje..."
                    class="search-input"
                />

                <select
                    v-model="selectedPremio"
                    v-if="activeTab === 'historial'"
                    class="select-filter"
                >
                    <option value="">Todos los premios</option>
                    <option
                        v-for="premio in premiosDisponibles"
                        :key="premio"
                        :value="premio"
                    >
                        {{ premio.charAt(0).toUpperCase() + premio.slice(1) }}
                    </option>
                </select>
            </div>
        </div>

        <!-- Encabezado de la Tabla -->
        <div class="table-header">
            <span>Fecha y Hora</span>
            <span>Usuario de Twitch</span>
            <span>Personaje</span>
            <span>Premio</span>
            <span>Estado</span>
            <span>Acciones</span>
        </div>

        <!-- Cuerpo de la Tabla -->
        <div v-if="sorteosFiltrados.length" class="table-body">
            <div
                v-for="(sorteo, index) in sorteosFiltrados"
                :key="index"
                class="table-row"
            >
                <span>{{ sorteo.fecha }}</span>

                <!-- Usuario de Twitch Clickeable -->
                <span
                    class="clickable user-twitch"
                    @click="filterBy('usuario', sorteo.usuario)"
                >
                    @{{ sorteo.usuario }}
                </span>

                <!-- Personaje Clickeable -->
                <span
                    class="clickable"
                    @click="filterBy('personaje', sorteo.personaje)"
                >
                    {{ sorteo.personaje }}
                </span>

                <!-- Mostrar el icono de premio correspondiente -->
                <span class="premio">
                    <span
                        v-if="sorteo.premio === 'arma'"
                        v-html="svgEspada"
                    ></span>
                    <span
                        v-if="sorteo.premio === 'montura'"
                        v-html="svgMontura"
                    ></span>
                    {{ sorteo.premio }}
                </span>

                <!-- Estado con Icono -->
                <span
                    :class="{
                        'status-pending': !sorteo.entregado,
                        'status-delivered': sorteo.entregado,
                    }"
                >
                    <span v-if="!sorteo.entregado">⏳</span>
                    {{ sorteo.entregado ? "Entregado" : "Pendiente" }}
                </span>

                <button
                    class="deliver-btn"
                    @click="marcarEntregado(index)"
                    v-if="!sorteo.entregado"
                >
                    Marcar Entregado
                </button>
            </div>
        </div>

        <p v-else class="no-data">No hay sorteos en esta categoría.</p>
    </div>
</template>

<script setup>
import { ref, computed, onMounted } from "vue";

const props = defineProps({
    activeTab: String, // Recibe la pestaña activa desde el componente padre
});

const sorteos = ref([]);
const searchQuery = ref("");
const selectedPremio = ref("");
const selectedFilter = ref(null); // Guarda el usuario/personaje seleccionado
const filterType = ref(null); // Indica si el filtro es por usuario o personaje

/** Cargar sorteos al montar el componente */
onMounted(() => {
    const data = localStorage.getItem("sorteos");
    if (data) {
        sorteos.value = JSON.parse(data);
    }
});

const svgEspada = `
  <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="#ef4444" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-sword mr-2 h-4 w-4 text-red-500">
    <polyline points="14.5 17.5 3 6 3 3 6 3 17.5 14.5"></polyline>
    <line x1="13" x2="19" y1="19" y2="13"></line>
    <line x1="16" x2="20" y1="16" y2="20"></line>
    <line x1="19" x2="21" y1="21" y2="19"></line>
  </svg>
`;

const svgMontura = `
  <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="#3b82f6" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-dog mr-2 h-4 w-4 text-blue-500">
    <path d="M10 5.172C10 3.782 8.423 2.679 6.5 3c-2.823.47-4.113 6.006-4 7 .08.703 1.725 1.722 3.656 1 1.261-.472 1.96-1.45 2.344-2.5"></path>
    <path d="M14.267 5.172c0-1.39 1.577-2.493 3.5-2.172 2.823.47 4.113 6.006 4 7-.08.703-1.725 1.722-3.656 1-1.261-.472-1.855-1.45-2.239-2.5"></path>
    <path d="M8 14v.5"></path>
    <path d="M16 14v.5"></path>
    <path d="M11.25 16.25h1.5L12 17l-.75-.75Z"></path>
    <path d="M4.42 11.247A13.152 13.152 0 0 0 4 14.556C4 18.728 7.582 21 12 21s8-2.272 8-6.444c0-1.061-.162-2.2-.493-3.309m-9.243-6.082A8.801 8.801 0 0 1 12 5c.78 0 1.5.108 2.161.306"></path>
  </svg>
`;

/** Obtener premios únicos */
const premiosDisponibles = computed(() => {
    return [...new Set(sorteos.value.map((s) => s.premio))];
});

/** Filtrar sorteos según la pestaña, búsqueda y premio */
const sorteosFiltrados = computed(() => {
    let filtrados =
        props.activeTab === "recientes"
            ? sorteos.value.filter((sorteo) => !sorteo.entregado) // Solo los no entregados
            : sorteos.value; // Mostrar todos en historial

    if (selectedFilter.value) {
        filtrados = filtrados.filter((sorteo) =>
            filterType.value === "usuario"
                ? sorteo.usuario === selectedFilter.value
                : sorteo.personaje === selectedFilter.value
        );
    } else {
        if (searchQuery.value) {
            const query = searchQuery.value.toLowerCase();
            filtrados = filtrados.filter(
                (sorteo) =>
                    sorteo.usuario.toLowerCase().includes(query) ||
                    sorteo.personaje.toLowerCase().includes(query)
            );
        }

        if (selectedPremio.value) {
            filtrados = filtrados.filter(
                (sorteo) => sorteo.premio === selectedPremio.value
            );
        }
    }

    return filtrados;
});

/** Función para filtrar por usuario o personaje */
const filterBy = (tipo, valor) => {
    selectedFilter.value = valor;
    filterType.value = tipo;
};

/** Restablecer la vista completa */
const resetFilter = () => {
    selectedFilter.value = null;
    filterType.value = null;
};

const obtenerIconoPremio = (premio) => {
    return premio === "montura" ? "🦄" : "⚔️";
};

const marcarEntregado = (index) => {
    const sorteoIndex = sorteos.value.findIndex(
        (s) => s === sorteosFiltrados.value[index]
    );
    if (sorteoIndex !== -1) {
        sorteos.value[sorteoIndex].entregado = true;
        localStorage.setItem("sorteos", JSON.stringify(sorteos.value));
    }
};
</script>

<style scoped>
.table-container {
    padding-top: 10px;
}
/* Filtros */
.filters {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    gap: 10px;
    margin-bottom: 10px;
    width: 100%;
}

/* Contenedor del buscador y selector */
.filters-container {
    display: flex;
    width: 100%;
    gap: 10px;
}

.filters input,
.filters select {
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 5px;
    font-size: 14px;
}

/* Input de búsqueda ocupa más espacio */
.search-input {
    flex: 2;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 5px;
    font-size: 14px;
}
/* Selector ocupa menos espacio */
.select-filter {
    flex: 1;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 5px;
    font-size: 14px;
}

/* Botón Volver */
.back-btn {
    background: #ccc;
    color: black;
    border: none;
    padding: 8px 16px;
    border-radius: 6px;
    cursor: pointer;
    font-weight: 600;
}

/* Tabla */
.table-header,
.table-row {
    display: grid;
    grid-template-columns: 1.5fr 2fr 1.5fr 1.5fr 1.5fr 1.5fr;
    padding: 1.1rem 0;
    border-bottom: 1px solid #ddd;
}

.table-header {
    font-weight: bold;
    background: #f8f8f8;
    padding: 10px;
}

.table-body {
    background: white;
}

.table-row {
    display: grid;
    align-items: center;
    font-size: 14px;
}

/* Clickable */
.clickable {
    cursor: pointer;
    text-decoration: none;
    transition: color 0.2s;
}

.user-twitch {
    color: var(--color-primary);
}

.clickable:hover {
    text-decoration: underline;
}

/* Estado */
.status-pending {
    color: #f39c12;
    font-weight: bold;
    display: flex;
    align-items: center;
    gap: 5px;
}

.status-delivered {
    color: #2ecc71;
    font-weight: bold;
}

/* Botón de entrega */
.deliver-btn {
    background: #2ecc71;
    color: white;
    border: none;
    padding: 5px 10px;
    border-radius: 5px;
    cursor: pointer;
}

/* Mensaje de "No hay sorteos" */
.no-data {
    text-align: center;
    font-size: 14px;
    color: #777;
    margin-top: 10px;
}

/* Botón Volver */
.back-btn {
    background: #ccc;
    color: black;
    border: none;
    padding: 8px 16px;
    border-radius: 6px;
    cursor: pointer;
    font-weight: 600;
}
</style>

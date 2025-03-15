<template>
    <div class="table-container">
        <!-- Sección de Filtros -->
        <div class="filters">
            <!-- Botón "Volver" cuando un usuario/personaje está seleccionado -->
            <button v-if="selectedFilter" class="back-btn" @click="resetFilter">
                <svg
                    xmlns="http://www.w3.org/2000/svg"
                    width="18"
                    height="18"
                    viewBox="0 0 24 24"
                    fill="none"
                    stroke="currentColor"
                    stroke-width="2"
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    class="lucide lucide-arrow-left h-4 w-4 mr-2"
                >
                    <path d="m12 19-7-7 7-7"></path>
                    <path d="M19 12H5"></path>
                </svg>
                <span>Volver</span>
            </button>

            <!-- Título de Historial del Usuario/Personaje -->
            <h3 v-if="selectedFilter" class="filters-title">
                <span class="filters-subtitle">
                    <span
                        ><svg
                            xmlns="http://www.w3.org/2000/svg"
                            width="22"
                            height="22"
                            viewBox="0 0 24 24"
                            fill="none"
                            stroke="var(--color-primary)"
                            stroke-width="2"
                            stroke-linecap="round"
                            stroke-linejoin="round"
                            class="lucide lucide-user h-4 w-4"
                        >
                            <path
                                d="M19 21v-2a4 4 0 0 0-4-4H9a4 4 0 0 0-4 4v2"
                            ></path>
                            <circle cx="12" cy="7" r="4"></circle></svg></span
                    >Historial de usuario:
                </span>
                <span class="filters-username">
                    {{
                        filterType === "usuario"
                            ? "@" + selectedFilter
                            : selectedFilter
                    }}</span
                >
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
            <span @click="toggleSortByDate" class="sortable">
                Fecha y Hora
                <svg
                    v-if="sortByDateDesc"
                    xmlns="http://www.w3.org/2000/svg"
                    width="16"
                    height="16"
                    viewBox="0 0 24 24"
                    fill="none"
                    stroke="currentColor"
                    stroke-width="2"
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    class="lucide lucide-chevron-down"
                >
                    <path d="m6 9 6 6 6-6"></path>
                </svg>
                <svg
                    v-else
                    xmlns="http://www.w3.org/2000/svg"
                    width="16"
                    height="16"
                    viewBox="0 0 24 24"
                    fill="none"
                    stroke="currentColor"
                    stroke-width="2"
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    class="lucide lucide-chevron-up"
                >
                    <path d="m18 15-6-6-6 6"></path>
                </svg>
            </span>
            <span>Usuario de Twitch</span>
            <span>Personaje</span>
            <span>Premio</span>
            <span>Estado</span>
            <span>Acciones</span>
            <span>Eliminar</span>
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
                        class="premio_icon"
                    ></span>
                    <span
                        v-if="sorteo.premio === 'montura'"
                        v-html="svgMontura"
                        class="premio_icon"
                    ></span>
                    {{ sorteo.premio }}
                </span>

                <!-- Estado con Icono -->
                <!-- Estado con Tooltip -->
                <!-- Contenedor del estado con tooltip -->
                <!-- Contenedor del estado con tooltip (Solo para entregados) -->
                <div class="tooltip-container">
                    <span
                        :class="{
                            'status-pending': !sorteo.entregado,
                            'status-delivered': sorteo.entregado,
                        }"
                        @mouseover="
                            sorteo.entregado
                                ? mostrarTooltip(index, sorteo.fechaEntrega)
                                : null
                        "
                        @mouseleave="ocultarTooltip"
                    >
                        <span v-if="!sorteo.entregado">
                            <svg
                                xmlns="http://www.w3.org/2000/svg"
                                width="16"
                                height="16"
                                viewBox="0 0 24 24"
                                fill="none"
                                stroke="currentColor"
                                stroke-width="2"
                                stroke-linecap="round"
                                stroke-linejoin="round"
                                class="lucide lucide-clock mr-1 h-3 w-3"
                            >
                                <circle cx="12" cy="12" r="10"></circle>
                                <polyline points="12 6 12 12 16 14"></polyline>
                            </svg>
                        </span>

                        <span v-if="sorteo.entregado">
                            <svg
                                xmlns="http://www.w3.org/2000/svg"
                                width="16"
                                height="16"
                                viewBox="0 0 24 24"
                                fill="none"
                                stroke="currentColor"
                                stroke-width="2"
                                stroke-linecap="round"
                                stroke-linejoin="round"
                                class="lucide lucide-check h-3 w-3"
                            >
                                <path d="M20 6 9 17l-5-5"></path>
                            </svg>
                        </span>

                        {{ sorteo.entregado ? "Entregado" : "Pendiente" }}

                        <span v-if="sorteo.entregado">
                            <svg
                                xmlns="http://www.w3.org/2000/svg"
                                width="16"
                                height="16"
                                viewBox="0 0 24 24"
                                fill="none"
                                stroke="currentColor"
                                stroke-width="2"
                                stroke-linecap="round"
                                stroke-linejoin="round"
                                class="lucide lucide-calendar h-3 w-3 ml-1"
                            >
                                <path d="M8 2v4"></path>
                                <path d="M16 2v4"></path>
                                <rect
                                    width="18"
                                    height="18"
                                    x="3"
                                    y="4"
                                    rx="2"
                                ></rect>
                                <path d="M3 10h18"></path>
                            </svg>
                        </span>
                    </span>

                    <!-- Tooltip flotante solo para entregados -->
                    <div
                        v-if="
                            tooltipVisible &&
                            tooltipIndex === index &&
                            sorteo.entregado
                        "
                        class="tooltip"
                    >
                        <div class="tooltip-arrow"></div>
                        Entregado el: {{ tooltipFecha }}
                    </div>
                </div>

                <button
                    class="deliver-btn"
                    @click="marcarEntregado(index)"
                    :class="{ 'hidden-btn': sorteo.entregado }"
                    v-if="!sorteo.entregado"
                >
                    Entregado
                </button>
                <!-- Botón para eliminar -->
                <span class="delete-btn" @click="eliminarSorteo(index)">
                    <svg
                        xmlns="http://www.w3.org/2000/svg"
                        width="16"
                        height="16"
                        viewBox="0 0 24 24"
                        fill="none"
                        stroke="currentColor"
                        stroke-width="2"
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        class="lucide lucide-trash h-4 w-4"
                    >
                        <path d="M3 6h18"></path>
                        <path d="M8 6V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"></path>
                        <path d="M10 11v6"></path>
                        <path d="M14 11v6"></path>
                        <path
                            d="M4 6h16l-1 14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2L4 6z"
                        ></path>
                    </svg>
                </span>
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
const tooltipVisible = ref(false);
const tooltipFecha = ref("");
const tooltipIndex = ref(null);

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

const sortByDateDesc = ref(true); // Controla el orden de la fecha (true = descendente, false = ascendente)

const parseFecha = (fechaStr) => {
    try {
        // Separar fecha y hora
        const [fecha, hora] = fechaStr.split(", ");

        // Separar día, mes y año
        const [dia, mes, año] = fecha.split("/").map(Number);

        // Separar hora, minutos y segundos
        const [horas, minutos, segundos] = hora.split(":").map(Number);

        // Construir nueva fecha en formato adecuado
        const fechaValida = new Date(
            año,
            mes - 1,
            dia,
            horas,
            minutos,
            segundos
        );

        return fechaValida;
    } catch (error) {
        console.error("Error al convertir la fecha:", error);
        return new Date(0); // Fecha inválida para evitar fallos en la comparación
    }
};

const sorteosOrdenados = computed(() => {
    const sorteosCopia = [...sorteos.value].sort((a, b) => {
        const fechaA = parseFecha(a.fecha);
        const fechaB = parseFecha(b.fecha);

        return sortByDateDesc.value ? fechaB - fechaA : fechaA - fechaB;
    });

    return sorteosCopia;
});

const sorteosFiltrados = computed(() => {
    let filtrados =
        props.activeTab === "recientes"
            ? sorteosOrdenados.value.filter((sorteo) => !sorteo.entregado) // Solo los no entregados
            : sorteosOrdenados.value; // Mostrar todos en historial

    // Aplicar filtro de usuario/personaje
    if (selectedFilter.value) {
        filtrados = filtrados.filter((sorteo) =>
            filterType.value === "usuario"
                ? sorteo.usuario.toLowerCase() === selectedFilter.value.toLowerCase()
                : sorteo.personaje.toLowerCase() === selectedFilter.value.toLowerCase()
        );
    }

    // Aplicar filtro de búsqueda
    if (searchQuery.value.trim() !== "") {
        const query = searchQuery.value.toLowerCase();
        filtrados = filtrados.filter(
            (sorteo) =>
                sorteo.usuario.toLowerCase().includes(query) ||
                sorteo.personaje.toLowerCase().includes(query)
        );
    }

    // Aplicar filtro por tipo de premio
    if (selectedPremio.value !== "") {
        filtrados = filtrados.filter((sorteo) => sorteo.premio === selectedPremio.value);
    }

    return filtrados;
});


/** Cambiar el orden al hacer clic en "Fecha y Hora" */
const toggleSortByDate = () => {
    sortByDateDesc.value = !sortByDateDesc.value; // Alternar el orden de fechas
};

/** Obtener premios únicos */
const premiosDisponibles = computed(() => {
    return [...new Set(sorteos.value.map((s) => s.premio))];
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

const mostrarTooltip = (index, fechaEntrega) => {
    if (fechaEntrega) {
        // Solo muestra si hay fecha de entrega
        tooltipIndex.value = index;
        tooltipFecha.value = fechaEntrega;
        tooltipVisible.value = true;
    }
};

const eliminarSorteo = (index) => {
    const sorteoIndex = sorteos.value.findIndex(
        (s) => s === sorteosFiltrados.value[index]
    );

    if (sorteoIndex !== -1) {
        const confirmar = confirm("¿Estás seguro de eliminar este sorteo?");
        if (confirmar) {
            // Eliminar el sorteo del array
            sorteos.value.splice(sorteoIndex, 1);

            // Actualizar el localStorage
            localStorage.setItem("sorteos", JSON.stringify(sorteos.value));
        }
    }
};

const ocultarTooltip = () => {
    tooltipVisible.value = false;
    tooltipIndex.value = null;
};

const marcarEntregado = (index) => {
    const sorteoIndex = sorteos.value.findIndex(
        (s) => s === sorteosFiltrados.value[index]
    );

    if (sorteoIndex !== -1) {
        // Guardamos la fecha actual como fecha de entrega
        const fechaActual = new Date();
        const fechaEntregaStr = `${fechaActual.getDate()}/${
            fechaActual.getMonth() + 1
        }/${fechaActual.getFullYear()}, ${fechaActual.getHours()}:${fechaActual.getMinutes()}:${fechaActual.getSeconds()}`;

        sorteos.value[sorteoIndex].entregado = true;
        sorteos.value[sorteoIndex].fechaEntrega = fechaEntregaStr; // Guardamos la fecha de entrega

        localStorage.setItem("sorteos", JSON.stringify(sorteos.value));
    }
};
</script>

<style lang="scss" scoped>
.table-container {
    padding-top: 10px;
}
/* Filtros */
.filters {
    display: flex;
    flex-direction: row;
    align-items: center;
    gap: 15px;
    margin-bottom: 10px;
    width: 100%;
    .filters-title {
        background-color: #faf5ff;
        padding: 5px 10px;
        border-radius: 10px;
        display: flex;
        gap: 7px;
        .filters-subtitle {
            color: #842acdce;
            display: flex;
            align-items: center;
            flex-wrap: nowrap;
            gap: 4px;
        }
        .filters-username {
            color: #7a26c0;
        }
    }
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
    color: black;
    border: none;
    padding: 8px 16px;
    border-radius: 6px;
    cursor: pointer;
    font-weight: 600;
    display: flex;
    align-items: center;
    gap: 3px;
    span {
        height: 18px;
        text-align: center;
        font-size: 15px;
    }
}

/* Tabla */
.table-header,
.table-row {
    display: grid;
    grid-template-columns: 1.5fr 2fr 1.5fr 1.5fr 1.5fr 1.5fr 0.5fr;
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
    background-color: rgb(255, 251, 235);
    border: 1px solid rgb(253, 230, 138);
    width: max-content;
    padding: 2px 10px;
    border-radius: 20px;
    display: flex;
    align-items: center;
    justify-content: center;
    color: #b45309;
    font-weight: bold;
    gap: 5px;
}

.status-pending > span {
    height: 16px;
}

.status-delivered {
    color: #15803d;
    font-weight: bold;
    background-color: #f0fdf4;
    border: 1px solid rgb(187, 247, 208);
    width: max-content;
    padding: 2px 10px;
    border-radius: 20px;
    display: flex;
    align-items: center;
    font-weight: bold;
    gap: 5px;
}
.status-delivered > span {
    height: 16px;
}

/* Botón de entrega */
.deliver-btn {
    color: #16a34a;
    border: 1px solid #2ecc71;
    padding: 5px 10px;
    border-radius: 5px;
    cursor: pointer;
    background-color: transparent;
    width: max-content;
    padding: 10px;
    font-weight: 600;
    font-family: var(--font-primary);
}

/* Mensaje de "No hay sorteos" */
.no-data {
    text-align: center;
    font-size: 14px;
    color: #777;
    margin-top: 10px;
}

.premio {
    display: flex;
    align-items: center;
    gap: 7px;
}
.premio_icon {
    height: 24px;
}

/* Hace que la columna "Fecha y Hora" sea clickeable */
.sortable {
    cursor: pointer;
    display: flex;
    align-items: center;
    gap: 5px;
    font-weight: bold;
}

.tooltip-container {
    position: relative;
    display: inline-block;
}

.tooltip {
    position: absolute;
    top: -130%;
    left: 25%;
    transform: translateX(-50%);
    background: rgba(0, 0, 0, 0.85);
    color: white;
    padding: 6px 12px;
    font-size: 12px;
    border-radius: 6px;
    white-space: nowrap;
    box-shadow: 0px 2px 8px rgba(0, 0, 0, 0.2);
    z-index: 10;
    opacity: 0;
    animation: fadeIn 0.3s forwards;
}
/* Asegura que la columna de eliminar esté centrada */
.delete-btn {
    cursor: pointer;
    display: flex;
    align-items: center;
    grid-column: 7 / 8;
    color: red;
    width: max-content;
    padding: 10px;
    border-radius: 9px;
    transition: background-color 0.3s ease-in-out, transform 0.2s ease-in-out;
}

.delete-btn:hover {
    background-color: #fac0c08a;
    transform: scale(1.1); /* Pequeño efecto de zoom */
}

/* Mantiene la alineación de las columnas */
.table-row {
    display: grid;
    grid-template-columns: 1.5fr 2fr 1.5fr 1.5fr 1.5fr 1.5fr 0.5fr;
    align-items: center;
}

/* Flechita del tooltip */
.tooltip-arrow {
    position: absolute;
    bottom: -5px;
    left: 50%;
    transform: translateX(-50%);
    width: 0;
    height: 0;
    border-left: 6px solid transparent;
    border-right: 6px solid transparent;
    border-top: 6px solid rgba(0, 0, 0, 0.85);
}

@keyframes fadeIn {
    from {
        opacity: 0;
        transform: translateX(-50%) translateY(-5px);
    }
    to {
        opacity: 1;
        transform: translateX(-50%) translateY(0);
    }
}
</style>

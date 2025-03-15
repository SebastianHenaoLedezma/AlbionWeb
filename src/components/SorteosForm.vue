<template>
    <div class="form-container">
        <!-- Encabezado del formulario -->
        <div class="form-header">
            <h2>Registrar Nuevo Sorteo</h2>
            <button class="close-btn" @click="$emit('close-form')">✖</button>
        </div>

        <!-- Formulario -->
        <form @submit.prevent="guardarSorteo">
            <!-- Usuario y Personaje -->
            <div class="form-row">
                <div class="form-group">
                    <label>Usuario de Twitch</label>
                    <input
                        type="text"
                        placeholder="@usuario"
                        v-model="nuevoSorteo.usuario"
                        @input="filtrarUsuarios"
                    />
                    <ul v-if="filtroUsuarios.length" class="suggestions">
                        <li
                            v-for="usuario in filtroUsuarios"
                            :key="usuario"
                            @click="seleccionarUsuario(usuario)"
                        >
                            @{{ usuario }}
                        </li>
                    </ul>
                </div>

                <div class="form-group">
                    <label>Nombre del Personaje</label>
                    <input
                        type="text"
                        placeholder="Nombre en el juego"
                        v-model="nuevoSorteo.personaje"
                        @input="filtrarPersonajes"
                    />
                    <ul v-if="filtroPersonajes.length" class="suggestions">
                        <li
                            v-for="personaje in filtroPersonajes"
                            :key="personaje"
                            @click="seleccionarPersonaje(personaje)"
                        >
                            {{ personaje }}
                        </li>
                    </ul>
                </div>
            </div>

            <!-- Tipo de Premio -->
            <div class="form-group">
                <label>Tipo de Premio</label>
                <select v-model="nuevoSorteo.premio">
                    <option value="montura">Montura</option>
                    <option value="arma">Arma</option>
                </select>
            </div>

            <!-- Checkbox de Premio Entregado -->
            <div class="checkbox-group">
                <input
                    type="checkbox"
                    id="entregado"
                    v-model="nuevoSorteo.entregado"
                />
                <label for="entregado">Premio ya entregado</label>
            </div>

            <!-- Botones de acción -->
            <div class="form-actions">
                <button
                    type="button"
                    class="cancel-btn"
                    @click="$emit('close-form')"
                >
                    Cancelar
                </button>
                <button type="submit" class="save-btn">Guardar Sorteo</button>
            </div>
        </form>
    </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

const nuevoSorteo = ref({
    usuario: "",
    personaje: "",
    premio: "montura",
    entregado: false,
});

const emit = defineEmits(["close-form"]);

const usuariosGuardados = ref({});
const filtroUsuarios = ref([]);
const filtroPersonajes = ref([]);

onMounted(() => {
    const data = localStorage.getItem("usuariosTwitch");
    if (data) {
        usuariosGuardados.value = JSON.parse(data);
    }
});

const filtrarUsuarios = () => {
    const query = nuevoSorteo.value.usuario.toLowerCase();
    filtroUsuarios.value = Object.keys(usuariosGuardados.value).filter((user) =>
        user.toLowerCase().includes(query)
    );
};

const filtrarPersonajes = () => {
    const query = nuevoSorteo.value.personaje.toLowerCase();
    filtroPersonajes.value = Object.values(usuariosGuardados.value).filter(
        (personaje) => personaje.toLowerCase().includes(query)
    );
};

const seleccionarUsuario = (usuario) => {
    if (usuario) {
        nuevoSorteo.value.usuario = usuario;
        nuevoSorteo.value.personaje = usuariosGuardados.value[usuario] || "";
    }
    filtroUsuarios.value = [];
};

const seleccionarPersonaje = (personaje) => {
    const usuario = Object.keys(usuariosGuardados.value).find(
        (user) => usuariosGuardados.value[user] === personaje
    );
    if (usuario) {
        nuevoSorteo.value.usuario = usuario;
        nuevoSorteo.value.personaje = personaje;
    }
    filtroPersonajes.value = [];
};

const guardarSorteo = () => {
    if (nuevoSorteo.value.usuario && nuevoSorteo.value.personaje) {
        usuariosGuardados.value[nuevoSorteo.value.usuario] =
            nuevoSorteo.value.personaje;
        localStorage.setItem(
            "usuariosTwitch",
            JSON.stringify(usuariosGuardados.value)
        );
    }

    const sorteos = JSON.parse(localStorage.getItem("sorteos")) || [];
    sorteos.push({ ...nuevoSorteo.value, fecha: new Date().toLocaleString() });
    localStorage.setItem("sorteos", JSON.stringify(sorteos));

    nuevoSorteo.value = {
        usuario: "",
        personaje: "",
        premio: "montura",
        entregado: false,
    };
    emit("close-form");
};
</script>

<style lang="scss" scoped>
/* Contenedor del formulario */
.form-container {
    border: 2px dashed #ddd;
    padding: 20px;
    border-radius: 10px;
    background: white;
}

/* Encabezado */
.form-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 15px;
}

.form-header h2 {
    font-size: 18px;
    font-weight: bold;
}

.close-btn {
    background: none;
    border: none;
    font-size: 18px;
    cursor: pointer;
}

/* Contenedor de la fila de inputs */
.form-row {
    display: flex;
    gap: 10px; /* Espaciado entre los inputs */
    margin-bottom: 15px;
    flex-wrap: wrap;
    width: 100%;
    max-width: 100%;
    box-sizing: border-box; /* Evita que se salga del contenedor */
}

/* Cada input ocupa el 50% del espacio sin salirse */
.form-group {
    flex: 1;
    min-width: calc(50% - 5px); /* Ajustamos para que respete el gap */
    position: relative;
    box-sizing: border-box; /* Evita que el padding o el border aumenten su tamaño */
}

label {
    display: block;
    font-size: 14px;
    font-weight: 600;
    margin-bottom: 5px;
}
input {
    width: 97.5%;
}

.checkbox-group {
    display: flex;
    align-items: center;
    margin-top: 15px;
    input {
        width: auto;
        margin-right: 5px;
    }
    label {
        margin: 0;
    }
}

select {
    width: 100%;
}

input,
select {
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 6px;
    font-size: 14px;
}

/* Sugerencias */
.suggestions {
  color: var(--color-primary);
    position: absolute;
    width: 100%;
    background: white;
    border: 1px solid #ccc;
    border-radius: 5px;
    max-height: 120px;
    overflow-y: auto;
    list-style: none;
    padding: 5px;
    margin-top: 7px;
    z-index: 1000;
}

.suggestions li {
    padding: 8px;
    cursor: pointer;
    transition: 0.3s;
}

.suggestions li:hover {
    background: #f0f0f0;
}

/* Botones */
.form-actions {
    display: flex;
    justify-content: flex-end;
    gap: 10px;
    margin-top: 20px;
}

.cancel-btn {
    background: white;
    border: 1px solid #ccc;
    padding: 10px 16px;
    border-radius: 6px;
    cursor: pointer;
    font-size: 14px;
}

.save-btn {
    background: var(--color-primary);
    color: white;
    border: none;
    padding: 10px 16px;
    border-radius: 6px;
    cursor: pointer;
    font-size: 14px;
}

.save-btn:hover {
    background: #7c3aed;
}
</style>

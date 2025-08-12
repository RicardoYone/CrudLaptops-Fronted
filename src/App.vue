<template>
  <div style="max-width: 800px; margin: 0 auto; padding: 2rem">
    <h1 style="font-size: 2rem; font-weight: bold; margin-bottom: 1rem">
      Gestión de Equipos
    </h1>

    <div style="margin-bottom: 1rem">
      <label style="display: block; margin-bottom: 0.5rem"
        >Pegar códigos (uno por línea):</label
      >
      <textarea
        v-model="codigosPegados"
        rows="4"
        style="width: 100%; margin-bottom: 0.5rem"
        placeholder="EQ001
EQ002
EQ999"
      ></textarea>
      <button @click="filtrarPorCodigos" style="margin-right: 0.5rem">
        Filtrar por códigos pegados
      </button>
      <button @click="limpiarFiltros">Mostrar todos</button>
    </div>

    <div
      v-if="codigosNoEncontrados.length > 0"
      style="
        background: #fff3cd;
        color: #856404;
        padding: 0.5rem;
        margin-bottom: 1rem;
        border-radius: 4px;
      "
    >
      <strong>Códigos no encontrados:</strong>
      {{ codigosNoEncontrados.join(", ") }}
    </div>

    <input
      v-model="busquedaGlobal"
      placeholder="Búsqueda global"
      style="margin-bottom: 1rem; width: 100%; padding: 0.5rem"
    />

    <table
      border="1"
      cellpadding="8"
      cellspacing="0"
      style="width: 100%; border-collapse: collapse"
    >
      <thead>
        <tr>
          <th>Código</th>
          <th>Cliente</th>
          <th>Tipo</th>
          <th>Estado</th>
          <th>Fecha Entrega</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="equipo in equiposFiltradosMostrados" :key="equipo.codigo">
          <td>{{ equipo.codigo }}</td>
          <td>{{ equipo.cliente }}</td>
          <td>{{ equipo.tipo }}</td>
          <td>{{ equipo.estado }}</td>
          <td>{{ formatDate(equipo.fecha_entrega) }}</td>
        </tr>
        <tr v-if="equiposFiltradosMostrados.length === 0">
          <td colspan="5" style="text-align: center">
            No hay equipos para mostrar
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from "vue";
import axios from "axios";

const equipos = ref([]);
const equiposFiltrados = ref([]);
const codigosPegados = ref("");
const codigosNoEncontrados = ref([]);
const busquedaGlobal = ref("");

const cargarEquipos = async () => {
  try {
    const response = await axios.get("/api/equipos");
    equipos.value = response.data;
    equiposFiltrados.value = equipos.value;

    console.log("Equipos cargados:", response.data);
  } catch (error) {
    console.error("Error al cargar equipos:", error);
  }
};

const filtrarPorCodigos = async () => {
  if (!codigosPegados.value.trim()) return;
  const codigos = codigosPegados.value
    .split("\n")
    .map((codigo) => codigo.trim())
    .filter((codigo) => codigo);
  try {
    const response = await axios.post("/api/validar-equipos", { codigos });
    codigosNoEncontrados.value = response.data.no_encontrados;
    equiposFiltrados.value = equipos.value.filter((equipo) =>
      codigos.some((codigo) => equipo.codigo.includes(codigo))
    );
  } catch (error) {
    console.error("Error al validar códigos:", error);
  }
};

const limpiarFiltros = () => {
  equiposFiltrados.value = equipos.value;
  codigosPegados.value = "";
  codigosNoEncontrados.value = [];
  busquedaGlobal.value = "";
};

const formatDate = (date) => {
  return date ? new Date(date).toLocaleDateString() : "N/A";
};

const equiposFiltradosMostrados = computed(() => {
  if (!busquedaGlobal.value) return equiposFiltrados.value;
  const texto = busquedaGlobal.value.toLowerCase();
  return equiposFiltrados.value.filter(
    (equipo) =>
      equipo.codigo?.toLowerCase().includes(texto) ||
      equipo.cliente?.toLowerCase().includes(texto) ||
      equipo.tipo?.toLowerCase().includes(texto) ||
      equipo.estado?.toLowerCase().includes(texto)
  );
});

onMounted(() => {
  cargarEquipos();
});
</script>

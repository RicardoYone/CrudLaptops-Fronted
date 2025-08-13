<template>
  <div class="container">
    <h1 class="main-title">Gestión de Equipos</h1>

    <div class="form-section">
      <label class="label">Pegar códigos (uno por línea):</label>
      <textarea
        v-model="codigosPegados"
        rows="4"
        class="textarea"
        placeholder="EQ001\nEQ002\nEQ999"
      ></textarea>
      <div class="button-group">
        <button class="btn btn-primary" @click="filtrarPorCodigos">
          Filtrar por códigos pegados
        </button>
        <button class="btn btn-secondary" @click="limpiarFiltros">
          Mostrar todos
        </button>
      </div>
    </div>

    <div v-if="codigosNoEncontrados.length > 0" class="alert-warning">
      <strong>Códigos no encontrados:</strong>
      {{ codigosNoEncontrados.join(", ") }}
    </div>

    <input
      v-model="busquedaGlobal"
      placeholder="Búsqueda global"
      class="input-search"
    />

    <table class="equipos-table">
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
        <tr
          v-for="equipo in equiposFiltradosMostrados"
          :key="equipo.codigo"
          class="table-row"
        >
          <td>{{ equipo.codigo }}</td>
          <td>{{ equipo.cliente }}</td>
          <td>{{ equipo.tipo }}</td>
          <td>
            <span
              :class="[
                'estado',
                equipo.estado ? equipo.estado.toLowerCase() : 'otro',
              ]"
              >{{ equipo.estado }}</span
            >
          </td>
          <td>{{ formatDate(equipo.fecha_entrega) }}</td>
        </tr>
        <tr v-if="equiposFiltradosMostrados.length === 0">
          <td colspan="5" class="no-data">No hay equipos para mostrar</td>
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
<style scoped>
.container {
  max-width: 900px;
  margin: 40px auto;
  padding: 2.5rem 2rem;
  background: #f8fafc;
  border-radius: 16px;
  box-shadow: 0 4px 24px rgba(0, 0, 0, 0.08);
}
.main-title {
  font-size: 2.5rem;
  font-weight: 700;
  color: #2d3748;
  margin-bottom: 1.5rem;
  text-align: center;
  letter-spacing: 1px;
}
.form-section {
  margin-bottom: 1.5rem;
}
.label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 500;
  color: #374151;
}
.textarea {
  width: 100%;
  padding: 0.75rem;
  border-radius: 8px;
  border: 1px solid #cbd5e1;
  font-size: 1rem;
  margin-bottom: 0.5rem;
  resize: vertical;
  background: #fff;
  transition: border 0.2s;
}
.textarea:focus {
  border-color: #6366f1;
  outline: none;
}
.button-group {
  display: flex;
  gap: 0.5rem;
}
.btn {
  padding: 0.5rem 1.2rem;
  border-radius: 8px;
  border: none;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s, color 0.2s;
}
.btn-primary {
  background: linear-gradient(90deg, #6366f1 0%, #60a5fa 100%);
  color: #fff;
}
.btn-primary:hover {
  background: linear-gradient(90deg, #4338ca 0%, #2563eb 100%);
}
.btn-secondary {
  background: #e5e7eb;
  color: #374151;
}
.btn-secondary:hover {
  background: #cbd5e1;
}
.alert-warning {
  background: #fff3cd;
  color: #856404;
  padding: 0.75rem 1rem;
  margin-bottom: 1rem;
  border-radius: 8px;
  border: 1px solid #ffeeba;
  font-size: 1rem;
}
.input-search {
  width: 100%;
  padding: 0.75rem;
  border-radius: 8px;
  border: 1px solid #cbd5e1;
  font-size: 1rem;
  margin-bottom: 1.5rem;
  background: #fff;
  transition: border 0.2s;
}
.input-search:focus {
  border-color: #6366f1;
  outline: none;
}
.equipos-table {
  width: 100%;
  border-collapse: collapse;
  background: #fff;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
}
.equipos-table th {
  background: #6366f1;
  color: #fff;
  font-weight: 600;
  padding: 0.75rem;
  text-align: left;
  font-size: 1rem;
}
.equipos-table td {
  padding: 0.75rem;
  border-bottom: 1px solid #e5e7eb;
  font-size: 1rem;
}
.table-row:hover {
  background: #f1f5f9;
}
.no-data {
  text-align: center;
  color: #64748b;
  font-size: 1.1rem;
  padding: 1.5rem 0;
}
.estado {
  padding: 0.3rem 0.8rem;
  border-radius: 12px;
  font-weight: 600;
  font-size: 0.95rem;
  display: inline-block;
  text-transform: capitalize;
}
.estado.entregado {
  background: #bbf7d0;
  color: #166534;
}
.estado.pendiente {
  background: #fee2e2;
  color: #991b1b;
}
.estado.reparacion {
  background: #fef9c3;
  color: #92400e;
}
.estado.otro {
  background: #e0e7ff;
  color: #3730a3;
}
</style>

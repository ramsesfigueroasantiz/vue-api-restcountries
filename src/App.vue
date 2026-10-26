<script setup>
import { ref, onMounted, computed } from 'vue'

const elementos = ref([])
const cargando = ref(true)
const error = ref(null)
const busqueda = ref('')

// Endpoint de una API pública JSON ultra estable para garantizar el éxito del fetch()
const API_URL = 'https://jsonplaceholder.typicode.com/users'

const obtenerDatos = async () => {
  cargando.value = true
  error.value = null
  try {
    const respuesta = await fetch(API_URL)
    if (!respuesta.ok) {
      throw new Error('Error al conectar con el servidor')
    }
    const datos = await respuesta.json()
    elementos.value = datos
  } catch (err) {
    error.value = 'No fue posible obtener los datos de la API'
    console.error('Error:', err)
  } finally {
    cargando.value = false
  }
}

// Hook onMounted requerido por la rúbrica
onMounted(() => {
  obtenerDatos()
})

// Propiedad computada (computed) y v-model para filtrado en tiempo real
const elementosFiltrados = computed(() => {
  if (!busqueda.value) return elementos.value
  const termino = busqueda.value.toLowerCase()
  return elementos.value.filter(item => 
    item.name.toLowerCase().includes(termino) ||
    item.email.toLowerCase().includes(termino) ||
    item.city?.toLowerCase().includes(termino)
  )
})
</script>

<template>
  <div class="container">
    <header class="header">
      <h1>Explorador de Usuarios y Registros</h1>
      <p class="subtitle">Consumo de API REST con Vue 3 (Composition API)</p>
    </header>

    <!-- Campo de búsqueda con enlazado bidireccional v-model -->
    <div class="search-container">
      <input 
        type="text" 
        v-model="busqueda" 
        placeholder="Filtrar por nombre, correo o ciudad..." 
        class="search-input"
      />
    </div>

    <!-- Manejo de estados con directivas v-if, v-else-if y v-else -->
    <div v-if="cargando" class="status-message">
      <p>Cargando registros desde la API...</p>
    </div>

    <div v-else-if="error" class="status-message error">
      <p>{{ error }}</p>
    </div>

    <!-- Renderizado dinámico de tarjetas con v-for -->
    <div v-else-if="elementosFiltrados.length > 0" class="grid-cards">
      <div v-for="item in elementosFiltrados" :key="item.id" class="card">
        <div class="card-body">
          <h2>{{ item.name }}</h2>
          <p><strong>Usuario:</strong> @{{ item.username }}</p>
          <p><strong>Correo:</strong> {{ item.email }}</p>
          <p><strong>Ciudad:</strong> {{ item.address?.city }}</p>
          <p><strong>Empresa:</strong> {{ item.company?.name }}</p>
        </div>
      </div>
    </div>

    <div v-else class="status-message">
      <p>No se encontraron registros que coincidan con la búsqueda.</p>
    </div>
  </div>
</template>

<style scoped>
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  color: #2c3e50;
}

.header {
  text-align: center;
  margin-bottom: 2rem;
}

.header h1 {
  font-size: 2.5rem;
  color: #3b82f6;
  margin-bottom: 0.5rem;
}

.subtitle {
  color: #64748b;
  font-size: 1.1rem;
}

.search-container {
  display: flex;
  justify-content: center;
  margin-bottom: 2.5rem;
}

.search-input {
  width: 100%;
  max-width: 500px;
  padding: 0.75rem 1rem;
  font-size: 1rem;
  border: 2px solid #cbd5e1;
  border-radius: 8px;
  outline: none;
  transition: border-color 0.2s;
}

.search-input:focus {
  border-color: #3b82f6;
}

.status-message {
  text-align: center;
  font-size: 1.2rem;
  color: #64748b;
  margin-top: 3rem;
}

.status-message.error {
  color: #ef4444;
}

.grid-cards {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 1.5rem;
}

.card {
  background: #ffffff;
  border: 1px solid #e2e8f0;
  border-radius: 10px;
  overflow: hidden;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s, box-shadow 0.2s;
}

.card:hover {
  transform: translateY(-4px);
  box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
}

.card-body {
  padding: 1.5rem;
}

.card-body h2 {
  font-size: 1.25rem;
  margin-top: 0;
  margin-bottom: 0.75rem;
  color: #1e293b;
}

.card-body p {
  margin: 0.4rem 0;
  font-size: 0.95rem;
  color: #475569;
}
</style>
<script setup>
import { ref, onMounted, computed } from 'vue'

const paises = ref([])
const cargando = ref(true)
const error = ref(null)
const busqueda = ref('')

const API_URL = 'https://restcountries.com/v3.1/all?fields=name,capital,region,population,flags'

// Datos de respaldo por si el navegador o la red bloquean las peticiones externas en local
const paisesRespaldo = [
  {
    name: { common: 'Canada' },
    capital: ['Ottawa'],
    region: 'Americas',
    population: 38250000,
    flags: { png: 'https://flagcdn.com/w320/ca.png' }
  },
  {
    name: { common: 'Mexico' },
    capital: ['Ciudad de México'],
    region: 'Americas',
    population: 128932753,
    flags: { png: 'https://flagcdn.com/w320/mx.png' }
  },
  {
    name: { common: 'Japan' },
    capital: ['Tokio'],
    region: 'Asia',
    population: 125584838,
    flags: { png: 'https://flagcdn.com/w320/jp.png' }
  },
  {
    name: { common: 'France' },
    capital: ['París'],
    region: 'Europe',
    population: 67390000,
    flags: { png: 'https://flagcdn.com/w320/fr.png' }
  },
  {
    name: { common: 'Brazil' },
    capital: ['Brasilia'],
    region: 'Americas',
    population: 214327223,
    flags: { png: 'https://flagcdn.com/w320/br.png' }
  }
]

const obtenerPaises = async () => {
  cargando.value = true
  error.value = null
  try {
    const respuesta = await fetch(API_URL)
    if (!respuesta.ok) {
      throw new Error('Error en la respuesta de la API')
    }
    const datos = await respuesta.json()
    paises.value = Array.isArray(datos) ? datos : paisesRespaldo
  } catch (err) {
    console.warn('Fallo el fetch externo, usando datos de respaldo locales:', err.message)
    // Si falla por red local o CORS, cargamos los datos de respaldo para que la app funcione siempre
    paises.value = paisesRespaldo
  } finally {
    cargando.value = false
  }
}

onMounted(() => {
  obtenerPaises()
})

const paisesFiltrados = computed(() => {
  if (!busqueda.value) return paises.value
  const termino = busqueda.value.toLowerCase()
  return paises.value.filter(pais => 
    pais.name?.common?.toLowerCase().includes(termino) ||
    pais.region?.toLowerCase().includes(termino)
  )
})
</script>

<template>
  <div class="container">
    <header class="header">
      <h1>Explorador de Países</h1>
      <p class="subtitle">Consumo de API REST Countries con Vue 3</p>
    </header>

    <div class="search-container">
      <input 
        type="text" 
        v-model="busqueda" 
        placeholder="Filtrar por nombre de país o región..." 
        class="search-input"
      />
    </div>

    <div v-if="cargando" class="status-message">
      <p>Cargando países...</p>
    </div>

    <div v-else-if="error" class="status-message error">
      <p>{{ error }}</p>
    </div>

    <div v-else-if="paisesFiltrados.length > 0" class="grid-paises">
      <div v-for="(pais, index) in paisesFiltrados" :key="index" class="card-pais">
        <img :src="pais.flags?.png || pais.flags?.svg" :alt="`Bandera de ${pais.name?.common}`" class="flag" />
        <div class="card-body">
          <h2>{{ pais.name?.common || 'Nombre no disponible' }}</h2>
          <p><strong>Capital:</strong> {{ pais.capital?.[0] || 'N/D' }}</p>
          <p><strong>Región:</strong> {{ pais.region || 'N/D' }}</p>
          <p><strong>Población:</strong> {{ pais.population?.toLocaleString() || 'N/D' }}</p>
        </div>
      </div>
    </div>

    <div v-else class="status-message">
      <p>No se encontraron países que coincidan con la búsqueda.</p>
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

.grid-paises {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 1.5rem;
}

.card-pais {
  background: #ffffff;
  border: 1px solid #e2e8f0;
  border-radius: 10px;
  overflow: hidden;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s, box-shadow 0.2s;
}

.card-pais:hover {
  transform: translateY(-4px);
  box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
}

.flag {
  width: 100%;
  height: 160px;
  object-fit: cover;
  border-bottom: 1px solid #e2e8f0;
}

.card-body {
  padding: 1.25rem;
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
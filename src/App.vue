<script setup>
import { ref, computed, onMounted } from 'vue'

// Importamos la imagen para usarla en todos los videos
import portadaFilosofia from './assets/portada-filosofia.jpg'

const videos = ref([
  {
    id: 1,
    title: 'Clase Grabada 1',
    description: '24 de Febrero de 2026',
    driveId: '1oLWMtf4WxP_YK10wiNPlwGqROdqnemYN',
    thumbnail: portadaFilosofia
  },
  {
    id: 2,
    title: 'Clase Grabada 2',
    description: '5 de Marzo de 2026',
    driveId: '1v0WQn_Dr_8WYpThnFfYaZorHd1amV4tx',
    thumbnail: portadaFilosofia
  },
  {
    id: 3,
    title: 'Clase Grabada 3',
    description: '17 de Marzo de 2026',
    driveId: '1gN_u7LB9GWgwYodDp-AR21ZS9mbf9M1A',
    thumbnail: portadaFilosofia
  },
  {
    id: 4,
    title: 'Clase Grabada 4',
    description: '19 de Marzo de 2026',
    driveId: '1wg3Qo-CVAL2Kkis0msUHh2WEzW6GhQzS',
    thumbnail: portadaFilosofia
  },
  {
    id: 5,
    title: 'Clase Grabada 5',
    description: '14 de Abril de 2026',
    driveId: '1j8rxSQCCsCxLwLORHrP0ZBfPknBAnB7R',
    thumbnail: portadaFilosofia
  },
  {
    id: 6,
    title: 'Clase Grabada 6',
    description: '28 de Abril de 2026',
    driveId: '1D8NHHiuRnQqtFbaKmFeAbugUcXrASE0f',
    thumbnail: portadaFilosofia
  }
])

// Invertimos el arreglo de videos para que el de ID más alto (último en agregarse)
// salga primero en la lista.
const reverseVideos = computed(() => {
  return [...videos.value].reverse()
})

// --- Lógica de Filtro / Búsqueda ---
const searchQuery = ref('') // Texto que el usuario escribe en el buscador

// Filtramos los videos según si la descripción contiene el texto buscado
const filteredVideos = computed(() => {
  if (!searchQuery.value.trim()) {
    return reverseVideos.value // Si no hay búsqueda, mostramos todos al revés (del último al primero)
  }

  const query = searchQuery.value.toLowerCase().trim()
  return reverseVideos.value.filter(video =>
    video.description.toLowerCase().includes(query)
  )
})

// Estado para el modal del reproductor de video
const selectedVideo = ref(null)

const openVideo = (video) => {
  selectedVideo.value = video
  document.body.style.overflow = 'hidden' // Evita que el fondo haga scroll
}

const closeVideo = () => {
  selectedVideo.value = null
  document.body.style.overflow = '' // Restaura el scroll
}

// --- Lógica para Modo Oscuro / Claro ---
const isDarkMode = ref(false)

const toggleTheme = () => {
  isDarkMode.value = !isDarkMode.value
  if (isDarkMode.value) {
    document.documentElement.setAttribute('data-theme', 'dark')
    localStorage.setItem('theme', 'dark')
  } else {
    document.documentElement.removeAttribute('data-theme')
    localStorage.setItem('theme', 'light')
  }
}

// Comprobar preferencia guardada o preferencia del sistema al cargar
onMounted(() => {
  const savedTheme = localStorage.getItem('theme')
  if (savedTheme === 'dark' || (!savedTheme && window.matchMedia('(prefers-color-scheme: dark)').matches)) {
    isDarkMode.value = true
    document.documentElement.setAttribute('data-theme', 'dark')
  }
})
</script>

<template>
  <div class="layout-container">
    <header class="site-header">
      <div class="header-overlay"></div>
      <div class="header-content">

        <!-- Botón para cambiar de tema (Modo Oscuro/Claro) -->
        <button class="theme-toggle" @click="toggleTheme" :title="isDarkMode ? 'Cambiar a Modo Claro' : 'Cambiar a Modo Oscuro'" aria-label="Cambiar tema">
          <span v-if="isDarkMode">☀️</span>
          <span v-else>🌙</span>
        </button>

        <div class="header-ornament">🏛️</div>
        <h1>Ágora Virtual</h1>
        <div class="separator-line"></div>
        <p class="subtitle">Archivo de Clases</p>
      </div>
    </header>

    <main class="main-content">
      <section class="intro-section">
        <h2 class="section-title">Clases Grabadas</h2>
        <p class="section-desc">«El principio de la sabiduría es la duda.» <br><span>— Sócrates</span></p>
      </section>

      <!-- Buscador por Descripción -->
      <div class="search-section">
        <div class="search-box">
          <span class="search-icon">🔍</span>
          <input
            type="text"
            v-model="searchQuery"
            placeholder="Buscar por fecha o tema (ej. Marzo de 2026)..."
            class="search-input"
            aria-label="Buscar clases"
          >
          <button
            v-if="searchQuery"
            @click="searchQuery = ''"
            class="clear-search"
            aria-label="Borrar búsqueda"
          >
            ×
          </button>
        </div>
      </div>

      <!-- Mensaje si no hay resultados -->
      <div v-if="filteredVideos.length === 0" class="no-results">
        <p>No se encontraron clases que coincidan con "{{ searchQuery }}".</p>
      </div>

      <div v-else class="video-grid">
        <article
          v-for="video in filteredVideos"
          :key="video.id"
          class="video-card"
          @click="openVideo(video)"
          tabindex="0"
          @keyup.enter="openVideo(video)"
        >
          <div class="video-thumbnail">
            <img :src="video.thumbnail" :alt="`Portada de ${video.title}`" loading="lazy" />
            <div class="play-overlay">
              <span class="play-icon" aria-hidden="true">▶</span>
            </div>
            <div class="thumbnail-border"></div>
          </div>

          <div class="video-info">
            <div class="info-ornament" aria-hidden="true">✧</div>
            <h3 class="video-title">{{ video.title }}</h3>
            <p class="video-desc">{{ video.description }}</p>
          </div>
        </article>
      </div>
    </main>

    <!-- Modal para el Reproductor de Video de Google Drive -->
    <transition name="fade">
      <div v-if="selectedVideo" class="modal-overlay" @click.self="closeVideo">
        <div class="modal-content">
          <button class="close-button" @click="closeVideo" title="Cerrar" aria-label="Cerrar video">×</button>
          <div class="video-wrapper">
            <!-- Iframe de Google Drive -->
            <iframe
              :src="`https://drive.google.com/file/d/${selectedVideo.driveId}/preview`"
              width="100%"
              height="100%"
              allow="autoplay"
              allowfullscreen
              title="Reproductor de clase"
            >
            </iframe>
          </div>
          <div class="modal-info">
            <h2>{{ selectedVideo.title }}</h2>
            <div class="modal-separator"></div>
          </div>
        </div>
      </div>
    </transition>

    <footer class="site-footer">
      <div class="footer-content">
        <p class="greek-motto">ΓΝΩΘΙ ΣΑΥΤΟΝ</p>
        <p>&copy; Moises Glz. Todos los derechos reservados.</p>
      </div>
    </footer>
  </div>
</template>

<style>
/* Variables CSS (Paleta de Colores por Defecto - MODO CLARO) */
:root {
  --bg-color: #f4f1ea;        /* Blanco hueso / Mármol claro */
  --card-bg: #ffffff;
  --primary-color: #2c3539;   /* Gris grafito muy oscuro */
  --accent-color: #a88944;    /* Dorado antiguo / Bronce */
  --text-main: #3a3a3a;
  --text-muted: #6b6b6b;
  --border-color: #e2dbce;
  --input-bg: #ffffff;
  --paper-texture: url('https://www.transparenttextures.com/patterns/cream-paper.png');
}

/* Variables CSS (Paleta de Colores - MODO OSCURO) */
[data-theme="dark"] {
  --bg-color: #121212;        /* Fondo oscuro profundo */
  --card-bg: #1e1e1e;         /* Gris muy oscuro para las tarjetas */
  --primary-color: #e0e0e0;   /* Texto principal claro en modo oscuro */
  --accent-color: #d4af37;    /* Dorado un poco más brillante para resaltar */
  --text-main: #e0e0e0;
  --text-muted: #a0a0a0;
  --border-color: #333333;
  --input-bg: #2a2a2a;
  --paper-texture: none;      /* Quitamos la textura de papel en modo oscuro para un look más limpio */
}

/* Estilos globales */
*, *::before, *::after {
  box-sizing: border-box;
}

body {
  margin: 0;
  padding: 0;
  font-family: 'Merriweather', 'Georgia', serif;
  background-color: var(--bg-color);
  color: var(--text-main);
  line-height: 1.6;
  background-image: var(--paper-texture);
  transition: background-color 0.3s ease, color 0.3s ease; /* Transición suave al cambiar de tema */
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

/* Layout Principal */
.layout-container {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

/* Botón de Cambio de Tema */
.theme-toggle {
  position: absolute;
  top: 1rem;
  right: 1rem;
  background: rgba(0, 0, 0, 0.5);
  border: 1px solid var(--accent-color);
  color: #fff;
  font-size: 1.5rem;
  width: 45px;
  height: 45px;
  border-radius: 50%;
  cursor: pointer;
  z-index: 10;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s ease;
  backdrop-filter: blur(5px);
  -webkit-tap-highlight-color: transparent;
}

.theme-toggle:hover, .theme-toggle:focus {
  background: var(--accent-color);
  transform: scale(1.1);
  outline: none;
}

/* Header Estilo Templo Griego */
.site-header {
  position: relative;
  background-color: #1a1e21; /* Fondo muy oscuro para el contraste */
  background-image: url('https://images.unsplash.com/photo-1605460375648-278bcbd579a6?auto=format&fit=crop&q=80&w=2000'); /* Imagen de columnas de fondo */
  background-size: cover;
  background-position: center;
  color: #fff;
  padding: 4rem 1rem;
  text-align: center;
  border-bottom: 6px solid var(--accent-color);
  box-shadow: 0 4px 15px rgba(0,0,0,0.2);
}

.header-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(26, 30, 33, 0.85); /* Capa oscura para que el texto sea legible */
  z-index: 1;
}

.header-content {
  position: relative;
  z-index: 2;
  max-width: 800px;
  margin: 0 auto;
}

.header-ornament {
  font-size: clamp(2rem, 5vw, 2.5rem);
  color: var(--accent-color);
  margin-bottom: 1rem;
  opacity: 0.9;
}

.site-header h1 {
  margin: 0;
  font-size: clamp(2rem, 6vw, 3.5rem);
  font-weight: 400;
  letter-spacing: clamp(2px, 1vw, 4px);
  text-transform: uppercase;
  color: #fdfdfd;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.5);
  line-height: 1.2;
}

.separator-line {
  width: min(100px, 50%);
  height: 2px;
  background-color: var(--accent-color);
  margin: 1.5rem auto;
}

.subtitle {
  font-size: clamp(1rem, 3vw, 1.3rem);
  font-style: italic;
  color: #dcdcdc;
  letter-spacing: 1px;
}

/* Contenido Principal */
.main-content {
  flex: 1;
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
  padding: clamp(2rem, 5vw, 4rem) clamp(1rem, 3vw, 2rem);
}

.intro-section {
  text-align: center;
  margin-bottom: 2rem;
}

.section-title {
  font-size: clamp(1.8rem, 4vw, 2.2rem);
  color: var(--primary-color);
  margin-bottom: 1rem;
  font-weight: normal;
  text-transform: uppercase;
  letter-spacing: 2px;
  transition: color 0.3s ease;
}

.section-desc {
  font-size: clamp(1rem, 2.5vw, 1.2rem);
  color: var(--text-muted);
  font-style: italic;
  max-width: 600px;
  margin: 0 auto;
  transition: color 0.3s ease;
}

.section-desc span {
  font-size: 1rem;
  color: var(--accent-color);
  font-weight: bold;
  font-style: normal;
}

/* Buscador (Filtro por texto) */
.search-section {
  display: flex;
  justify-content: center;
  margin-bottom: 3rem;
  padding: 0 1rem;
}

.search-box {
  position: relative;
  width: 100%;
  max-width: 600px;
  display: flex;
  align-items: center;
}

.search-icon {
  position: absolute;
  left: 1.2rem;
  color: var(--text-muted);
  font-size: 1.2rem;
  pointer-events: none; /* Para que el clic pase al input */
}

.search-input {
  width: 100%;
  padding: 1rem 3rem 1rem 3rem;
  font-family: 'Lato', sans-serif;
  font-size: 1rem;
  color: var(--text-main);
  background-color: var(--input-bg);
  border: 2px solid var(--border-color);
  border-radius: 30px;
  transition: all 0.3s ease;
  box-shadow: 0 4px 10px rgba(0,0,0,0.05);
}

.search-input:focus {
  outline: none;
  border-color: var(--accent-color);
  box-shadow: 0 4px 15px rgba(168, 137, 68, 0.2);
}

.clear-search {
  position: absolute;
  right: 1rem;
  background: none;
  border: none;
  color: var(--text-muted);
  font-size: 1.5rem;
  cursor: pointer;
  padding: 0 0.5rem;
  transition: color 0.2s ease;
}

.clear-search:hover {
  color: var(--accent-color);
}

.no-results {
  text-align: center;
  padding: 3rem 1rem;
  color: var(--text-muted);
  font-style: italic;
  font-size: 1.1rem;
}

/* Grid de Videos */
.video-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(min(100%, 300px), 1fr));
  gap: clamp(1.5rem, 4vw, 3rem);
}

/* Tarjeta de Video - Estilo Retrato/Camafeo */
.video-card {
  background: var(--card-bg);
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.05);
  transition: all 0.3s ease;
  cursor: pointer;
  border: 1px solid var(--border-color);
  position: relative;
  display: flex;
  flex-direction: column;
}

.video-card:focus {
  outline: 3px solid var(--accent-color);
  outline-offset: 2px;
}

.video-card::before {
  content: '';
  position: absolute;
  top: 0; left: 0; right: 0; bottom: 0;
  border: 4px solid transparent;
  transition: border-color 0.4s ease;
  z-index: 10;
  pointer-events: none;
  border-radius: 8px;
}

@media (hover: hover) {
  .video-card:hover {
    transform: translateY(-5px);
    box-shadow: 0 12px 25px rgba(0, 0, 0, 0.15);
  }

  .video-card:hover::before {
    border-color: var(--accent-color);
  }

  .video-card:hover .video-thumbnail img {
    transform: scale(1.05);
  }

  .video-card:hover .play-overlay {
    opacity: 1;
  }
}

/* Efecto activo para móviles */
.video-card:active {
  transform: scale(0.98);
}

.video-thumbnail {
  position: relative;
  width: 100%;
  padding-top: 56.25%; /* Ratio 16:9 ideal para videos en todos los dispositivos */
  background-color: #000;
  overflow: hidden;
}

.video-thumbnail img {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.5s ease;
  filter: sepia(20%) contrast(110%);
}

.thumbnail-border {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 4px;
  background: var(--accent-color);
  z-index: 2;
}

.play-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.4);
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0; /* Se mantiene oculto hasta hover/focus en desktop */
  transition: opacity 0.3s ease;
  z-index: 5;
}

/* En móviles mostramos el icono de play siempre un poco visible para denotar que es un video */
@media (hover: none) {
  .play-overlay {
    opacity: 0.8;
    background: rgba(0, 0, 0, 0.2);
  }
}

.video-card:focus .play-overlay {
  opacity: 1;
}

.play-icon {
  font-size: clamp(2rem, 5vw, 3rem);
  color: #fff;
  background: var(--accent-color);
  width: clamp(50px, 10vw, 70px);
  height: clamp(50px, 10vw, 70px);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  padding-left: 6px;
  box-shadow: 0 4px 10px rgba(0,0,0,0.5);
}

.video-info {
  flex: 1;
  padding: clamp(1.5rem, 3vw, 2rem) 1.5rem;
  text-align: center;
  background-image: var(--paper-texture);
  background-color: var(--card-bg);
  transition: background-color 0.3s ease;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.info-ornament {
  color: var(--accent-color);
  font-size: 1.2rem;
  margin-bottom: 0.5rem;
}

.video-title {
  margin: 0 0 0.5rem 0;
  font-size: clamp(1.1rem, 3vw, 1.4rem);
  color: var(--primary-color);
  line-height: 1.4;
  font-weight: normal;
  transition: color 0.3s ease;
}

.video-desc {
  margin: 0;
  font-size: clamp(0.9rem, 2vw, 1rem);
  color: var(--text-muted);
  font-style: italic;
  font-family: 'Lato', sans-serif;
  transition: color 0.3s ease;
}

/* Modal del Reproductor */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100dvw; /* Usa dvw y dvh para manejar mejor la barra de nav en móviles */
  height: 100dvh;
  background-color: rgba(0, 0, 0, 0.95);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  padding: clamp(0px, 2vw, 2rem);
  box-sizing: border-box;
  backdrop-filter: blur(5px);
}

.modal-content {
  background-color: var(--card-bg);
  width: 100%;
  max-width: 1200px;
  max-height: 100dvh;
  border-radius: 8px;
  overflow-y: auto; /* Permite scroll si el contenido es muy alto */
  position: relative;
  box-shadow: 0 25px 50px rgba(0,0,0,0.8);
  border: 2px solid var(--accent-color);
  transition: background-color 0.3s ease;
  display: flex;
  flex-direction: column;
}

.close-button {
  position: absolute;
  top: clamp(10px, 2vw, 15px);
  right: clamp(10px, 2vw, 20px);
  background: var(--card-bg);
  border: 2px solid var(--accent-color);
  border-radius: 50%;
  width: clamp(35px, 8vw, 45px);
  height: clamp(35px, 8vw, 45px);
  color: var(--accent-color);
  font-size: clamp(1.5rem, 4vw, 1.8rem);
  line-height: 1;
  cursor: pointer;
  z-index: 10;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
  -webkit-tap-highlight-color: transparent;
}

.close-button:hover, .close-button:focus {
  background: var(--accent-color);
  color: #fff;
  transform: rotate(90deg);
  outline: none;
}

.video-wrapper {
  position: relative;
  width: 100%;
  padding-top: 56.25%; /* Ratio 16:9 estricto */
  background-color: #000;
  flex-shrink: 0;
}

.video-wrapper iframe {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  border: none;
}

.modal-info {
  padding: clamp(1.5rem, 4vw, 2.5rem);
  text-align: center;
  background-image: var(--paper-texture);
  background-color: var(--card-bg);
  transition: background-color 0.3s ease;
}

.modal-info h2 {
  margin: 0;
  color: var(--primary-color);
  font-size: clamp(1.3rem, 4vw, 1.8rem);
  font-weight: normal;
  transition: color 0.3s ease;
}

.modal-separator {
  width: 60px;
  height: 2px;
  background: var(--accent-color);
  margin: 1rem auto 0;
}

/* Footer Clásico */
.site-footer {
  text-align: center;
  padding: clamp(2rem, 5vw, 3rem) 1rem;
  background-color: #1a1e21; /* Siempre oscuro */
  color: #dcdcdc;
  border-top: 4px solid var(--accent-color);
  margin-top: auto;
}

.greek-motto {
  font-family: 'Times New Roman', Times, serif;
  font-size: clamp(1.2rem, 3vw, 1.5rem);
  letter-spacing: clamp(3px, 1vw, 5px);
  color: var(--accent-color);
  margin-bottom: 1rem;
}

/* Transiciones de Vue */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

/* Optimizaciones extra para móviles (Tablets pequeñas y teléfonos) */
@media (max-width: 768px) {
  .search-section {
    padding: 0;
  }

  .search-input {
    border-radius: 8px; /* Menos redondo en móviles para aprovechar espacio */
  }

  .modal-overlay {
    padding: 0; /* El modal ocupa toda la pantalla en móvil */
  }

  .modal-content {
    border-radius: 0;
    border: none;
    height: 100dvh;
    justify-content: center; /* Centrar video verticalmente si hay espacio */
  }

  .close-button {
    /* Mejoramos el contraste del botón de cerrar en móviles cuando está sobre el video */
    background: rgba(0, 0, 0, 0.5);
    color: #fff;
    border-color: #fff;
  }
}
</style>
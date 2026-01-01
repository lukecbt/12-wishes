<script setup lang="ts">
import { ref, onMounted, computed } from 'vue'
import wishesData from './wishes.json'

interface Grape {
  id: number
  title: string
  defaultWish: string
  defaultSong: string
  x: number
  y: number
  isOpen: boolean
  wish: string
  songOfTheDay: string
}

const grapes = ref<Grape[]>([])
const activeWish = ref<number | null>(null)
const wishText = ref('')
const songName = ref('')

const generateRandomPositions = () => {
  const positions = [
    { x: 23, y: 8 }, { x: 62, y: 15 }, { x: 81, y: 22 },
    { x: 12, y: 28 }, { x: 38, y: 41 }, { x: 74, y: 33 },
    { x: 29, y: 52 }, { x: 67, y: 64 }, { x: 85, y: 58 },
    { x: 16, y: 71 }, { x: 41, y: 85 }, { x: 69, y: 79 }
  ]
  return wishesData.map((wish, index) => ({
    ...wish,
    x: positions[index].x,
    y: positions[index].y,
    isOpen: false,
    wish: '',
    songOfTheDay: ''
  }))
}

const loadFromStorage = () => {
  const saved = localStorage.getItem('12-wishes')
  if (saved) {
    const savedGrapes = JSON.parse(saved)
    grapes.value = wishesData.map((wish, index) => {
      const savedGrape = savedGrapes.find((g: Grape) => g.id === wish.id)
      const positions = [
        { x: 23, y: 8 }, { x: 62, y: 15 }, { x: 81, y: 22 },
        { x: 12, y: 28 }, { x: 38, y: 41 }, { x: 74, y: 33 },
        { x: 29, y: 52 }, { x: 67, y: 64 }, { x: 85, y: 58 },
        { x: 16, y: 71 }, { x: 41, y: 85 }, { x: 69, y: 79 }
      ]
      return savedGrape ? { ...wish, ...savedGrape, x: positions[index].x, y: positions[index].y } : { ...wish, x: positions[index].x, y: positions[index].y, isOpen: false, wish: '', songOfTheDay: '' }
    })
  } else {
    grapes.value = generateRandomPositions()
  }
}

const saveToStorage = () => {
  localStorage.setItem('12-wishes', JSON.stringify(grapes.value))
}

const openGrape = (id: number) => {
  const grape = grapes.value.find(g => g.id === id)
  if (grape) {
    grape.isOpen = true
    activeWish.value = id
    wishText.value = grape.wish || grape.defaultWish
    songName.value = grape.songOfTheDay || grape.defaultSong
    saveToStorage()
  }
}

const closeWish = () => {
  if (activeWish.value) {
    const grape = grapes.value.find(g => g.id === activeWish.value)
    if (grape) {
      grape.wish = wishText.value
      grape.songOfTheDay = songName.value
      saveToStorage()
    }
  }
  activeWish.value = null
  wishText.value = ''
  songName.value = ''
}

const activeGrape = computed(() => 
  activeWish.value ? grapes.value.find(g => g.id === activeWish.value) : null
)

onMounted(() => {
  loadFromStorage()
})
</script>

<template>
  <div class="app">
    <div class="background">
      <div class="firework" v-for="n in 6" :key="n" :style="{ left: Math.random() * 100 + '%', top: Math.random() * 100 + '%', animationDelay: Math.random() * 3 + 's' }"></div>
      <div class="star" v-for="n in 20" :key="n" :style="{ left: Math.random() * 100 + '%', top: Math.random() * 100 + '%', animationDelay: Math.random() * 2 + 's' }"></div>
    </div>
    
    <h1 class="title">🎊 12 New Year Wishes 🎊</h1>
    
    <div class="grapes-container">
      <div 
        v-for="grape in grapes" 
        :key="grape.id"
        class="grape"
        :class="{ open: grape.isOpen }"
        :style="{ left: grape.x + '%', top: grape.y + '%' }"
        @click="openGrape(grape.id)"
      >
        {{ grape.isOpen ? '✨' : '🍇' }}
      </div>
    </div>
    
    <div v-if="activeWish" class="wish-panel" @click.self="closeWish">
      <div class="wish-content">
        <div class="wish-header">
          <h2>{{ activeGrape?.title }}</h2>
          <button @click="closeWish" class="close-btn">×</button>
        </div>
        <div class="wish-text">{{ wishText }}</div>
        <div class="song-section">
          <label class="song-label">🎵 Song of the Day</label>
          <div class="song-display">{{ songName }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.app {
  min-height: 100vh;
  background: linear-gradient(135deg, #1a1a2e 0%, #16213e 50%, #0f3460 100%);
  position: relative;
  overflow: hidden;
  padding: 20px;
  box-sizing: border-box;
}

.background {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
}

.firework {
  position: absolute;
  width: 4px;
  height: 4px;
  background: #ffd700;
  border-radius: 50%;
  animation: sparkle 3s infinite;
}

.star {
  position: absolute;
  width: 2px;
  height: 2px;
  background: #fff;
  border-radius: 50%;
  animation: twinkle 2s infinite;
}

@keyframes sparkle {
  0%, 100% { opacity: 0; transform: scale(0); }
  50% { opacity: 1; transform: scale(1); }
}

@keyframes twinkle {
  0%, 100% { opacity: 0.3; }
  50% { opacity: 1; }
}

.title {
  text-align: center;
  color: #ffd700;
  font-size: 1.7rem;
  margin-bottom: 2rem;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.5);
  position: relative;
  z-index: 1;
}

.grapes-container {
  position: relative;
  height: 70vh;
  width: 100%;
}

.grape {
  position: absolute;
  font-size: 2rem;
  cursor: pointer;
  transition: transform 0.3s ease;
  user-select: none;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  padding: 20px;
}

.grape:hover {
  transform: scale(1.2);
}

.grape.open {
  animation: bounce 0.6s ease;
}

@keyframes bounce {
  0%, 20%, 50%, 80%, 100% { transform: translateY(0); }
  40% { transform: translateY(-10px); }
  60% { transform: translateY(-5px); }
}

.wish-panel {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0,0,0,0.8);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  animation: fadeIn 0.3s ease;
}

.wish-content {
  background: linear-gradient(135deg, #2d1b69 0%, #11998e 100%);
  width: 90%;
  max-width: 500px;
  max-height: 80vh;
  border-radius: 20px;
  padding: 20px;
  box-shadow: 0 10px 30px rgba(0,0,0,0.3);
  animation: scaleIn 0.3s ease;
}

.wish-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.wish-header h2 {
  color: #ffd700;
  margin: 0;
  font-size: 1.3rem;
}

.close-btn {
  background: none;
  border: none;
  color: #ffd700;
  font-size: 2rem;
  cursor: pointer;
  padding: 0;
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  transition: background 0.3s ease;
}

.close-btn:hover {
  background: rgba(255,215,0,0.2);
}

.wish-text {
  color: #fff;
  font-size: 1.1rem;
  line-height: 1.6;
  padding: 15px;
  background: rgba(255,255,255,0.1);
  border-radius: 10px;
  border: 2px solid rgba(255,215,0,0.3);
  min-height: 100px;
  display: flex;
  align-items: center;
  text-align: center;
  margin-bottom: 15px;
}

.song-section {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.song-label {
  color: #ffd700;
  font-size: 1rem;
  font-weight: bold;
}

.song-display {
  background: rgba(255,255,255,0.1);
  border: 2px solid rgba(255,215,0,0.3);
  border-radius: 8px;
  padding: 10px;
  color: #fff;
  font-size: 1rem;
  min-height: 20px;
  display: flex;
  align-items: center;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

@keyframes scaleIn {
  from {
    transform: scale(0.8);
    opacity: 0;
  }
  to {
    transform: scale(1);
    opacity: 1;
  }
}

@media (min-width: 768px) {
  .title {
    font-size: 2.2rem;
  }
  
  .grape {
    font-size: 2.5rem;
  }
}
</style>

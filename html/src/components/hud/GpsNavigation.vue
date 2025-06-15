<template>
  <div v-if="showGps" class="gps-navigation">
    <div class="gps-container">
      <!-- Current Direction -->
      <div class="direction-box current">
        <div class="nav-arrow" :style="{ transform: `rotate(${arrowRotation}deg)` }">
          <v-icon color="orange" size="large">mdi-arrow-up-bold</v-icon>
        </div>
        <div class="distance-value">{{ formatDistance(distance) }}</div>
      </div>

      <!-- Next Direction Preview -->
      <div v-if="nextTurn" class="direction-box next">
        <div class="next-arrow" :style="{ transform: `rotate(${nextTurn.rotation}deg)` }">
          <v-icon color="orange" size="small">mdi-arrow-up-bold</v-icon>
        </div>
        <div class="distance-preview">{{ formatDistance(nextTurn.distance) }}</div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, computed } from 'vue'
import type { GpsData } from '@/store/types'
import { useGlobalStore } from "@/store/global"

interface TurnInfo {
  rotation: number;
  distance: number;
}
// Development testing data
const isDev = import.meta.env.DEV
const devData: GpsData = {
  isWaypointSet: true,
  distance: 150,
  rotation: 45,
  nextTurn: {
    distance: 300,
    rotation: 90
  }
}

// Reactive state
const showGps = ref(isDev)
const distance = ref(isDev ? devData.distance : 0)
const arrowRotation = ref(isDev ? devData.rotation : 0)
const nextTurn = ref(isDev ? devData.nextTurn : null)

// Listen for GPS updates from client script
const handleGpsUpdate = (event: MessageEvent) => {
  if (event.data.type === 'updateGps') {
    const data = event.data
    showGps.value = data.isWaypointSet
    distance.value = data.distance
    arrowRotation.value = data.rotation

    // Handle next turn data if available
    if (data.nextTurn) {
      nextTurn.value = {
        rotation: data.nextTurn.rotation,
        distance: data.nextTurn.distance
      }
    } else {
      nextTurn.value = null
    }
  }
}

onMounted(() => {
  window.addEventListener('message', handleGpsUpdate)
})

onUnmounted(() => {
  window.removeEventListener('message', handleGpsUpdate)
})

const formatDistance = (meters: number) => {
  if (!meters) return '0m'
  return meters > 1000 
    ? `${(meters / 1000).toFixed(1)}km` 
    : `${Math.round(meters)}m`
}
</script>

<style scoped lang="scss">
@use '@/styles/variables' as v;

$orange-glow: rgba(255, 165, 0, 0.15);
$orange-border: rgba(255, 165, 0, 0.2);

.gps-navigation {
  position: fixed;
  left: 50%;
  top: 15px;
  transform: translateX(-50%);
  z-index: 100;
}

.gps-container {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  background: rgba(0, 0, 0, 0.85);
  border: 1px solid rgba(255, 255, 255, 0.1);
  padding: 0.5rem 0.75rem;
  border-radius: 4px;
  box-shadow: 
    0 4px 8px rgba(0, 0, 0, 0.4),
    inset 0 1px rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(8px);
}

.direction-box {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.3rem;
  padding: 0.25rem;
}

.nav-arrow {
  width: 40px;
  height: 40px;
  background: rgba(255, 165, 0, 0.1);
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
  transform-origin: center;
  transition: all 0.2s ease;
  box-shadow: 
    inset 0 0 0 1px rgba(255, 165, 0, 0.15),
    0 0 10px rgba(255, 165, 0, 0.1);

  .v-icon {
    opacity: 0.9;
    filter: drop-shadow(0 0 4px rgba(255, 165, 0, 0.3));
  }
}

.direction-box.next {
  border-left: 1px solid rgba(255, 255, 255, 0.1);
  padding-left: 0.75rem;
  margin-left: 0.25rem;

  .next-arrow {
    width: 28px;
    height: 28px;
    background: rgba(255, 165, 0, 0.08);
    border-radius: 3px;
    display: flex;
    align-items: center;
    justify-content: center;
    transform-origin: center;
    transition: transform 0.2s ease;
    box-shadow: inset 0 0 0 1px rgba(255, 165, 0, 0.1);

    .v-icon {
      opacity: 0.7;
      filter: drop-shadow(0 0 3px rgba(255, 165, 0, 0.2));
    }
  }
}

.distance-value,
.distance-preview {
  font-family: 'JetBrains Mono', monospace;
  font-weight: 500;
  color: rgba(255, 255, 255, 0.95);
  letter-spacing: 0.05em;
  text-align: center;
  font-size: 0.75rem;
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.3);
}

.distance-preview {
  font-size: 0.65rem;
  opacity: 0.7;
}
</style>

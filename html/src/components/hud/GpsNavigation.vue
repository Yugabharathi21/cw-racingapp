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
  width: 75%;
  align-items: center;
  gap: 3.75rem;
  padding: 1rem 0.75rem;
  
}

.direction-box {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.3rem;
}

.nav-arrow {
  width: 50px;
  height: 50px;
  background: $orange-glow;
  border-radius: 6px;
  display: flex;
  align-items: center;
  justify-content: center;
  transform-origin: center;
  transition: all 0.3s cubic-bezier(0.4, 0.0, 0.2, 1);
  box-shadow: 
    inset 0 0 0 1px $orange-border,
    0 0 15px rgba(255, 165, 0, 0.1);

  .v-icon {
    filter: drop-shadow(0 0 8px rgba(255, 165, 0, 0.4));
  }

  &:after {
    content: '';
    position: absolute;
    inset: -1px;
    border-radius: inherit;
    background: linear-gradient(45deg, rgba(255, 165, 0, 0.2) 0%, rgba(255, 165, 0, 0.1) 50%, rgba(255, 165, 0, 0.05) 100%);
    mask: 
      linear-gradient(black, black) content-box,
      linear-gradient(black, black);
    mask-composite: exclude;
    pointer-events: none;
  }

  &:hover {
    background: rgba(255, 165, 0, 0.2);
    box-shadow: 
      inset 0 0 0 1px rgba(255, 165, 0, 0.3),
      0 0 20px rgba(255, 165, 0, 0.15);
  }
}

.direction-box.next {
  border-left: 1px solid rgba(255, 255, 255, 0.05);
  padding-left: 0.75rem;

  .next-arrow {
    width: 30px;
    height: 30px;
    background: rgba(255, 165, 0, 0.1);
    border-radius: 4px;
    display: flex;
    align-items: center;
    justify-content: center;
    transform-origin: center;
    transition: transform 0.3s cubic-bezier(0.4, 0.0, 0.2, 1);
    box-shadow: inset 0 0 0 1px rgba(255, 165, 0, 0.1);

    .v-icon {
      opacity: 0.8;
      filter: drop-shadow(0 0 4px rgba(255, 165, 0, 0.3));
    }

    &:hover {
      background: rgba(255, 165, 0, 0.15);
      box-shadow: inset 0 0 0 1px rgba(255, 165, 0, 0.2);
    }
  }
}

.distance-value,
.distance-preview {
  font-family: 'JetBrains Mono', monospace;
  font-weight: 500;
  color: rgba(255, 255, 255, 0.9);
  letter-spacing: 0.05em;
  text-align: center;
  font-size: 0.8rem;
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.5);
}

.distance-preview {
  font-size: 0.7rem;
  opacity: 0.7;
}
</style>

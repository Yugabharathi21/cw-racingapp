<template>
  <div class="race">
    <div class="race__container">
      <!-- Left Side: Position List -->
      <div class="race__positions">
        <RacerList :racers="globalStore.activeRace.positions" />
      </div>

      <!-- Right Side: Race Info -->
      <div class="race__info">
        <!-- Track Info -->
        <div class="info-block track-info">
          <div class="track-icon">
            <v-icon>mdi-flag-checkered</v-icon>
          </div>
          <div class="track-name">{{ globalStore.activeRace.raceName }}</div>
        </div>        <!-- Race Stats -->
        <div class="info-block stats">
          <!-- Race Timings -->
          <RaceTimings 
            :time="globalStore.activeRace.time || 0"
            :total-time="globalStore.activeRace.totalTime || 0"
            :best-lap="globalStore.activeRace.bestLap || 0"
          />
          <div class="stats-grid">
            <!-- Position -->
            <div class="stat-module position" v-if="globalStore.activeRace.totalRacers && globalStore.activeRace.totalRacers !== 1">
              <div class="stat-icon">
                <v-icon>mdi-trophy-outline</v-icon>
              </div>
              <div class="stat-content">
                <TransitionGroup name="number-slide" tag="div" class="position-value">
                  <span :key="'pos' + globalStore.activeRace.position" class="current">
                    {{ globalStore.activeRace.position }}
                  </span>
                </TransitionGroup>
                <span class="stat-divider">/</span>
                <span class="total">{{ globalStore.activeRace.totalRacers }}</span>
              </div>
            </div>

            <!-- Checkpoints -->
            <div class="stat-module checkpoints">
              <div class="stat-icon">
                <v-icon>mdi-map-marker-path</v-icon>
              </div>
              <div class="stat-content">
                {{ globalStore.activeRace.currentCheckpoint }}/{{ globalStore.activeRace.totalCheckpoints }}
              </div>
            </div>

            <!-- Lap Counter -->
            <div class="stat-module laps">
              <div class="stat-icon">
                <v-icon>mdi-refresh</v-icon>
              </div>
              <div class="stat-content">
                <TransitionGroup name="number-slide" tag="div" class="lap-value">
                  <span :key="'lap' + globalStore.activeRace.currentLap">{{ lapText }}</span>
                </TransitionGroup>
              </div>
            </div>          </div>
        </div>

        <!-- Ghost Mode Indicator -->
        <div v-if="globalStore.activeRace.ghosted" class="ghost-indicator">
          <v-icon size="small">mdi-ghost</v-icon>
          <span>GHOST</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { useGlobalStore } from "@/store/global";
import { computed } from "vue";
import { translate } from "@/helpers/translate";
import RacerList from "./RacerList.vue";
import RaceTimings from "./RaceTimings.vue";

const globalStore = useGlobalStore();

const lapText = computed(() => {
  if (globalStore.activeRace.totalLaps === 0) return "SPRINT";
  else if (globalStore.activeRace.totalLaps === -1)
    return `${globalStore.activeRace.currentLap}/-`;
  return `${globalStore.activeRace.currentLap}/${globalStore.activeRace.totalLaps}`;
});
</script>

<style scoped lang="scss">
@use '@/styles/variables' as v;

.race {
  position: fixed;
  top: 2vh;
  left: 0;
  right: 0;
  z-index: 100;
  font-family: 'Rajdhani', sans-serif;
}

.race__container {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  padding: 0.5rem 1rem;
  gap: 1rem;
}

.race__positions {
  flex: 0 1 auto;
}

.race__info {
  display: flex;
  flex-direction: column;
  gap: 0.35rem;
  align-items: flex-end;
}

.info-block {
  background: rgba(0, 0, 0, 0.85);
  border: 1px solid rgba(255, 255, 255, 0.1);
  box-shadow: 
    0 4px 8px rgba(0, 0, 0, 0.4),
    inset 0 1px rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(8px);
  border-radius: 4px;
  padding: 0.4rem 0.6rem;
  min-width: 160px;
  transition: all 0.2s ease;

  &:hover {
    background: rgba(0, 0, 0, 0.9);
  }
}

.track-info {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.3rem 0.6rem;

  .track-icon {
    color: v.$primary-color-light;
    opacity: 0.9;
    font-size: 0.9rem;
  }

  .track-name {
    color: v.$text-color;
    font-size: 0.85rem;
    font-weight: 600;
    text-transform: uppercase;
    letter-spacing: 0.1em;
    opacity: 0.95;
  }
}

.stats {
  padding: 0.5rem;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(80px, 1fr));
  gap: 0.4rem;
}

.stat-module {
  display: flex;
  align-items: center;
  gap: 0.3rem;
  padding: 0.2rem 0.4rem;
  background: rgba(0, 0, 0, 0.95);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 3px;
  transition: all 0.2s ease;

  &:hover {
    background: rgba(0, 0, 0, 1);
    border-color: rgba(255, 255, 255, 0.12);
  }

  .stat-icon {
    color: v.$primary-color-light;
    opacity: 0.9;
    font-size: 0.9rem;
  }

  .stat-content {
    display: flex;
    align-items: center;
    gap: 0.2rem;
    color: v.$text-color;
    font-weight: 600;
    font-size: 0.9rem;
  }

  &.position {
    grid-column: span 2;
    background: rgba(0, 0, 0, 0.95);
    border: 1px solid rgba(255, 255, 255, 0.1);
    
    .stat-content {
      font-size: 1.4rem;
      color: v.$primary-color-light;
    }
    
    .stat-divider {
      opacity: 0.5;
      margin: 0 0.1rem;
      color: v.$text-color;
    }
    
    .total {
      font-size: 0.8em;
      opacity: 0.7;
      color: v.$text-color;
    }
  }

  &.checkpoints {
    font-family: 'JetBrains Mono', monospace;
    font-size: 0.85rem;
  }

  &.laps .stat-content {
    color: v.$primary-color-light;
    font-weight: 700;
  }
}

.ghost-indicator {
  display: flex;
  align-items: center;
  gap: 0.3rem;
  background: rgba(0, 0, 0, 0.85);
  border: 1px solid rgba(255, 255, 255, 0.1);
  color: v.$text-color;
  font-size: 0.7rem;
  padding: 0.15rem 0.4rem;
  border-radius: 3px;
  letter-spacing: 0.15em;
  opacity: 0.9;
  backdrop-filter: blur(8px);
  transition: all 0.2s ease;
  
  &:hover {
    background: rgba(0, 0, 0, 0.95);
    opacity: 1;
  }
  
  .v-icon {
    font-size: 0.8rem;
    opacity: 0.9;
  }
}

// Animations
.number-slide-enter-active,
.number-slide-leave-active {
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
  position: absolute;
}

.number-slide-enter-from {
  opacity: 0;
  transform: translateY(-100%) scale(0.95);
}

.number-slide-leave-to {
  opacity: 0;
  transform: translateY(100%) scale(0.95);
}

.position-value,
.lap-value {
  position: relative;
  display: inline-block;
  min-width: 1.2em;
  text-align: center;
}
</style>

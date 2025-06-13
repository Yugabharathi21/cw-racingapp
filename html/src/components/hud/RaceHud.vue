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
        </div>

        <!-- Race Stats -->
        <div class="info-block stats">
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
            </div>
          </div>

          <!-- Times -->
          <div class="times-block">
            <div class="time-module current">
              <div class="time-icon">
                <v-icon>mdi-clock-outline</v-icon>
              </div>
              <span class="time-value">{{ msToHMS(globalStore.activeRace.time) }}</span>
            </div>
            <div class="time-module total">
              <div class="time-icon">
                <v-icon>mdi-timer-outline</v-icon>
              </div>
              <span class="time-value">{{ msToHMS(globalStore.activeRace.totalTime) }}</span>
            </div>
            <div class="time-module best">
              <div class="time-icon">
                <v-icon>mdi-star-outline</v-icon>
              </div>
              <span class="time-value">{{ msToHMS(globalStore.activeRace.bestLap) }}</span>
            </div>
          </div>
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
import { msToHMS } from "@/helpers/msToHMS";
import { translate } from "@/helpers/translate";
import RacerList from "./RacerList.vue";

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
  background: rgba(0, 0, 0, 0.75);
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
  backdrop-filter: blur(8px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 6px;
  padding: 0.4rem 0.6rem;
  min-width: 160px;
  transition: all 0.3s ease;
}

.track-info {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.3rem 0.6rem;

  .track-icon {
    color: v.$primary-color-light;
    opacity: 0.8;
    font-size: 0.9rem;
  }

  .track-name {
    color: v.$text-color;
    font-size: 0.85rem;
    font-weight: 600;
    text-transform: uppercase;
    letter-spacing: 0.1em;
    opacity: 0.9;
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
  background: rgba(255, 255, 255, 0.05);
  border-radius: 4px;
  transition: all 0.2s ease;

  &:hover {
    background: rgba(255, 255, 255, 0.08);
  }

  .stat-icon {
    color: v.$primary-color-light;
    opacity: 0.8;
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
    background: rgba(v.$primary-color, 0.1);
    
    .stat-content {
      font-size: 1.4rem;
      color: v.$primary-color-light;
    }
    
    .stat-divider {
      opacity: 0.3;
      margin: 0 0.1rem;
      color: v.$text-color;
    }
    
    .total {
      font-size: 0.8em;
      opacity: 0.5;
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

.times-block {
  display: flex;
  flex-direction: column;
  gap: 0.2rem;
  margin-top: 0.3rem;
  padding-top: 0.3rem;
  border-top: 1px solid rgba(255, 255, 255, 0.08);
}

.time-module {
  display: flex;
  align-items: center;
  gap: 0.4rem;
  padding: 0.15rem 0.3rem;
  transition: all 0.2s ease;

  .time-icon {
    color: v.$primary-color-light;
    opacity: 0.6;
    font-size: 0.8rem;
  }

  .time-value {
    font-family: 'JetBrains Mono', monospace;
    font-size: 0.8rem;
    color: rgba(v.$text-color, 0.8);
    letter-spacing: 0.05em;
  }

  &.best {
    .time-icon {
      color: v.$primary-color-light;
      opacity: 0.9;
    }
    .time-value {
      color: v.$primary-color-light;
      font-weight: 600;
    }
  }

  &:hover {
    background: rgba(255, 255, 255, 0.05);
    border-radius: 4px;
  }
}

.ghost-indicator {
  display: flex;
  align-items: center;
  gap: 0.3rem;
  background: rgba(0, 0, 0, 0.6);
  color: v.$text-color;
  font-size: 0.7rem;
  padding: 0.15rem 0.4rem;
  border-radius: 4px;
  letter-spacing: 0.15em;
  opacity: 0.6;
  border: 1px solid rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(4px);
  
  .v-icon {
    font-size: 0.8rem;
    opacity: 0.8;
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

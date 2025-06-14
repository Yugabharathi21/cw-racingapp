<template>
  <Transition name="fade-scale">
  <div class="countdown" v-if="globalStore.countdown !== -1">
      <div class="countdown__wrapper">
        <!-- Big Number Display for counts > 5 -->
        <TransitionGroup name="countdown-number" v-if="globalStore.countdown > 5">
          <div :key="'big-' + globalStore.countdown" class="countdown__number countdown__number--large">
            {{ globalStore.countdown }}
          </div>
        </TransitionGroup>

        <!-- F1 Traffic Lights - Show only for countdown 5 and below -->
        <div class="traffic-lights" v-else>
          <div class="lights-container">
            <div class="light-row">
              <div class="light" :class="{ 
                'active': isLightActive(0), 
                'green': globalStore.countdown === 0 
              }"></div>
              <div class="light" :class="{ 
                'active': isLightActive(1), 
                'green': globalStore.countdown === 0 
              }"></div>
              <div class="light" :class="{ 
                'active': isLightActive(2), 
                'green': globalStore.countdown === 0 
              }"></div>
              <div class="light" :class="{ 
                'active': isLightActive(3), 
                'green': globalStore.countdown === 0 
              }"></div>
              <div class="light" :class="{ 
                'active': isLightActive(4), 
                'green': globalStore.countdown === 0 
              }"></div>
            </div>            <!-- Subtle Number Display -->
            <div class="number-display">
              <TransitionGroup name="countdown-number">
                <div v-if="globalStore.countdown > 5" :key="'pre-' + globalStore.countdown" class="countdown__number countdown__number--large">
                  {{ globalStore.countdown }}
                </div>
                <div v-else-if="globalStore.countdown > 0" :key="'count-' + globalStore.countdown" class="countdown__number">
                  {{ globalStore.countdown }}
                </div>
                <div v-else :key="'go'" class="countdown__go">
                  GO!
                </div>
              </TransitionGroup>
            </div>
          </div>
        </div>

        <!-- Decorative Elements -->
        <div class="countdown__rays" :class="{ 'animate-go': globalStore.countdown === 0 }"></div>
      </div>
    </div>
  </Transition>
</template>

<script setup lang="ts">
import { useGlobalStore } from "@/store/global";
import { computed } from "vue";

const globalStore = useGlobalStore();

// F1-style light sequence timing (5 lights)
const isLightActive = (index: number) => {
  const count = globalStore.countdown;
  
  // All lights green for GO!
  if (count === 0) return true;
  
  // Progressive lighting sequence for 5 and below:
  // 5: 1 light
  // 4: 2 lights
  // 3: 3 lights
  // 2: 4 lights
  // 1: 5 lights
  // 0: all green (handled above)
  const activeLights = 6 - count; // Convert countdown to number of active lights
  return index < activeLights;
};

const countdownColor = computed(() => {
  return globalStore.countdown === 0 ? "#00ff00" : "#ff0000";
});
</script>

<style scoped lang="scss">
@use '@/styles/variables' as v;

.countdown {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  background: transparent; // Remove dark background
}

.countdown__wrapper {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.traffic-lights {
  background: rgba(0, 0, 0, 0.9);
  padding: 1.2rem 1.5rem;
  border-radius: 0.8rem;
  box-shadow:
    0 0 0 2px rgba(255, 255, 255, 0.1),
    0 0 30px rgba(0, 0, 0, 0.5),
    inset 0 0 30px rgba(0, 0, 0, 0.5);
  position: relative;

  &::before {
    content: '';
    position: absolute;
    inset: -1px;
    background: linear-gradient(90deg,
      rgba(255, 255, 255, 0.1),
      rgba(255, 255, 255, 0.2),
      rgba(255, 255, 255, 0.1)
    );
    background-size: 200% 100%;
    border-radius: 0.8rem;
    animation: borderGlow 4s linear infinite;
    z-index: -1;
  }
}

.lights-container {
  display: flex;
  flex-direction: column;
  gap: 0.8rem;
  align-items: center;
}

.light-row {
  display: flex;
  gap: 0.8rem;
}

.light {
  width: 2rem;
  height: 2rem;
  border-radius: 50%;
  background: rgba(255, 0, 0, 0.2);
  box-shadow:
    0 0 10px rgba(0, 0, 0, 0.5),
    inset 0 0 10px rgba(0, 0, 0, 0.5);
  position: relative;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);

  &::before {
    content: '';
    position: absolute;
    inset: 2px;
    border-radius: 50%;
    background: radial-gradient(circle at 30% 30%,
      rgba(255, 255, 255, 0.5),
      rgba(255, 0, 0, 0.2) 60%
    );
    opacity: 0;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  }

  &.active {
    background: rgb(255, 0, 0);
    box-shadow:
      0 0 20px rgba(255, 0, 0, 0.5),
      0 0 40px rgba(255, 0, 0, 0.3),
      inset 0 0 15px rgba(255, 255, 255, 0.5);

    &::before {
      opacity: 1;
    }

    &.green {
      background: rgb(0, 255, 0);
      box-shadow:
        0 0 20px rgba(0, 255, 0, 0.5),
        0 0 40px rgba(0, 255, 0, 0.3),
        inset 0 0 15px rgba(255, 255, 255, 0.5);

      &::before {
        background: radial-gradient(circle at 30% 30%,
          rgba(255, 255, 255, 0.5),
          rgba(0, 255, 0, 0.2) 60%
        );
      }
    }
  }
}

.number-display {
  position: relative;
  height: 1.5rem;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 0.5rem;
}

.countdown__number {
  position: absolute;
  font-family: "Rajdhani", sans-serif;
  font-weight: 600;
  text-align: center;
  font-size: 1rem;
  letter-spacing: 0.05em;
  opacity: 0.7;
  color: rgba(255, 255, 255, 0.8);

  &--large {
    position: relative; // Override absolute positioning for large numbers
    font-size: 8rem; // Bigger size for better visibility
    font-weight: 700;
    opacity: 0.9;
    color: #ffffff;
    text-shadow: 
      0 0 20px rgba(255, 255, 255, 0.5),
      0 0 40px rgba(255, 255, 255, 0.3);
    margin: 2rem 0; // Add some spacing
  }
}

.countdown__go {
  position: absolute;
  font-family: "Rajdhani", sans-serif;
  font-weight: 700;
  text-align: center;
  color: #00ff00;
  text-shadow: 0 0 10px rgba(0, 255, 0, 0.5);
  font-size: 1.1rem;
}

.countdown__rays {
  position: absolute;
  width: 200%;
  height: 200%;
  background: radial-gradient(
    circle,
    transparent 30%,
    rgba(v.$primary-color, 0) 31%,
    rgba(v.$primary-color, 0.1) 32%,
    transparent 33%
  );
  background-size: 50px 50px;
  animation: rotateRays 20s linear infinite;
  opacity: 0.2;
  z-index: -1;

  &.animate-go {
    background: radial-gradient(
      circle,
      transparent 30%,
      rgba(0, 255, 0, 0) 31%,
      rgba(0, 255, 0, 0.1) 32%,
      transparent 33%
    );
    animation: rotateRaysGo 1s linear infinite;
    opacity: 0.4;
  }
}

@keyframes borderGlow {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}

// Transitions
.fade-scale-enter-active,
.fade-scale-leave-active {
  transition: all 0.5s ease;
}

.fade-scale-enter-from,
.fade-scale-leave-to {
  opacity: 0;
  transform: scale(0.9);
}

.countdown-number-enter-active,
.countdown-number-leave-active {
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  position: absolute;
}

.countdown-number-enter-from {
  opacity: 0;
  transform: scale(0.8) translateY(-50%);
}

.countdown-number-leave-to {
  opacity: 0;
  transform: scale(1.2) translateY(50%);
}

@keyframes rotateRays {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

@keyframes rotateRaysGo {
  from {
    transform: rotate(0deg) scale(1);
    opacity: 0.4;
  }
  to {
    transform: rotate(360deg) scale(1.5);
    opacity: 0;
  }
}
</style>
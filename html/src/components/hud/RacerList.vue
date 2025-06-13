<template>
  <div class="racers-holder">
    <TransitionGroup name="position-change">
      <div
        class="box"
        v-for="(racer, index) in shortenedRacers"
        :key="racer.RacerName"
        :class="{
          'me': index === globalStore.activeRace.position-1,
          'finished': racer.Finished
        }"
      >
        <div class="position-wrapper">
          <div class="number">{{ index + 1 }}</div>
          <div class="position-change-indicator" :class="getPositionChangeClass(racer)">
            <i class="fas" :class="getPositionChangeIcon(racer)"></i>
          </div>
        </div>
        <div class="racer-info">
          <span class="name">{{ racer.RacerName }}</span>
          <span class="difference" v-if="index === 0 && racer.Finished">{{ translate('winner') }}</span>
          <span class="difference" v-else-if="racer.Finished">{{ translate('finished') }}</span>
          <span class="difference" v-else-if="index !== globalStore.activeRace.position-1">
            {{ getTimeDifference(racers[globalStore.activeRace.position - 1], racer) }}
          </span>
        </div>
      </div>
    </TransitionGroup>
  </div>
</template>

<script setup lang="ts">
import { useGlobalStore } from "@/store/global";
import { ActiveRacer } from "../../store/types";
import { computed, ref, watch } from "vue";
import { translate } from "@/helpers/translate";

const props = defineProps<{
  racers: ActiveRacer[];
}>();

const globalStore = useGlobalStore();
const previousPositions = ref(new Map<string, number>());

const shortenedRacers = computed(() =>
  props.racers?.slice(
    0,
    globalStore.baseData?.data?.hudSettings?.maxPositions || 10
  )
);

watch(() => props.racers, (newRacers) => {
  // Store previous positions before update
  newRacers?.forEach((racer, index) => {
    if (!previousPositions.value.has(racer.RacerName)) {
      previousPositions.value.set(racer.RacerName, index);
    }
  });

  // Update positions for next comparison
  setTimeout(() => {
    previousPositions.value = new Map(
      newRacers?.map((racer, index) => [racer.RacerName, index]) || []
    );
  }, 100);
}, { deep: true });

const getPositionChangeClass = (racer: ActiveRacer) => {
  const currentPos = props.racers?.findIndex(r => r.RacerName === racer.RacerName) || 0;
  const previousPos = previousPositions.value.get(racer.RacerName) || currentPos;
  
  if (currentPos < previousPos) return 'position-up';
  if (currentPos > previousPos) return 'position-down';
  return '';
};

const getPositionChangeIcon = (racer: ActiveRacer) => {
  const currentPos = props.racers?.findIndex(r => r.RacerName === racer.RacerName) || 0;
  const previousPos = previousPositions.value.get(racer.RacerName) || currentPos;
  
  if (currentPos < previousPos) return 'fa-chevron-up';
  if (currentPos > previousPos) return 'fa-chevron-down';
  return '';
};

const formatTimeDifference = (timeDiffMs: number): string => {
  if (timeDiffMs === 0) {
    return "0.000";
  }

  const isAhead = timeDiffMs > 0;
  const absoluteDiffSeconds = Math.abs(timeDiffMs) / 1000;

  // Format to 3 decimal places
  const formattedTime = absoluteDiffSeconds.toFixed(3);

  return isAhead ? `+${formattedTime}` : `-${formattedTime}`;
};

const getTimeDifference = (racer1: ActiveRacer, racer2: ActiveRacer) => {
  const racer1Checkpoints = racer1.CheckpointTimes.length;
  const racer2Checkpoints = racer2.CheckpointTimes.length;

  if (racer1Checkpoints === 0 || racer2Checkpoints === 0) {
    return ''; // Not enough data to compare
  }

  const lastCommonCheckpoint = Math.min(racer1Checkpoints, racer2Checkpoints) - 1;
  const racer1Time = racer1.CheckpointTimes[lastCommonCheckpoint].time;
  const racer2Time = racer2.CheckpointTimes[lastCommonCheckpoint].time;

  const timeDifference = formatTimeDifference(racer2Time - racer1Time);

  return timeDifference;
};
</script>

<style scoped lang="scss">
@use '@/styles/variables' as v;

.racers-holder {
  display: flex;
  flex-direction: column;
  gap: 0.35em;
  padding: 0.4em;
  font-family: 'Rajdhani', sans-serif;
  position: relative;
  width: max-content;
  min-width: 280px;
}

@keyframes borderGlow {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}

.box {
  background: rgba(0, 0, 0, 0.75);
  width: 100%;
  font-size: 0.95em;
  height: 2.6em;
  display: flex;
  align-items: center;
  padding: 0 0.4em;
  border-radius: 4px;
  backdrop-filter: blur(8px);
  position: relative;
  overflow: hidden;
  transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);

  // Glowing border effect
  &::before {
    content: '';
    position: absolute;
    inset: 0;
    padding: 1px;
    border-radius: 4px;
    background: linear-gradient(
      90deg,
      rgba(v.$primary-color, 0.3),
      rgba(v.$primary-color-light, 0.1),
      rgba(v.$primary-color, 0.3)
    );
    background-size: 200% 100%;
    -webkit-mask: linear-gradient(#000 0 0) content-box, linear-gradient(#000 0 0);
    mask: linear-gradient(#000 0 0) content-box, linear-gradient(#000 0 0);
    -webkit-mask-composite: xor;
    mask-composite: exclude;
    animation: borderGlow 4s linear infinite;
  }

  &:hover {
    transform: translateX(2px);
    background: rgba(0, 0, 0, 0.85);
  }

  &.me {
    background: linear-gradient(
      90deg,
      rgba(v.$primary-color, 0.15),
      rgba(0, 0, 0, 0.75)
    );

    &::before {
      background: linear-gradient(
        90deg,
        rgba(v.$primary-color-light, 0.5),
        rgba(v.$primary-color, 0.3),
        rgba(v.$primary-color-light, 0.5)
      );
      background-size: 200% 100%;
    }

    .number {
      background: v.$primary-color-light;
      color: rgba(0, 0, 0, 0.9);
      font-weight: 700;
    }
    .name {
      color: v.$primary-color-light;
    }
  }

  &.finished {
    &::before {
      background: linear-gradient(
        90deg,
        rgba(v.$positive-color, 0.3),
        rgba(v.$positive-color, 0.1),
        rgba(v.$positive-color, 0.3)
      );
      background-size: 200% 100%;
    }
    .number {
      background: v.$positive-color;
    }
  }
}

.position-wrapper {
  display: flex;
  align-items: center;
  gap: 0.3em;
  min-width: 3.6em;
}

.number {
  background: rgba(v.$primary-color, 0.9);
  color: v.$text-color;
  width: 1.8em;
  height: 1.8em;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 3px;
  font-weight: 600;
  font-size: 1em;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.racer-info {
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex: 1;
  padding: 0 0.6em;
  gap: 0.5em;
}

.name {
  color: v.$text-color;
  font-weight: 500;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  max-width: 18em;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  transition: color 0.2s ease;
}

.difference {
  color: rgba(v.$text-color, 0.8);
  font-weight: 500;
  font-size: 0.85em;
  min-width: 4.5em;
  text-align: right;
  font-family: 'JetBrains Mono', monospace;
  letter-spacing: -0.02em;
}

.position-change-indicator {
  width: 1em;
  text-align: center;
  font-size: 0.8em;
  opacity: 0.9;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  
  &.position-up {
    color: v.$positive-color;
    animation: slideUp 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  }
  
  &.position-down {
    color: v.$negative-color;
    animation: slideDown 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  }
}

// Animations
@keyframes slideUp {
  from {
    transform: translateY(100%) scale(0.8);
    opacity: 0;
  }
  to {
    transform: translateY(0) scale(1);
    opacity: 1;
  }
}

@keyframes slideDown {
  from {
    transform: translateY(-100%) scale(0.8);
    opacity: 0;
  }
  to {
    transform: translateY(0) scale(1);
    opacity: 1;
  }
}

// Position change transitions
.position-change-move {
  transition: transform 0.5s cubic-bezier(0.4, 0, 0.2, 1);
}

.position-change-enter-active,
.position-change-leave-active {
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

.position-change-enter-from,
.position-change-leave-to {
  opacity: 0;
  transform: translateX(20px) scale(0.95);
}
</style>

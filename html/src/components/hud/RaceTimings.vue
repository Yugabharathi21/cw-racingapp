<template>
  <div class="times-grid">
    <!-- Current Time -->
    <div class="time-item" :class="{ 'time-item--highlight': isHighlightedTime('current') }">
      <div class="time-label">
        <v-icon size="x-small" color="orange">mdi-clock-outline</v-icon>
      </div>
      <div class="time-value">{{ msToHMS(time || 0) }}</div>
    </div>

    <!-- Total Time -->
    <div class="time-item" :class="{ 'time-item--highlight': isHighlightedTime('total') }">
      <div class="time-label">
        <v-icon size="x-small" color="orange">mdi-timer-outline</v-icon>
      </div>
      <div class="time-value">{{ msToHMS(totalTime) }}</div>
    </div>

    <!-- Best Lap -->
    <div class="time-item" :class="{ 'time-item--highlight': isHighlightedTime('best') }">
      <div class="time-label">
        <v-icon size="x-small" color="orange">mdi-star-outline</v-icon>
      </div>
      <div class="time-value">{{ msToHMS(bestLap) }}</div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { msToHMS } from "@/helpers/msToHMS";
import { ref, watch } from 'vue';

const props = defineProps<{
  time: number;
  totalTime: number;
  bestLap: number;
}>();

const highlightedTime = ref<string | null>(null);

const isHighlightedTime = (type: string) => highlightedTime.value === type;

// Format time labels with translations if needed
const formatTimeLabel = (type: string) => {
  const labels = {
    current: 'Current',
    total: 'Total',
    best: 'Best'
  };
  return labels[type as keyof typeof labels] || type;
};

// Watch for changes in times to trigger highlights
watch(() => props.time, (newVal, oldVal) => {
  if (oldVal && newVal < oldVal) highlightedTime.value = 'current';
  setTimeout(() => { if (highlightedTime.value === 'current') highlightedTime.value = null; }, 1000);
});

watch(() => props.bestLap, (newVal, oldVal) => {
  if (oldVal && newVal < oldVal) highlightedTime.value = 'best';
  setTimeout(() => { if (highlightedTime.value === 'best') highlightedTime.value = null; }, 1000);
});
</script>

<style scoped lang="scss">
@use '@/styles/variables' as v;

.times-grid {
  display: grid;
  grid-template-columns: repeat(3, minmax(80px, 1fr));
  gap: 0.4rem;
  margin-bottom: 0.4rem;
  width: 100%;
}

.time-item {
  position: relative;
  padding: 0.3rem 0.4rem;
  background: rgba(255, 255, 255, 0.05);
  border-radius: 4px;
  transition: all 0.2s ease;
  &:hover {
    background: rgba(255, 255, 255, 0.08);
  }

  .time-label {
    display: flex;
    align-items: center;
    justify-content: center;
    margin-bottom: 0.15rem;

    .v-icon {
      color: v.$primary-color-light;
      opacity: 0.8;
      font-size: 0.85rem;
    }
  }

  .time-value {
    font-family: 'JetBrains Mono', monospace;
    font-size: 0.85rem;
    color: v.$text-color;
    text-align: center;
    letter-spacing: 0.05em;
    font-weight: 600;
  }
  &--highlight {
    background: rgba(v.$primary-color, 0.15);
    animation: highlight-time 0.5s cubic-bezier(0.4, 0, 0.2, 1);

    .time-value {
      color: v.$primary-color-light;
      animation: pulse-text 0.5s cubic-bezier(0.4, 0, 0.2, 1);
    }

    .v-icon {
      opacity: 1;
    }
  }
}

@keyframes highlight-time {
  0% {
    background: rgba(v.$primary-color, 0.25);
    transform: scale(1.02);
  }
  100% {
    background: rgba(v.$primary-color, 0.15);
    transform: scale(1);
  }
}

@keyframes pulse-text {
  0% {
    opacity: 0.8;
    transform: scale(1.02);
  }
  50% {
    opacity: 1;
    transform: scale(1.05);
  }
  100% {
    opacity: 0.9;
    transform: scale(1);
  }
}
</style>
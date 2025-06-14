<template>
  <div class="editor">
    <div class="editor__container">
      <!-- Track Info -->
      <div class="editor__panel">
        <div class="info-list">
          <div class="info-item">
            <div class="info-label">
              <v-icon icon="mdi-map-marker" size="small" class="info-icon"></v-icon>
              <span>{{ translate("track_name") }}</span>
            </div>
            <span class="info-value">{{ globalStore.creatorData.RaceName }}</span>
          </div>
          <div class="info-item">
            <div class="info-label">
              <v-icon icon="mdi-flag-checkered" size="small" class="info-icon"></v-icon>
              <span>{{ translate("checkpoints") }}</span>
            </div>
            <span class="info-value">{{ globalStore.creatorData.Checkpoints?.length || 0 }}</span>
          </div>
          <div class="info-item">
            <div class="info-label">
              <v-icon icon="mdi-crosshairs-gps" size="small" class="info-icon"></v-icon>
              <span>{{ translate("closest_checkpoint") }}</span>
            </div>
            <span class="info-value">{{ globalStore.creatorData.ClosestCheckpoint || "-" }}</span>
          </div>
        </div>
      </div>

      <!-- Controls -->
      <div class="editor__panel">
        <div class="controls-list">
          <!-- Main Controls -->
          <div class="control-item">
            <span class="key success-key">{{ globalStore.buttons.AddCheckpoint }}</span>
            <span class="control-label">{{ translate("add_checkpoint") }}</span>
          </div>
          
          <div class="control-item" v-if="globalStore.creatorData.ClosestCheckpoint">
            <span class="key danger-key">{{ globalStore.buttons.DeleteCheckpoint }}</span>
            <span class="control-label">{{ translate("delete_checkpoint") }} {{ globalStore.creatorData.ClosestCheckpoint }}</span>
          </div>
          
          <div class="control-item">
            <span class="key info-key">{{ globalStore.buttons.MoveCheckpoint }}</span>
            <span class="control-label">{{ translate("modify_checkpoint") }}</span>
          </div>

          <!-- Distance Control -->
          <div class="control-item distance">
            <div class="distance-group">
              <span class="key danger-key">{{ globalStore.buttons.DecreaseDistance }}</span>
              <span class="key info-key">{{ globalStore.buttons.IncreaseDistance }}</span>
            </div>
            <div class="control-label">
              <span>{{ translate("tire_distance") }}</span>
              <span class="distance-value">[{{ globalStore.creatorData.TireDistance }}]</span>
            </div>
          </div>

          <!-- Actions -->
          <div class="actions-group">
            <div class="control-item">
              <span class="key danger-key">{{ globalStore.buttons.Exit }}</span>
              <span class="control-label">{{ translate("close_editor") }}</span>
            </div>
            <div class="control-item">
              <span class="key success-key">{{ globalStore.buttons.SaveRace }}</span>
              <span class="control-label">{{ translate("save_track") }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { useGlobalStore } from "@/store/global";
import { translate } from "@/helpers/translate";

const globalStore = useGlobalStore();
</script>

<style scoped lang="scss">
@use '@/styles/variables' as v;

.editor {
  position: fixed;
  top: 1vh;
  right: 1vw;
  z-index: 10;
}

.editor__container {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  max-width: 350px;
}

.editor__panel {
  background: rgba(0, 0, 0, 0.4);
  backdrop-filter: blur(6px);
  border-radius: v.$border-radius;
  padding: 0.6rem;
  box-shadow: 
    inset 0 0 0 1px rgba(255, 255, 255, 0.05),
    0 2px 4px rgba(0, 0, 0, 0.2);
  transition: all 0.2s ease;

  &:hover {
    background: rgba(0, 0, 0, 0.45);
    backdrop-filter: blur(8px);
  }
}

.info-list {
  display: flex;
  flex-direction: column;
  gap: 0.4rem;
}

.info-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 1rem;
  padding: 0.3rem 0.4rem;
  border-radius: v.$border-radius;
  transition: all 0.2s ease;
  background: rgba(255, 255, 255, 0.02);

  &:hover {
    background: rgba(255, 255, 255, 0.05);
  }

  .info-label {
    display: flex;
    align-items: center;
    gap: 0.4rem;
    color: rgba(v.$text-color, 0.7);
    font-size: 0.8rem;

    .info-icon {
      color: v.$primary-color;
      opacity: 0.9;
      filter: drop-shadow(0 0 2px rgba(v.$primary-color, 0.3));
    }
  }

  .info-value {
    color: v.$text-color;
    font-size: 0.8rem;
    font-weight: 600;
    letter-spacing: 0.02em;
  }
}

.controls-list {
  display: flex;
  flex-direction: column;
  gap: 0.4rem;
}

.control-item {
  display: flex;
  align-items: center;
  gap: 0.6rem;
  padding: 0.2rem 0;

  &.distance {
    .distance-group {
      display: flex;
      gap: 0.3rem;
    }

    .distance-value {
      color: v.$primary-color;
      margin-left: 0.3rem;
    }
  }
}

.control-label {
  color: rgba(v.$text-color, 0.8);
  font-size: 0.8rem;
  display: flex;
  align-items: center;
  gap: 0.3rem;
}

.key {
  background: rgba(255, 255, 255, 0.05);
  color: v.$text-color;
  padding: 0.15rem 0.4rem;
  border-radius: v.$border-radius;
  font-size: 0.75rem;
  min-width: 1.5rem;
  text-align: center;
  border: 1px solid rgba(255, 255, 255, 0.1);
  box-shadow: 
    0 2px 4px rgba(0, 0, 0, 0.1),
    inset 0 1px rgba(255, 255, 255, 0.05);
  transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
  backdrop-filter: blur(4px);

  &.success-key {
    background: rgba(v.$positive-color, 0.1);
    border-color: rgba(v.$positive-color, 0.2);
    color: rgba(v.$positive-color, 0.9);
    box-shadow: 
      0 2px 4px rgba(v.$positive-color, 0.1),
      inset 0 1px rgba(v.$positive-color, 0.05);
  }

  &.danger-key {
    background: rgba(v.$negative-color, 0.1);
    border-color: rgba(v.$negative-color, 0.2);
    color: rgba(v.$negative-color, 0.9);
    box-shadow: 
      0 2px 4px rgba(v.$negative-color, 0.1),
      inset 0 1px rgba(v.$negative-color, 0.05);
  }

  &.info-key {
    background: rgba(v.$primary-color, 0.1);
    border-color: rgba(v.$primary-color, 0.2);
    color: rgba(v.$primary-color, 0.9);
    box-shadow: 
      0 2px 4px rgba(v.$primary-color, 0.1),
      inset 0 1px rgba(v.$primary-color, 0.05);
  }

  &:hover {
    transform: translateY(-1px) scale(1.05);
    filter: brightness(1.2);
    border-color: currentColor;
  }
}

.actions-group {
  margin-top: 0.4rem;
  padding-top: 0.4rem;
  border-top: 1px solid rgba(255, 255, 255, 0.05);
  display: flex;
  flex-direction: column;
  gap: 0.4rem;
  position: relative;

  &::after {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    height: 1px;
    background: linear-gradient(
      90deg,
      transparent,
      rgba(255, 255, 255, 0.1),
      transparent
    );
  }
}

.control-item {
  display: flex;
  align-items: center;
  gap: 0.6rem;
  padding: 0.3rem 0.4rem;
  border-radius: v.$border-radius;
  transition: all 0.2s ease;
  background: rgba(255, 255, 255, 0.02);

  &:hover {
    background: rgba(255, 255, 255, 0.05);
  }

  &.distance {
    .distance-group {
      display: flex;
      gap: 0.3rem;
    }

    .distance-value {
      color: v.$primary-color;
      margin-left: 0.3rem;
      font-weight: 600;
      text-shadow: 0 0 10px rgba(v.$primary-color, 0.3);
    }
  }
}

// Simple fade transition
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>


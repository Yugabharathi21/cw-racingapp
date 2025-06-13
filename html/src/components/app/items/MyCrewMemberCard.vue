<template>
  <v-card 
    rounded="lg" 
    class="member-card" 
    :elevation="0"
  >
    <div class="card-wrapper">
      <!-- Member Info Section -->
      <div class="member-info">
        <!-- Name and Role -->
        <div class="name-section">
          <div class="member-name">
            <h3 class="name">{{ member.racername }}</h3>
            <v-tooltip 
              v-if="memberIsFounder"
              location="top" 
              text="Crew Founder"
            >
              <template v-slot:activator="{ props }">
                <v-icon
                  v-bind="props"
                  color="warning"
                  size="16"
                  class="founder-badge"
                  icon="mdi-star-circle"
                />
              </template>
            </v-tooltip>
          </div>
          <div class="status-badges">
            <v-chip
              size="small"
              color="primary"
              variant="flat"
              density="comfortable"
              class="id-chip"
            >
              <v-icon start size="14">mdi-id-card</v-icon>
              {{ member.citizenID }}
            </v-chip>
          </div>
        </div>

        <!-- Member Stats -->
        <div class="member-stats">
          <span class="stat">
            <v-icon size="16" color="primary">mdi-calendar-clock</v-icon>
            {{ translate('joined') }}: {{ new Date().toLocaleDateString() }}
          </span>
          <span v-if="isFounder" class="action-hint">
            <v-icon size="16" color="primary">mdi-information</v-icon>
            {{ translate('manage_member_hint') }}
          </span>
        </div>
      </div>

      <!-- Action Menu for Founder -->
      <div v-if="isFounder && !memberIsFounder" class="action-menu">
        <v-menu location="bottom end">
          <template v-slot:activator="{ props }">
            <v-btn
              v-bind="props"
              variant="text"
              density="comfortable"
              class="menu-btn"
              icon="mdi-dots-vertical"
            />
          </template>
          <v-list density="compact" nav class="action-list">
            <v-list-item
              prepend-icon="mdi-account-remove"
              @click="$emit('removeMember', member)"
              color="error"
              class="remove-item"
            >
              {{ translate('remove_member') }}
            </v-list-item>
          </v-list>
        </v-menu>
      </div>
    </div>
  </v-card>
</template>

<script setup lang="ts">
import { CrewMember } from "@/store/types";
import { translate } from "@/helpers/translate";

const props = defineProps<{
  member: CrewMember;
  memberIsFounder: boolean;
  isFounder: boolean;
}>();

defineEmits(['triggerReload', 'removeMember']);
</script>

<style scoped lang="scss">
@use "@/styles/variables" as v;

.member-card {
  width: 100%;
  background: linear-gradient(165deg, rgba(20, 20, 20, 0.95), rgba(15, 15, 15, 0.9));
  border: 1px solid rgba(255, 255, 255, 0.06);
  overflow: hidden;
  backdrop-filter: blur(10px);
  transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);

  &:hover {
    border-color: rgba(v.$primary-color, 0.15);
    background: linear-gradient(165deg, rgba(25, 25, 25, 0.95), rgba(20, 20, 20, 0.9));
  }
}

.card-wrapper {
  display: flex;
  gap: 1rem;
  padding: 0.75rem;
  align-items: center;
}

.member-info {
  flex: 1;
  min-width: 0;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.name-section {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  min-width: 0;

  @media (max-width: 500px) {
    flex-direction: column;
    align-items: flex-start;
    gap: 0.5rem;
  }
}

.member-name {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  min-width: 0;

  .name {
    margin: 0;
    font-size: 1rem;
    font-weight: 600;
    color: rgba(255, 255, 255, 0.95);
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  .founder-badge {
    flex-shrink: 0;
  }
}

.status-badges {
  display: flex;
  gap: 0.5rem;
  flex-shrink: 0;

  .id-chip {
    font-size: 0.8rem;
    font-weight: 500;
  }
}

.member-stats {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 1rem;
  font-size: 0.85rem;
  color: rgba(255, 255, 255, 0.7);

  .stat {
    display: flex;
    align-items: center;
    gap: 0.35rem;
  }

  .action-hint {
    display: flex;
    align-items: center;
    gap: 0.35rem;
    font-style: italic;
    color: rgba(255, 255, 255, 0.5);
  }
}

.action-menu {
  flex-shrink: 0;

  .menu-btn {
    width: 36px;
    height: 36px;
    border-radius: 4px;
    background: rgba(255, 255, 255, 0.05);

    &:hover {
      background: rgba(255, 255, 255, 0.1);
    }
  }
}

:deep(.action-list) {
  min-width: 200px;
  padding: 4px;
  background: rgba(30, 30, 30, 0.95);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.08);

  .remove-item {
    border-radius: 4px;
    margin: 0;
    min-height: 36px;
    font-size: 0.9rem;
    
    &:hover {
      background: rgba(244, 67, 54, 0.15) !important;
    }
  }
}
</style>

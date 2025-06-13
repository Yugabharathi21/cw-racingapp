<template>
  <v-card rounded="lg" class="bounty-card" :elevation="0">
    <div class="card-wrapper">
      <!-- Bounty Info Section -->
      <div class="bounty-info">
        <div class="bounty-header">
          <div class="name-section">
            <h3 class="track-name">{{ bounty.trackName }}</h3>
            <div class="badge-row">
              <v-chip 
                size="small"
                :color="meetsRequiredRank ? 'success' : 'error'"
                variant="flat"
                density="comfortable"
                class="rank-chip"
              >
                <v-icon start size="14">{{ meetsRequiredRank ? 'mdi-check-circle' : 'mdi-alert-circle' }}</v-icon>
                {{ translate('rank') }} {{ bounty.rankRequired }}
              </v-chip>
              <v-chip
                size="small"
                :color="hasBeenCompleted ? 'success' : 'primary'"
                variant="flat"
                density="comfortable"
                class="time-chip"
              >
                <v-icon start size="14">mdi-clock-outline</v-icon>
                {{ msToHMS(bounty.timeToBeat) }}
              </v-chip>
            </div>
          </div>

          <div class="track-stats">
            <span class="stat price">
              <v-icon size="16" color="success">mdi-cash</v-icon>
              {{ translate('currency_text') }}{{ bounty.price }}
            </span>
            <span class="stat-divider">•</span>
            <span class="stat">
              <v-icon size="16" color="primary">mdi-car</v-icon>
              {{ translate('class') }} {{ bounty.maxClass }}
            </span>
            <span class="stat-divider">•</span>
            <span class="stat">
              <v-icon size="16" color="primary">mdi-flag-checkered</v-icon>
              {{ translate(bounty.sprint ? 'sprint' : 'circuit') }}
            </span>
            <v-icon 
              v-if="bounty.reversed" 
              size="16" 
              color="primary" 
              class="status-icon"
              icon="mdi-backup-restore"
            />
          </div>
        </div>
      </div>

      <!-- Action Buttons -->
      <div class="action-bar">
        <v-btn
          variant="tonal"
          density="comfortable"
          class="action-btn"
          @click="recordsDialog = true"
        >
          <v-icon start size="18">mdi-trophy</v-icon>
          {{ translate('records') }}
        </v-btn>
        <v-btn
          variant="flat"
          color="primary"
          density="comfortable"
          class="action-btn"
          :disabled="!meetsRequiredRank || loading"
          :loading="loading"
          @click="quickHost"
        >
          <v-icon start size="18">mdi-play</v-icon>
          {{ translate('quick_host') }}
        </v-btn>
      </div>
    </div>

    <!-- Records Dialog -->
    <v-dialog 
      attach=".app-container" 
      v-model="recordsDialog"
      width="500"
      class="records-dialog"
      content-class="dialog-content"
    >
      <v-card rounded="lg" class="records-card">
        <div class="dialog-header">
          <div class="header-content">
            <h3 class="dialog-title">{{ bounty.trackName }}</h3>
            <div class="dialog-stats">
              <v-chip color="primary" size="small" variant="flat" density="comfortable">
                <v-icon start size="14">mdi-car</v-icon>
                {{ translate('class') }} {{ bounty.maxClass }}
              </v-chip>
              <v-chip v-if="bounty.reversed" color="primary" size="small" variant="flat" density="comfortable">
                <v-icon start size="14">mdi-backup-restore</v-icon>
                {{ translate('reversed') }}
              </v-chip>
              <v-chip color="primary" size="small" variant="flat" density="comfortable">
                <v-icon start size="14">mdi-flag-checkered</v-icon>
                {{ translate(bounty.sprint ? 'sprint' : 'circuit') }}
              </v-chip>
            </div>
          </div>
          <v-btn
            variant="text"
            icon="mdi-close"
            size="small"
            @click="recordsDialog = false"
          />
        </div>

        <v-card-text class="records-content">
          <v-table v-if="times && times.length > 0">
            <thead>
              <tr>
                <th class="text-left">#</th>
                <th class="text-left">{{ translate('racer') }}</th>
                <th class="text-left">{{ translate('time') }}</th>
                <th class="text-left">{{ translate('vehicle') }}</th>
              </tr>
            </thead>
            <tbody>
              <tr
                v-for="(item, index) in times"
                :key="index"
                :class="{ 'current-user': item.racerName === globalStore.baseData.data.currentRacerName }"
              >
                <td class="text-left">{{ index + 1 }}</td>
                <td>{{ item.racerName }}</td>
                <td>{{ msToHMS(item.time) }}</td>
                <td>{{ item.vehicleModel }}</td>
              </tr>
            </tbody>
          </v-table>
          <InfoText v-else :title="translate('no_data')" />
        </v-card-text>
      </v-card>
    </v-dialog>
  </v-card>
</template>

<script setup lang="ts">

import { msToHMS } from "@/helpers/msToHMS";
import { translate } from "@/helpers/translate";
import { useGlobalStore } from "@/store/global";
import { Bounty } from "@/store/types";
import { computed, ref } from "vue";
import InfoText from "../components/InfoText.vue";
import api from "@/api/axios";

const props = defineProps<{
  bounty: Bounty;
}>();

const loading = ref(false)
const recordsDialog = ref(false)
const globalStore = useGlobalStore()
const times = computed(() => {
  const claims = props.bounty.claimed
  const timearray = Object.values(claims).sort((res1, res2) => res1.time < res2.time ? -1 : 1 )
  return timearray
})


const hasBeenCompleted = computed(() => {
  return times.value.filter((time) => time.racerName === globalStore.baseData.data.currentRacerName).length > 0
})

const meetsRequiredRank = computed(() => {
  return props.bounty.rankRequired <= globalStore.baseData.data.currentRanking
})


const quickHost = async () => {
  loading.value = true
  const res = await api.post("UiQuickSetupBounty", JSON.stringify(props.bounty));
  if (res.data) {
    loading.value = false
  }
}

</script>

<style scoped lang="scss">
@use "@/styles/variables" as v;

.bounty-card {
  width: 100%;
  background: linear-gradient(165deg, rgba(20, 20, 20, 0.95), rgba(15, 15, 15, 0.9));
  border: 1px solid rgba(255, 255, 255, 0.06);
  overflow: hidden;
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

  @media (max-width: 600px) {
    flex-direction: column;
    align-items: stretch;
    gap: 0.75rem;
  }
}

.bounty-info {
  flex: 1;
  min-width: 0;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.bounty-header {
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

  @media (max-width: 600px) {
    flex-direction: column;
    align-items: flex-start;
    gap: 0.5rem;
  }
}

.track-name {
  margin: 0;
  font-size: 1rem;
  font-weight: 600;
  color: rgba(255, 255, 255, 0.95);
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  min-width: 0;
  flex: 1;
}

.badge-row {
  display: flex;
  gap: 0.5rem;
  flex-shrink: 0;

  .v-chip {
    font-weight: 500;
    font-size: 0.8rem;
  }
}

.track-stats {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  flex-wrap: wrap;
  font-size: 0.85rem;
  color: rgba(255, 255, 255, 0.7);

  .stat {
    display: flex;
    align-items: center;
    gap: 0.25rem;

    &.price {
      color: #4CAF50;
    }
  }

  .stat-divider {
    color: rgba(255, 255, 255, 0.2);
    font-size: 0.75rem;
    margin: 0 0.25rem;
  }

  .status-icon {
    margin-left: 0.25rem;
  }
}

.action-bar {
  display: flex;
  gap: 0.5rem;
  flex-shrink: 0;

  @media (max-width: 600px) {
    border-top: 1px solid rgba(255, 255, 255, 0.06);
    padding-top: 0.75rem;
  }

  .action-btn {
    height: 36px;
    font-size: 0.9rem;
    font-weight: 500;
    border-radius: 4px;
    padding: 0 1rem;
    text-transform: none;
    white-space: nowrap;

    .v-icon {
      margin-right: 4px;
    }

    @media (max-width: 600px) {
      flex: 1;
    }
  }
}

// Dialog Styles
.records-dialog {
  :deep(.dialog-content) {
    border-radius: 8px;
    overflow: hidden;
  }
}

.records-card {
  background: rgba(20, 20, 20, 0.98);
  border: 1px solid rgba(255, 255, 255, 0.08);
  backdrop-filter: blur(10px);
}

.dialog-header {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 1rem;
  padding: 1rem;
  border-bottom: 1px solid rgba(255, 255, 255, 0.06);

  .header-content {
    flex: 1;
    min-width: 0;
    display: flex;
    flex-direction: column;
    gap: 0.75rem;
  }
}

.dialog-title {
  margin: 0;
  font-size: 1.1rem;
  font-weight: 600;
  color: rgba(255, 255, 255, 0.95);
}

.dialog-stats {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;

  .v-chip {
    font-size: 0.8rem;
    font-weight: 500;
  }
}

.records-content {
  padding: 1rem;

  :deep(.v-table) {
    background: transparent !important;
    
    th {
      color: rgba(255, 255, 255, 0.6);
      font-size: 0.8rem;
      font-weight: 600;
      text-transform: uppercase;
      letter-spacing: 0.5px;
      padding: 0.5rem 1rem;
      border-bottom: 1px solid rgba(255, 255, 255, 0.06);
    }

    td {
      height: 40px;
      color: rgba(255, 255, 255, 0.8);
      font-size: 0.9rem;
      padding: 0.5rem 1rem;
    }

    tr.current-user td {
      color: v.$primary-color;
      font-weight: 500;
    }

    tr:hover td {
      background: rgba(255, 255, 255, 0.02) !important;
    }
  }
}
</style>

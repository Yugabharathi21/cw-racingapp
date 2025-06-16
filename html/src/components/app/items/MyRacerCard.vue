<template>
  <v-card 
    rounded="lg" 
    class="racer-card" 
    :elevation="0"
  >
    <div class="card-wrapper">
      <!-- Racer Info Section -->
      <div class="racer-info">
        <div class="racer-header">
          <!-- Name and Status -->
          <div class="name-section">
            <h3 class="racer-name">{{ racer.racername }}</h3>
            <div class="status-badges">
              <v-chip
                size="small"
                :color="revokedBool ? 'error' : 'success'"
                variant="flat"
                density="comfortable"
                class="status-chip"
              >
                <v-icon start size="14">
                  {{ revokedBool ? 'mdi-account-cancel' : 'mdi-account-check' }}
                </v-icon>
                {{ revokedBool ? translate('revoked') : translate('active') }}
              </v-chip>
              <v-chip
                size="small"
                color="primary"
                variant="flat"
                density="comfortable"
                class="auth-chip"
              >
                <v-icon start size="14">mdi-shield-account</v-icon>
                {{ racer.auth }}
              </v-chip>
            </div>
          </div>

          <!-- Racer Stats -->
          <div class="stats-grid">
            <div class="stat-item">
              <v-icon size="16" color="primary">mdi-id-card</v-icon>
              <span class="stat-label">{{ translate('citizen_id') }}</span>
              <span class="stat-value allow-select">{{ racer.citizenid }}</span>
            </div>
            <div class="stat-item">
              <v-icon size="16" color="primary">mdi-map-marker-path</v-icon>
              <span class="stat-label">{{ translate('tracks') }}</span>
              <span class="stat-value">{{ racer.tracks }}</span>
            </div>
            <div class="stat-item">
              <v-icon size="16" color="primary">mdi-flag-checkered</v-icon>
              <span class="stat-label">{{ translate('races') }}</span>
              <span class="stat-value">{{ racer.races }}</span>
            </div>
            <div class="stat-item">
              <v-icon size="16" color="success">mdi-trophy</v-icon>
              <span class="stat-label">{{ translate('wins') }}</span>
              <span class="stat-value">{{ racer.wins }}</span>
            </div>
          </div>

          <!-- Created By Info -->
          <div class="created-by">
            <v-icon size="16" color="primary">mdi-account-plus</v-icon>
            {{ translate('created_by') }}: {{ racer.createdby ? racer.createdby : translate('unknown') }}
          </div>
        </div>
      </div>

      <!-- Action Buttons -->
      <div class="action-bar">
        <v-btn
          v-if="globalStore.baseData.data.auth.controlAll"
          variant="tonal"
          density="comfortable"
          class="action-btn"
          @click="authDialog = true"
        >
          <v-icon start size="18">mdi-shield-edit</v-icon>
          {{ translate('auth') }}
        </v-btn>
        <v-btn
          :color="revokedBool ? 'success' : 'warning'"
          variant="tonal"
          density="comfortable"
          class="action-btn"
          @click="revokeDialog = true"
        >
          <v-icon start size="18">
            {{ revokedBool ? 'mdi-account-check' : 'mdi-account-cancel' }}
          </v-icon>
          {{ revokedBool ? translate('activate') : translate('revoke') }}
        </v-btn>
        <v-btn
          v-if="globalStore.baseData.data.auth.controlAll"
          color="error"
          variant="text"
          density="comfortable"
          class="action-btn"
          @click="deleteDialog = true"
        >
          <v-icon start size="18">mdi-delete</v-icon>
          {{ translate('delete_user') }}
        </v-btn>
      </div>
    </div>

    <!-- Auth Dialog -->
    <v-dialog 
      attach=".app-container" 
      v-model="authDialog"
      width="400"
      class="auth-dialog"
    >
      <v-card rounded="lg" class="dialog-card">
        <div class="dialog-header">
          <h3 class="dialog-title">
            {{ translate('change_auth') }}
          </h3>
          <v-btn
            variant="text"
            icon="mdi-close"
            size="small"
            @click="authDialog = false"
          />
        </div>

        <v-card-text class="dialog-content">
          <v-select
            color="primary"
            density="comfortable"
            :items="allAuths"
            item-value="value"
            item-title="text"
            :label="translate('auth')"
            v-model="authority"
            variant="outlined"
            hide-details
            class="auth-select"
          />
        </v-card-text>

        <v-card-actions class="dialog-actions">
          <v-spacer/>
          <v-btn
            variant="tonal"
            @click="authDialog = false"
          >
            {{ translate('cancel') }}
          </v-btn>
          <v-btn
            color="primary"
            variant="flat"
            @click="confirmAuth()"
          >
            {{ translate('confirm') }}
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Revoke Dialog -->
    <v-dialog 
      attach=".app-container" 
      v-model="revokeDialog"
      width="400"
      class="revoke-dialog"
    >
      <v-card rounded="lg" class="dialog-card">
        <div class="dialog-header">
          <h3 class="dialog-title">
            {{ revokedBool ? translate('activate') : translate('revoke') }}
            {{ translate('access_for') }} {{ racer.racername }}?
          </h3>
          <v-btn
            variant="text"
            icon="mdi-close"
            size="small"
            @click="revokeDialog = false"
          />
        </div>

        <v-card-text class="dialog-content">
          {{ translate('can_be_reverted') }}
        </v-card-text>

        <v-card-actions class="dialog-actions">
          <v-spacer/>
          <v-btn
            variant="tonal"
            @click="revokeDialog = false"
          >
            {{ translate('cancel') }}
          </v-btn>
          <v-btn
            :color="revokedBool ? 'success' : 'warning'"
            variant="flat"
            @click="revokeUser()"
          >
            {{ translate('confirm') }}
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Delete Dialog -->
    <v-dialog 
      attach=".app-container" 
      v-model="deleteDialog"
      width="400"
      class="delete-dialog"
    >
      <v-card rounded="lg" class="dialog-card">
        <div class="dialog-header">
          <h3 class="dialog-title">
            {{ translate('delete_user') }} {{ racer.racername }}?
          </h3>
          <v-btn
            variant="text"
            icon="mdi-close"
            size="small"
            @click="deleteDialog = false"
          />
        </div>

        <v-card-text class="dialog-content warning">
          <v-icon color="error" size="20">mdi-alert-circle</v-icon>
          {{ translate('cant_be_reverted') }}
        </v-card-text>

        <v-card-actions class="dialog-actions">
          <v-spacer/>
          <v-btn
            variant="tonal"
            @click="deleteDialog = false"
          >
            {{ translate('cancel') }}
          </v-btn>
          <v-btn
            color="error"
            variant="flat"
            @click="deleteUser()"
          >
            {{ translate('confirm') }}
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-card>
</template>

<script setup lang="ts">
import api from "@/api/axios";
import { useGlobalStore } from "@/store/global";
import { MyRacer } from "@/store/types";
import { computed } from "vue";
import { ref } from "vue";
import { translate } from "@/helpers/translate";

const props = defineProps<{
  racer: MyRacer;
}>();
const globalStore = useGlobalStore();

const authDialog = ref(false);
const revokeDialog = ref(false);
const deleteDialog = ref(false);
const emits = defineEmits(['triggerReload'])
const revokedBool = computed(() => props.racer.revoked == 1 ? true : false)
const authority = ref(props.racer.auth)

const allAuths = computed(() => {
  const newAuths = globalStore.baseData.data.allAuthorities;
  const authList: string[] = []
  Object.entries(newAuths).map(([key, value]) => {
    authList.push(key)
  });
  return authList
});

const confirmAuth = () => {
    api.post('UiChangeAuth', JSON.stringify({racername: props.racer.racername, auth: authority.value, citizenId: props.racer.citizenid }))
    authDialog.value = false;
    setTimeout(() => {
      emits('triggerReload')
    }, 1000);
};
const revokeUser = () => {
    api.post('UiRevokeRacer', JSON.stringify({racername: props.racer.racername, status: props.racer.revoked }))
    revokeDialog.value = false;
    setTimeout(() => {
      emits('triggerReload')
    }, 1000);
};
const deleteUser = async () => {
    api.post('UiRemoveRacer',  JSON.stringify({racername: props.racer.racername}))
    deleteDialog.value = false;
    setTimeout(() => {
      emits('triggerReload')
    }, 1000);
};


</script>

<style scoped lang="scss">
@use "@/styles/variables" as v;

.racer-card {
  background: rgba(0, 0, 0, 0.85) !important;
  border: 1px solid rgba(255, 255, 255, 0.08);
  min-width: 240px;
  padding: 0.75rem !important;
}

.card-wrapper {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.racer-info {
  flex: 1;
}

.racer-header {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.name-section {
  display: flex;
  flex-direction: column;
  gap: 0.35rem;
}

.racer-name {
  font-size: 1rem;
  font-weight: 500;
  margin: 0;
  color: rgba(255, 255, 255, 0.9);
}

.status-badges {
  display: flex;
  gap: 0.35rem;
  
  :deep(.v-chip) {
    height: 24px !important;
    font-size: 0.75rem;
    font-weight: 500;
    
    .v-icon {
      font-size: 14px;
      margin-right: 4px;
    }
  }
}

.stats-grid {
  display: flex;
  flex-direction: column;
  gap: 0.35rem;
  margin-top: 0.25rem;
}

.stat-item {
  display: flex;
  align-items: center;
  gap: 0.35rem;
  font-size: 0.8rem;

  .v-icon {
    opacity: 0.9;
  }

  .stat-label {
    color: rgba(255, 255, 255, 0.7);
    margin-right: auto;
  }

  .stat-value {
    font-family: 'JetBrains Mono', monospace;
    font-weight: 500;
    color: rgba(255, 255, 255, 0.9);
  }
}

.created-by {
  display: flex;
  align-items: center;
  gap: 0.35rem;
  font-size: 0.75rem;
  color: rgba(255, 255, 255, 0.6);
  margin-top: 0.25rem;
}

.action-bar {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
  margin-top: 0.25rem;

  .v-btn {
    height: 28px !important;
    font-size: 0.75rem !important;
    min-height: unset !important;
    text-transform: none;
    letter-spacing: 0;

    .v-icon {
      font-size: 14px !important;
      margin-right: 4px !important;
    }
  }
}

// Dialog Styles
.dialog-card {
  background: rgba(20, 20, 20, 0.98);
  border: 1px solid rgba(255, 255, 255, 0.08);
  backdrop-filter: blur(10px);
}

.dialog-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 1rem;
  border-bottom: 1px solid rgba(255, 255, 255, 0.06);
}

.dialog-title {
  margin: 0;
  font-size: 1.1rem;
  font-weight: 600;
  color: rgba(255, 255, 255, 0.95);
}

.dialog-content {
  padding: 1.25rem 1rem;
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.7);

  &.warning {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    color: rgb(244, 67, 54);
  }

  :deep(.v-input) {
    .v-field {
      border-radius: 4px;
      background: rgba(255, 255, 255, 0.05);
      
      &:hover {
        background: rgba(255, 255, 255, 0.08);
      }
    }
  }
}

.dialog-actions {
  padding: 1rem;
  border-top: 1px solid rgba(255, 255, 255, 0.06);
  gap: 0.5rem;

  .v-btn {
    min-width: 100px;
    font-weight: 500;
    text-transform: none;
  }
}
</style>

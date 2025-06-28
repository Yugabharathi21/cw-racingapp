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
  backdrop-filter: blur(6px);
  -webkit-backdrop-filter: blur(6px);
  min-width: 240px;
  padding: 0.75rem !important;
  position: relative;
  overflow: hidden;

  &::before {
    content: '';
    position: absolute;
    top: 0;
    left: -50%;
    width: 200%;
    height: 2px;
    background: linear-gradient(
      90deg,
      transparent,
      rgba(v.$primary-color, 0.2),
      rgba(v.$primary-color, 0.3),
      rgba(v.$primary-color, 0.2),
      transparent
    );
    opacity: 0;
    transition: opacity 0.3s ease;
  }

  &:hover::before {
    opacity: 1;
  }
}

.card-wrapper {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  position: relative;
  z-index: 1;
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
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.status-badges {
  display: flex;
  gap: 0.35rem;
  
  :deep(.v-chip) {
    height: 24px !important;
    font-size: 0.75rem;
    font-weight: 500;
    border: 1px solid rgba(255, 255, 255, 0.1);
    background: rgba(0, 0, 0, 0.3) !important;
    transition: all 0.2s ease;
    
    &:hover {
      border-color: rgba(255, 255, 255, 0.2);
    }
    
    .v-icon {
      font-size: 14px;
      margin-right: 4px;
      opacity: 0.9;
    }
  }
}

.stats-grid {
  display: flex;
  flex-direction: column;
  gap: 0.35rem;
  margin-top: 0.25rem;
  padding: 0.5rem;
  background: rgba(255, 255, 255, 0.03);
  border-radius: 4px;
  border: 1px solid rgba(255, 255, 255, 0.05);
}

.stat-item {
  display: flex;
  align-items: center;
  gap: 0.35rem;
  font-size: 0.8rem;
  padding: 0.25rem 0.5rem;
  border-radius: 3px;
  transition: background-color 0.2s ease;

  &:hover {
    background: rgba(255, 255, 255, 0.03);
  }

  .v-icon {
    opacity: 0.9;
    color: v.$primary-color !important;
  }

  .stat-label {
    color: rgba(255, 255, 255, 0.7);
    margin-right: auto;
    font-weight: 500;
  }

  .stat-value {
    font-family: 'JetBrains Mono', monospace;
    font-weight: 500;
    color: rgba(255, 255, 255, 0.9);
    text-shadow: 0 0 10px rgba(v.$primary-color, 0.3);
  }
}

.created-by {
  display: flex;
  align-items: center;
  gap: 0.35rem;
  font-size: 0.75rem;
  color: rgba(255, 255, 255, 0.6);
  margin-top: 0.25rem;
  padding: 0.25rem 0.5rem;
  border-radius: 3px;
  background: rgba(0, 0, 0, 0.2);

  .v-icon {
    color: v.$primary-color !important;
    opacity: 0.8;
  }
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
    border: 1px solid rgba(255, 255, 255, 0.1);
    background: rgba(0, 0, 0, 0.3) !important;
    transition: all 0.2s ease !important;

    &:hover {
      border-color: rgba(255, 255, 255, 0.2) !important;
      background: rgba(0, 0, 0, 0.4) !important;
    }

    .v-icon {
      font-size: 14px !important;
      margin-right: 4px !important;
      opacity: 0.9;
    }

    &.error {
      border-color: rgba(244, 67, 54, 0.3) !important;
      
      &:hover {
        border-color: rgba(244, 67, 54, 0.5) !important;
      }
    }
  }
}

// Dialog Styles
.dialog-card {
  background: rgba(0, 0, 0, 0.95) !important;
  border: 1px solid rgba(255, 255, 255, 0.08);
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
}

.dialog-header {
  padding: 1rem;
  border-bottom: 1px solid rgba(255, 255, 255, 0.06);
}

.dialog-title {
  margin: 0;
  font-size: 1rem;
  font-weight: 600;
  color: rgba(255, 255, 255, 0.95);
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.dialog-content {
  padding: 1.25rem 1rem;
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.7);

  :deep(.v-input) {
    .v-field {
      border-radius: 4px;
      background: rgba(255, 255, 255, 0.05) !important;
      border: 1px solid rgba(255, 255, 255, 0.1);
      transition: all 0.2s ease;
      
      &:hover {
        background: rgba(255, 255, 255, 0.08) !important;
        border-color: rgba(255, 255, 255, 0.2);
      }
    }
  }
}
</style>

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
  align-items: flex-start;

  @media (max-width: 700px) {
    flex-direction: column;
    align-items: stretch;
    gap: 0.75rem;
  }
}

.racer-info {
  flex: 1;
  min-width: 0;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.racer-header {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
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

.racer-name {
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

.status-badges {
  display: flex;
  gap: 0.5rem;
  flex-shrink: 0;

  .v-chip {
    font-size: 0.8rem;
    font-weight: 500;
  }
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
  gap: 0.75rem;
  padding: 0.5rem;
  background: rgba(255, 255, 255, 0.03);
  border-radius: 4px;
}

.stat-item {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 0.85rem;

  .stat-label {
    color: rgba(255, 255, 255, 0.6);
    margin-right: auto;
  }

  .stat-value {
    font-weight: 500;
    color: rgba(255, 255, 255, 0.9);

    &.allow-select {
      user-select: all;
    }
  }
}

.created-by {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 0.85rem;
  color: rgba(255, 255, 255, 0.6);
  padding-top: 0.25rem;
}

.action-bar {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  flex-shrink: 0;
  min-width: 140px;

  @media (max-width: 700px) {
    flex-direction: row;
    border-top: 1px solid rgba(255, 255, 255, 0.06);
    padding-top: 0.75rem;
  }

  @media (max-width: 500px) {
    flex-direction: column;
  }

  .action-btn {
    height: 36px;
    font-size: 0.9rem;
    font-weight: 500;
    border-radius: 4px;
    padding: 0 1rem;
    text-transform: none;
    white-space: nowrap;
    width: 100%;

    .v-icon {
      margin-right: 4px;
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

<template>
  <div class="admin-page">
    <v-card class="admin-card" flat>
      <!-- Header -->
      <div class="admin-header">
        <div class="header-content">
          <h2 class="header-title">
            <v-icon start size="20">mdi-shield-crown</v-icon>
            {{ translate('race_admin') }}
          </h2>
          <span class="header-subtitle">{{ translate('admin_controls') }}</span>
        </div>
      </div>

      <!-- Admin Controls -->
      <div class="admin-content">
        <!-- Hosting Controls -->
        <div v-if="globalStore.baseData.data.auth.handleHosting" class="control-section">
          <div class="section-header">
            <v-icon size="18" color="primary" class="section-icon">mdi-server</v-icon>
            <span class="section-title">{{ translate('hosting_controls') }}</span>
          </div>

          <div class="control-grid">
            <div class="control-item">
              <v-switch
                v-model="settings.hostingIsEnabled"
                color="primary"
                density="compact"
                hide-details
                :loading="loading"
                :ripple="false"
                class="control-switch"
                @change="updateHosting"
              >
                <template v-slot:label>
                  <div class="switch-content">
                    <span class="switch-label">{{ translate('enable_hosting') }}</span>
                    <v-icon size="16">mdi-cloud</v-icon>
                  </div>
                </template>
              </v-switch>
            </div>
          </div>
        </div>

        <!-- Auto-Host Controls -->
        <div v-if="globalStore.baseData.data.auth.handleAutoHost" class="control-section">
          <div class="section-header">
            <v-icon size="18" color="primary" class="section-icon">mdi-robot</v-icon>
            <span class="section-title">{{ translate('auto_host') }}</span>
          </div>

          <div class="control-grid">
            <div class="control-item">
              <v-switch
                v-model="settings.autoHostIsEnabled"
                color="primary"
                density="compact"
                hide-details
                :loading="loading"
                :ripple="false"
                class="control-switch"
                @change="updateAutoHost"
              >
                <template v-slot:label>
                  <div class="switch-content">
                    <span class="switch-label">{{ translate('enable_auto_hosting') }}</span>
                    <v-icon size="16">mdi-auto-fix</v-icon>
                  </div>
                </template>
              </v-switch>
            </div>

            <div class="control-item">
              <v-btn
                variant="flat"
                color="primary"
                density="compact"
                class="action-btn"
                :disabled="!settings.autoHostIsEnabled || loading"
                @click="newAutoHost"
              >
                <v-icon start size="16">mdi-plus</v-icon>
                {{ translate('trigger_new_autohost') }}
              </v-btn>
            </div>
          </div>
        </div>

        <!-- Bounty Controls -->
        <div v-if="globalStore.baseData.data.auth.handleBounties" class="control-section">
          <div class="section-header">
            <v-icon size="18" color="primary" class="section-icon">mdi-cash</v-icon>
            <span class="section-title">{{ translate('bounties') }}</span>
          </div>

          <div class="control-grid">
            <div class="control-item">
              <v-btn
                variant="flat"
                color="primary"
                density="compact"
                class="action-btn"
                :loading="loading"
                @click="rerollBounties"
              >
                <v-icon start size="16">mdi-refresh</v-icon>
                {{ translate('reroll_bounties') }}
              </v-btn>
            </div>
          </div>
        </div>
      </div>
    </v-card>
  </div>
</template>

<script setup lang="ts">
import api from "@/api/axios";
import { useGlobalStore } from "@/store/global";
import { ref } from "vue";
import { onMounted } from "vue";
import { translate } from "@/helpers/translate";

const settings = ref({
  autoHostIsEnabled: false,
  hostingIsEnabled: false,
})

const globalStore = useGlobalStore();
const loading = ref(false)

const updateAutoHost = async () => {
  try {
    loading.value = true;
    await api.post("UiToggleAutoHost");
  } catch (error) {
    console.error('Failed to toggle auto-host:', error);
  } finally {
    loading.value = false;
  }
};

const updateHosting = async () => {
  try {
    loading.value = true;
    await api.post("UiToggleHosting");
  } catch (error) {
    console.error('Failed to toggle hosting:', error);
  } finally {
    loading.value = false;
  }
};

const rerollBounties = async () => {
  try {
    loading.value = true;
    await api.post('UIRerollBounties');
  } catch (error) {
    console.error('Failed to reroll bounties:', error);
  } finally {
    loading.value = false;
  }
};

const newAutoHost = async () => {
  try {
    loading.value = true;
    await api.post('UINewAutoHost');
  } catch (error) {
    console.error('Failed to create new auto-host:', error);
  } finally {
    loading.value = false;
  }
};

const getSettings = async () => {
  try {
    loading.value = true;
    const res = await api.post("UiGetAdminData");
    settings.value = res.data;
  } catch (error) {
    console.error('Failed to fetch admin data:', error);
  } finally {
    loading.value = false;
  }
};

onMounted(() => {
  getSettings();
});
</script>

<style scoped lang="scss">
@use "@/styles/variables" as v;

.admin-page {
  height: 100%;
  padding: 1rem;
  background: linear-gradient(to bottom right, rgba(20, 20, 20, 0.95), rgba(15, 15, 15, 0.9));
}

.admin-card {
  height: 100%;
  background: rgba(30, 30, 30, 0.6) !important;
  border: 1px solid rgba(255, 255, 255, 0.05);
  overflow: hidden;
}

.admin-header {
  padding: 1rem;
  background: rgba(0, 0, 0, 0.2);
  border-bottom: 1px solid rgba(255, 255, 255, 0.05);
}

.header-content {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.header-title {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 1.25rem;
  font-weight: 600;
  color: rgba(255, 255, 255, 0.95);
  margin: 0;
}

.header-subtitle {
  font-size: 0.875rem;
  color: rgba(255, 255, 255, 0.5);
  margin-left: 1.75rem;
}

.admin-content {
  padding: 1rem;
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.control-section {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.section-header {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding-bottom: 0.5rem;
  border-bottom: 1px solid rgba(255, 255, 255, 0.05);
}

.section-title {
  font-size: 0.875rem;
  font-weight: 600;
  color: rgba(255, 255, 255, 0.8);
  letter-spacing: 0.5px;
  text-transform: uppercase;
}

.control-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 0.75rem;
  padding: 0 0.5rem;
}

.control-item {
  background: rgba(255, 255, 255, 0.03);
  border-radius: 4px;
  transition: all 0.2s ease;

  &:hover {
    background: rgba(255, 255, 255, 0.05);
  }

  .control-switch {
    width: 100%;
    margin: 0;
    padding: 0.5rem 0.75rem;
  }

  .action-btn {
    width: 100%;
    height: 36px;
    letter-spacing: 0.25px;
    font-weight: 500;
    font-size: 0.8125rem;
    text-transform: none;
    background: linear-gradient(45deg, rgba(v.$primary-color, 0.9), rgba(v.$primary-color, 0.8));

    &:hover:not(:disabled) {
      background: linear-gradient(45deg, rgba(v.$primary-color, 1), rgba(v.$primary-color, 0.9));
    }

    &:disabled {
      opacity: 0.5;
    }
  }
}

.switch-content {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.5rem;
  width: 100%;
}

.switch-label {
  font-size: 0.8125rem;
  font-weight: 500;
  color: rgba(255, 255, 255, 0.9);
  letter-spacing: 0.25px;
}

:deep(.v-switch) {
  .v-selection-control {
    min-height: unset;
  }

  .v-selection-control__wrapper {
    width: 34px;
    height: 18px;
  }

  .v-selection-control__input {
    width: 34px;
    height: 18px;
    opacity: 0.7;

    &:hover {
      opacity: 1;
    }
  }
}
</style>

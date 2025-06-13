<template>
  <div class="settings-page">
    <v-card class="settings-card" flat>
      <!-- Header -->
      <div class="settings-header">
        <div class="header-content">
          <h2 class="header-title">
            <v-icon start size="20">mdi-cog</v-icon>
            {{ translate('settings') }}
          </h2>
          <span class="header-subtitle">{{ translate('configure_racing_preferences') }}</span>
        </div>
      </div>

      <!-- Settings Content -->
      <div class="settings-content">
        <!-- Racing Options -->
        <div class="settings-section">
          <div class="section-header">
            <v-icon size="18" color="primary" class="section-icon">mdi-map-marker-path</v-icon>
            <span class="section-title">{{ translate('racing_options') }}</span>
          </div>

          <div class="settings-grid">
            <!-- GPS Settings -->
            <div class="setting-item">
              <v-switch
                v-model="settings.ShowGpsRoute"
                color="primary"
                density="compact"
                hide-details
                :ripple="false"
                class="setting-switch"
                @change="updateSetting('ShowGpsRoute')"
              >
                <template v-slot:label>
                  <div class="switch-content">
                    <span class="switch-label">{{ translate('show_gps') }}</span>
                    <v-icon size="16">mdi-map-marker-distance</v-icon>
                  </div>
                </template>
              </v-switch>
            </div>

            <div class="setting-item">
              <v-switch
                v-model="settings.IgnoreRoadsForGps"
                color="primary"
                density="compact"
                hide-details
                :ripple="false"
                class="setting-switch"
                @change="updateSetting('IgnoreRoadsForGps')"
              >
                <template v-slot:label>
                  <div class="switch-content">
                    <span class="switch-label">{{ translate('ignore_roads') }}</span>
                    <v-icon size="16">mdi-road-variant</v-icon>
                  </div>
                </template>
              </v-switch>
            </div>

            <div class="setting-item">
              <v-switch
                v-model="settings.UseUglyWaypoint"
                color="primary"
                density="compact"
                hide-details
                :ripple="false"
                class="setting-switch"
                @change="updateSetting('UseUglyWaypoint')"
              >
                <template v-slot:label>
                  <div class="switch-content">
                    <span class="switch-label">{{ translate('base_wps') }}</span>
                    <v-icon size="16">mdi-map-marker</v-icon>
                  </div>
                </template>
              </v-switch>
            </div>

            <div class="setting-item">
              <v-switch
                v-model="settings.CheckDistance"
                color="primary"
                density="compact"
                hide-details
                :ripple="false"
                class="setting-switch"
                @change="updateSetting('CheckDistance')"
              >
                <template v-slot:label>
                  <div class="switch-content">
                    <span class="switch-label">{{ translate('distance_check') }}</span>
                    <v-tooltip location="top" :text="translate('distance_info')">
                      <template v-slot:activator="{ props }">
                        <v-icon v-bind="props" size="16">mdi-ruler</v-icon>
                      </template>
                    </v-tooltip>
                  </div>
                </template>
              </v-switch>
            </div>

            <div class="setting-item">
              <v-switch
                v-model="settings.UseDrawTextWaypoint"
                color="primary"
                density="compact"
                hide-details
                :ripple="false"
                class="setting-switch"
                @change="updateSetting('UseDrawTextWaypoint')"
              >
                <template v-slot:label>
                  <div class="switch-content">
                    <span class="switch-label">{{ translate('pillar_columns') }}</span>
                    <v-tooltip location="top" :text="translate('distance_info')">
                      <template v-slot:activator="{ props }">
                        <v-icon v-bind="props" size="16">mdi-pillar</v-icon>
                      </template>
                    </v-tooltip>
                  </div>
                </template>
              </v-switch>
            </div>
          </div>
        </div>

        <!-- User Settings -->
        <div class="settings-section">
          <div class="section-header">
            <v-icon size="18" color="primary" class="section-icon">mdi-account-cog</v-icon>
            <span class="section-title">{{ translate('user_settings') }}</span>
          </div>

          <div class="user-settings">
            <v-select
              v-model="racerName"
              :items="globalStore.baseData.data.racerNames"
              item-value="racername"
              item-title="racername"
              :label="translate('user')"
              :loading="loading"
              :disabled="globalStore.activeHudData.InRace"
              color="primary"
              density="compact"
              hide-details
              variant="outlined"
              class="racer-select"
              @update:model-value="updateRacerName()"
            >
              <template v-slot:prepend>
                <v-icon size="16">mdi-account</v-icon>
              </template>
              <template v-slot:append>
                <v-progress-circular
                  v-if="loading"
                  indeterminate
                  size="16"
                  width="2"
                  color="primary"
                />
              </template>
            </v-select>
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
import { Settings } from "@/store/types";
import { Ref } from "vue";
import { getBaseData } from "@/helpers/getBaseData";
import { translate } from "@/helpers/translate";

const globalStore = useGlobalStore();
const settings: Ref<Settings> = ref({
    ShowGpsRoute: false,
    IgnoreRoadsForGps: false,
    UseUglyWaypoint: false,
    CheckDistance: false,
    UseDrawTextWaypoint: false,
})
const racerName = ref(globalStore.baseData.data.currentRacerName)
const loading = ref(false)
const updateSetting = async (setting: string) => {
  await api.post("UiUpdateSettings", JSON.stringify({setting: setting, value: settings.value[setting as unknown as keyof typeof settings.value] }));
};

const updateRacerName = async () => {
  loading.value = true
  await api.post("UiChangeRacerName", JSON.stringify(racerName.value));
  setTimeout(() => {
    getBaseData()
    loading.value = false
  }, 1000);
};

const getSettings = async () => {
  const res = await api.post("UiGetSettings");
  settings.value = res.data
};

onMounted(() => {
  getSettings();
});
</script>

<style scoped lang="scss">
@use "@/styles/variables" as v;

.settings-page {
  height: 100%;
  padding: 1rem;
  background: linear-gradient(to bottom right, rgba(20, 20, 20, 0.95), rgba(15, 15, 15, 0.9));
}

.settings-card {
  height: 100%;
  background: rgba(30, 30, 30, 0.6) !important;
  border: 1px solid rgba(255, 255, 255, 0.05);
  overflow: hidden;
}

.settings-header {
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

.settings-content {
  padding: 1rem;
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.settings-section {
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

.settings-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 0.5rem;
  padding: 0 0.5rem;
}

.setting-item {
  background: rgba(255, 255, 255, 0.03);
  border-radius: 4px;
  transition: all 0.2s ease;

  &:hover {
    background: rgba(255, 255, 255, 0.05);
  }

  .setting-switch {
    width: 100%;
    margin: 0;
    padding: 0.5rem 0.75rem;
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

.user-settings {
  padding: 0 0.5rem;

  .racer-select {
    max-width: 300px;

    :deep(.v-field) {
      border-radius: 4px;
      background: rgba(255, 255, 255, 0.03);
      min-height: 36px;

      &:hover {
        background: rgba(255, 255, 255, 0.05);
      }

      &.v-field--focused {
        background: rgba(v.$primary-color, 0.1);
      }
    }
  }
}
</style>

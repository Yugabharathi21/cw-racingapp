<template>
  <div id="ResultsPage" class="results-page">
    <!-- Page Header -->
    <div class="page-header">
      <h2 class="page-title">{{ translate('race_statistics') }}</h2>
      <v-tabs 
        v-model="tab" 
        class="stats-tabs"
        slider-color="primary"
        selected-class="tab-selected"
        height="44"
      >
        <v-tab 
          value="results"
          class="stats-tab"
        >
          <v-icon start size="18">mdi-flag-checkered</v-icon>
          {{ translate('race_results') }}
        </v-tab>
        <v-tab 
          value="crewRank"
          class="stats-tab"
        >
          <v-icon start size="18">mdi-account-group</v-icon>
          {{ translate('crew_rankings') }}
        </v-tab>
        <v-tab 
          value="racerRank"
          class="stats-tab"
        >
          <v-icon start size="18">mdi-account-star</v-icon>
          {{ translate('racer_rankings') }}
        </v-tab>
        <v-tab 
          value="records"
          class="stats-tab"
        >
          <v-icon start size="18">mdi-trophy</v-icon>
          {{ translate('track_records') }}
        </v-tab>
      </v-tabs>
    </div>

    <!-- Tab Content -->
    <v-window 
      v-model="tab"
      class="content-window"
    >
      <v-window-item 
        value="results" 
        class="window-content"
      >
        <div class="content-wrapper">
          <RaceResults 
            v-if="allResults" 
            :allResults="allResults"
            class="content-component"
          />
          <InfoText 
            v-else 
            :title="translate('no_data')"
            :subtitle="translate('no_race_results_yet')"
            icon="mdi-timer-sand-empty"
          />
        </div>
      </v-window-item>

      <v-window-item 
        value="crewRank" 
        class="window-content"
      >
        <div class="content-wrapper">
          <CrewTable class="content-component" />
        </div>
      </v-window-item>

      <v-window-item 
        value="racerRank" 
        class="window-content"
      >
        <div class="content-wrapper">
          <RacersTable class="content-component" />
        </div>
      </v-window-item>

      <v-window-item 
        value="records" 
        class="window-content"
      >
        <div class="content-wrapper">
          <RaceRecords 
            v-if="allRecords" 
            :allRecords="allRecords"
            class="content-component"
          />
          <InfoText 
            v-else 
            :title="translate('no_data')"
            :subtitle="translate('no_track_records_yet')"
            icon="mdi-trophy-broken"
          />
        </div>
      </v-window-item>
    </v-window>

    <!-- Loading Overlay -->
    <div v-if="loading" class="loading-overlay">
      <v-progress-circular
        indeterminate
        color="primary"
        size="32"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import api from "@/api/axios";
import { useGlobalStore } from "@/store/global";
import { Ref, onMounted, ref, watch } from "vue";
import RaceResults from "../components/RaceResults.vue";
import { Track, ResultData } from "@/store/types";
import RaceRecords from "../components/RaceRecords.vue";
import InfoText from "../components/InfoText.vue";
import CrewTable from "../components/CrewTable.vue";
import RacersTable from "../components/RacersTable.vue";
import { translate } from "@/helpers/translate";

const allResults: Ref<ResultData | undefined> = ref(undefined);
const allRecords: Ref<Track[] | undefined> = ref(undefined);
const globalStore = useGlobalStore();
const tab = ref(globalStore.currentTab);
const loading = ref(false);

const getResults = async () => {
  loading.value = true;
  try {
    const response = await api.post("UiGetRacingResults");
    if (response.data) allResults.value = response.data as ResultData;
  } catch (error) {
    console.error('Failed to fetch race results:', error);
  } finally {
    loading.value = false;
  }
};

const getRecords = async () => {
  loading.value = true;
  try {
    const response = await api.post("UiGetRaces");
    if (response.data) allRecords.value = response.data;
  } catch (error) {
    console.error('Failed to fetch race records:', error);
  } finally {
    loading.value = false;
  }
};

// Watch for tab changes to update global store
watch(tab, (newTab) => {
  globalStore.currentTab = newTab;
});

onMounted(() => {
  getResults();
  getRecords();
});
</script>

<style scoped lang="scss">
@use "@/styles/variables" as v;

.results-page {
  display: flex;
  flex-direction: column;
  height: 100%;
  position: relative;
  background: linear-gradient(165deg, rgba(18, 18, 20, 0.95), rgba(13, 13, 15, 0.9));
  border-radius: 8px;
  overflow: hidden;
}

.page-header {
  padding: 1.25rem 1.25rem 0;
  background: linear-gradient(to bottom, rgba(25, 25, 28, 0.95), rgba(20, 20, 23, 0.9));
  border-bottom: 1px solid rgba(255, 255, 255, 0.06);
}

.page-title {
  font-size: 1.5rem;
  font-weight: 600;
  color: rgba(255, 255, 255, 0.95);
  margin: 0 0 1rem;
  letter-spacing: 0.5px;
}

.stats-tabs {
  margin: 0 -1.25rem;
  padding: 0 1.25rem;

  :deep(.v-tab) {
    font-size: 0.875rem;
    font-weight: 500;
    letter-spacing: 0.25px;
    text-transform: none;
    min-width: 120px;
    padding: 0 1rem;
    color: rgba(255, 255, 255, 0.7);
    border-bottom: 2px solid transparent;

    &:hover {
      color: rgba(255, 255, 255, 0.9);
    }

    &.tab-selected {
      color: v.$primary-color;
      font-weight: 600;
    }

    .v-icon {
      margin-right: 0.5rem;
      opacity: 0.9;
    }
  }

  :deep(.v-slide-group__content) {
    gap: 0.5rem;
  }
}

.content-window {
  flex: 1;
  min-height: 0;
  position: relative;
}

.window-content {
  height: 100%;
  padding: 1.25rem;
}

.content-wrapper {
  height: 100%;
  background: rgba(255, 255, 255, 0.02);
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.04);
  backdrop-filter: blur(10px);
  overflow: hidden;
}

.content-component {
  height: 100%;
  padding: 1rem;
}

.loading-overlay {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(13, 13, 15, 0.7);
  backdrop-filter: blur(4px);
  z-index: 100;
}
</style>

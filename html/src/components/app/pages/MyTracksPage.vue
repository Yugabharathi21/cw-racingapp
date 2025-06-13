<template>
  <div id="MyTracksPage" class="modern-page">
    <!-- Tab Navigation -->
    <div class="tab-container">
      <div class="tab-wrapper">
        <div
          v-for="tabItem in tabs"
          :key="tabItem.value"
          class="tab-item"
          :class="{ 'active': tab === tabItem.value }"
          @click="tab = tabItem.value"
        >
          <v-icon>{{ tabItem.icon }}</v-icon>
          <span>{{ translate(tabItem.label) }}</span>
          <div class="tab-highlight"></div>
        </div>
      </div>
    </div>

    <!-- Content Area -->
    <v-window v-model="tab" class="content-container">
      <!-- My Tracks Tab -->
      <v-window-item value="myTracks" class="tab-content">
        <div class="page-header">
          <div class="header-content">
            <h2>{{ translate('my_tracks') }}</h2>
            <div class="search-field">
              <v-text-field
                v-model="search"
                :placeholder="translate('search_dot')"
                density="comfortable"
                hide-details
                prepend-inner-icon="mdi-magnify"
                variant="outlined"
                bg-color="rgba(0, 0, 0, 0.2)"
              ></v-text-field>
            </div>
          </div>
        </div>

        <div class="tracks-section" v-if="filtereredTracks && filtereredTracks.length > 0">
          <div class="tracks-grid">
            <MyTrackCard
              v-for="track in filtereredTracks"
              :key="track.TrackId"
              @update="getMyTracks"
              :track="track"
            ></MyTrackCard>
          </div>
        </div>
        <InfoText :title="translate('no_data')" v-else />

        <div class="tracks-section admin" v-if="globalStore.baseData.data.auth.controlAll">
          <div class="section-header">
            <h3>{{ translate('tracks') }}</h3>
            <span class="track-count">{{ filteredAllTracks?.length || 0 }}</span>
          </div>
          <div class="tracks-grid">
            <MyTrackCard
              v-for="track in filteredAllTracks"
              :key="track.TrackId"
              @update="getMyTracks"
              :track="track"
            ></MyTrackCard>
          </div>
        </div>
      </v-window-item>

      <!-- Create Track Tab -->
      <v-window-item value="create" class="tab-content">
        <div class="create-section">
          <!-- Editor Creation Card -->
          <div class="creation-card">
            <div class="card-header">
              <v-icon size="24" color="primary">mdi-map-marker-path</v-icon>
              <h3>{{ translate('create_with_editor') }}</h3>
            </div>
            <div class="card-content">
              <v-text-field
                v-model="trackName"
                :placeholder="translate('track_name')"
                density="comfortable"
                variant="outlined"
                bg-color="rgba(0, 0, 0, 0.2)"
                hide-details
                prepend-inner-icon="mdi-format-title"
              ></v-text-field>
              <v-btn
                block
                color="primary"
                height="48"
                @click="openRaceEditor()"
              >
                <v-icon start>mdi-play-circle-outline</v-icon>
                {{ translate('start_editor') }}
              </v-btn>
            </div>
          </div>

          <!-- Share Creation Card -->
          <div class="creation-card">
            <div class="card-header">
              <v-icon size="24" color="primary">mdi-share-variant</v-icon>
              <h3>{{ translate('create_with_share') }}</h3>
            </div>
            <div class="card-content">
              <v-text-field
                v-model="trackNameShare"
                :placeholder="translate('track_name')"
                density="comfortable"
                variant="outlined"
                bg-color="rgba(0, 0, 0, 0.2)"
                hide-details
                prepend-inner-icon="mdi-format-title"
              ></v-text-field>
              <v-textarea
                v-model="checkpoints"
                placeholder="Paste Checkpoints Here"
                density="comfortable"
                variant="outlined"
                bg-color="rgba(0, 0, 0, 0.2)"
                hide-details
                rows="4"
                auto-grow
              ></v-textarea>
              <v-btn
                block
                color="primary"
                height="48"
                @click="createRaceFromShare()"
              >
                <v-icon start>mdi-plus-circle-outline</v-icon>
                {{ translate('create_track') }}
              </v-btn>
            </div>
          </div>
        </div>
      </v-window-item>
    </v-window>
  </div>
</template>

<script setup lang="ts">
import { useGlobalStore } from "@/store/global";
import MyTrackCard from "../items/MyTrackCard.vue";
import { onMounted, ref } from "vue";
import api from "@/api/axios";
import { Ref } from "vue";
import { Track } from "@/store/types";
import { computed } from "vue";
import InfoText from "../components/InfoText.vue";
import { closeApp } from "@/helpers/closeApp";
import { translate } from "@/helpers/translate";

const globalStore = useGlobalStore();
const search = ref("");
const tab = ref(globalStore.currentPage);
const trackName: Ref<string | undefined> = ref(undefined);
const trackNameShare: Ref<string | undefined> = ref(undefined);
const tracks: Ref<Track[] | undefined> = ref(undefined);
const checkpoints: Ref<any | undefined> = ref(undefined);
const filtereredTracks = computed(() => {
  let fTracks = tracks?.value?.filter(
    (track) => track.CreatorName === globalStore.baseData.data.currentRacerName
  );
  if (fTracks && search.value !== "")
    fTracks = fTracks.filter(
      (track) =>
        track.RaceName.toLowerCase().includes(search.value.toLowerCase()) ||
        track.TrackId.toLowerCase().includes(search.value.toLowerCase())
    );

  return fTracks;
});

const filteredAllTracks = computed(() => {
  let fTracks = tracks.value
  if (fTracks && search.value !== "")
    fTracks = fTracks.filter(
      (track) =>
        track.RaceName.toLowerCase().includes(search.value.toLowerCase()) ||
        track.TrackId.toLowerCase().includes(search.value.toLowerCase())
    );

  return fTracks;
})

const getMyTracks = async () => {
  const response = await api.post("UiGetMyTracks");
  if (response.data) tracks.value = Object.values(response.data);
};

const openRaceEditor = async () => {
  const response = await api.post(
    "UiCreateTrack",
    JSON.stringify({ name: trackName.value })
  );
  if (response.data) closeApp();
};

const createRaceFromShare = async () => {
  const response = await api.post(
    "UiCreateTrack",
    JSON.stringify({
      name: trackNameShare.value,
      checkpoints: checkpoints.value,
    })
  );
  if (response.data) closeApp();
};

onMounted(() => {
  getMyTracks();
});

const tabs = [
  { value: "myTracks", label: "my_tracks", icon: "mdi-map-marker-multiple" },
  { value: "create", label: "create_track", icon: "mdi-plus-circle" },
];
</script>

<style scoped lang="scss">
@use "@/styles/variables" as v;

.modern-page {
  display: flex;
  flex-direction: column;
  gap: 2rem;
  padding: 1.5rem;
  max-width: 1400px;
  margin: 0 auto;
  width: 100%;

  @media (max-width: 768px) {
    padding: 1rem;
    gap: 1.5rem;
  }
}

/* Tab Navigation
   ========================================================================== */
.tab-container {
  position: relative;
  background: rgba(0, 0, 0, 0.4);
  border-radius: 16px;
  padding: 0.5rem;
  box-shadow: 0 4px 24px rgba(0, 0, 0, 0.2);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.05);
}

.tab-wrapper {
  display: flex;
  gap: 0.25rem;
  position: relative;
  z-index: 1;
}

.tab-item {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.75rem 1.25rem;
  border-radius: 12px;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  position: relative;
  color: rgba(255, 255, 255, 0.7);
  flex: 1;
  justify-content: center;
  overflow: hidden;

  &::before {
    content: '';
    position: absolute;
    inset: 0;
    background: linear-gradient(120deg, transparent, rgba(255, 255, 255, 0.05), transparent);
    transform: translateX(-100%);
    transition: transform 0.6s ease;
  }

  .v-icon {
    opacity: 0.7;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    font-size: 1.2rem;
  }

  span {
    font-weight: 500;
    font-size: 0.95rem;
  }

  &:hover {
    background: rgba(255, 255, 255, 0.08);
    color: white;

    &::before {
      transform: translateX(100%);
    }

    .v-icon {
      opacity: 1;
      transform: scale(1.1) translateY(-1px);
    }
  }

  &.active {
    background: rgba(v.$primary-color, 0.15);
    color: v.$primary-color-light;

    .v-icon {
      color: v.$primary-color-light;
      opacity: 1;
      transform: scale(1.1);
    }

    .tab-highlight {
      transform: scaleX(1);
    }

    &::before {
      background: linear-gradient(120deg, transparent, rgba(v.$primary-color, 0.1), transparent);
    }
  }

  .tab-highlight {
    position: absolute;
    bottom: -0.5rem;
    left: 0.5rem;
    right: 0.5rem;
    height: 2px;
    background: v.$primary-color;
    transform: scaleX(0);
    transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    border-radius: 2px;
    box-shadow: 0 0 15px rgba(v.$primary-color, 0.4);
  }
}

/* Content Area
   ========================================================================== */
.content-container {
  background: rgba(0, 0, 0, 0.4);
  border-radius: 16px;
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.05);
  box-shadow: 0 4px 24px rgba(0, 0, 0, 0.2);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  overflow: hidden;

  &:hover {
    transform: translateY(-2px);
    box-shadow: 0 6px 32px rgba(0, 0, 0, 0.3);
  }
}

.tab-content {
  padding: 2rem;

  @media (max-width: 768px) {
    padding: 1.5rem;
  }
}

/* Page Header
   ========================================================================== */
.page-header {
  margin-bottom: 2rem;

  .header-content {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 2rem;

    @media (max-width: 768px) {
      flex-direction: column;
      align-items: stretch;
      gap: 1rem;
    }

    h2 {
      font-size: 1.75rem;
      font-weight: 600;
      color: white;
      letter-spacing: -0.5px;
    }

    .search-field {
      width: 100%;
      max-width: 320px;

      @media (max-width: 768px) {
        max-width: none;
      }
    }
  }
}

/* Tracks Section
   ========================================================================== */
.tracks-section {
  margin-bottom: 3rem;

  &.admin {
    padding-top: 2rem;
    border-top: 1px solid rgba(255, 255, 255, 0.1);
  }
}

.section-header {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-bottom: 1.5rem;
  position: relative;

  &::after {
    content: '';
    position: absolute;
    bottom: -0.75rem;
    left: 0;
    right: 0;
    height: 1px;
    background: linear-gradient(90deg, rgba(v.$primary-color, 0.3), transparent);
  }

  h3 {
    font-size: 1.4rem;
    font-weight: 600;
    color: white;
    letter-spacing: -0.5px;
  }

  .track-count {
    background: rgba(v.$primary-color, 0.15);
    color: v.$primary-color-light;
    padding: 0.3rem 0.8rem;
    border-radius: 20px;
    font-size: 0.9rem;
    font-weight: 600;
    box-shadow: 0 0 15px rgba(v.$primary-color, 0.2);
  }
}

.tracks-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(320px, 1fr));
  gap: 1.5rem;

  @media (max-width: 768px) {
    grid-template-columns: 1fr;
  }
}

/* Create Section
   ========================================================================== */
.create-section {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(400px, 1fr));
  gap: 2rem;

  @media (max-width: 768px) {
    grid-template-columns: 1fr;
  }
}

.creation-card {
  background: rgba(0, 0, 0, 0.3);
  border-radius: 16px;
  border: 1px solid rgba(255, 255, 255, 0.05);
  overflow: hidden;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  position: relative;

  &::before {
    content: '';
    position: absolute;
    inset: 0;
    background: linear-gradient(120deg, transparent, rgba(255, 255, 255, 0.03), transparent);
    transform: translateX(-100%);
    transition: transform 0.6s ease;
  }

  &:hover {
    transform: translateY(-3px);
    border-color: rgba(v.$primary-color, 0.2);
    box-shadow: 0 8px 32px rgba(0, 0, 0, 0.2);

    &::before {
      transform: translateX(100%);
    }

    .card-header {
      .v-icon {
        transform: scale(1.1);
      }
    }
  }

  .card-header {
    display: flex;
    align-items: center;
    gap: 1rem;
    padding: 1.25rem;
    background: rgba(255, 255, 255, 0.03);
    border-bottom: 1px solid rgba(255, 255, 255, 0.05);

    .v-icon {
      transition: transform 0.3s ease;
    }

    h3 {
      font-size: 1.2rem;
      font-weight: 600;
      color: white;
      letter-spacing: -0.3px;
    }
  }

  .card-content {
    padding: 1.5rem;
    display: flex;
    flex-direction: column;
    gap: 1.25rem;
  }
}

/* Input Fields Styling
   ========================================================================== */
:deep(.v-field.v-field) {
  border-radius: 12px;
  transition: all 0.3s ease;
  background: rgba(0, 0, 0, 0.2);

  &:hover {
    background: rgba(0, 0, 0, 0.3);
  }
}

:deep(.v-field--focused) {
  background: rgba(0, 0, 0, 0.4);
  border-color: rgba(v.$primary-color, 0.5);
}

:deep(.v-btn) {
  letter-spacing: 0.3px;
  font-weight: 600;
  text-transform: none;
  border-radius: 12px;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);

  &:hover {
    transform: translateY(-1px);
    box-shadow: 0 4px 20px rgba(v.$primary-color, 0.4);
  }
}
</style>

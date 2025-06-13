<template>
  <v-card 
    rounded="lg" 
    class="track-card" 
    :elevation="0"
  >
    <div class="card-wrapper">
      <!-- Track Info Section -->
      <div class="track-info">
        <div class="track-header">
          <!-- Title and Actions -->
          <div class="title-section">
            <div class="name-group">
              <span class="track-badge" v-tooltip="translate('track_id')">ID: {{ track.TrackId }}</span>
              <h3 class="track-name">{{ track.RaceName }}</h3>
              <div class="badge-group">
                <v-chip
                  v-if="track.Curated"
                  color="success"
                  size="x-small"
                  variant="flat"
                  density="compact"
                  class="status-chip"
                >
                  <v-icon start size="12">mdi-star</v-icon>
                  {{ translate('curated') }}
                </v-chip>
                <v-chip
                  v-if="track.Access?.race?.length > 0"
                  size="x-small"
                  color="info"
                  variant="flat"
                  density="compact"
                  class="status-chip"
                >
                  <v-icon start size="12">mdi-shield-lock</v-icon>
                  {{ track.Access.race.length }}
                </v-chip>
              </div>
            </div>
            <div class="quick-actions">
              <v-tooltip location="top" text="Preview">
                <template v-slot:activator="{ props }">
                  <v-btn
                    v-bind="props"
                    variant="text"
                    density="compact"
                    class="action-btn preview"
                    @click="showRace"
                  >
                    <v-icon size="16">mdi-eye</v-icon>
                  </v-btn>
                </template>
              </v-tooltip>
              <v-tooltip location="top" text="Copy Checkpoint Data">
                <template v-slot:activator="{ props }">
                  <v-btn
                    v-bind="props"
                    variant="text"
                    density="compact"
                    class="action-btn copy"
                    @click="copyToClipboard"
                  >
                    <v-icon size="16">mdi-content-copy</v-icon>
                  </v-btn>
                </template>
              </v-tooltip>
            </div>
          </div>

          <!-- Track Meta -->
          <div class="track-meta">
            <div class="meta-group">
              <span class="meta-item creator">
                <v-icon size="14" color="primary">mdi-account-star</v-icon>
                {{ track.CreatorName }}
              </span>
              <span v-if="track.Metadata?.description" class="meta-item description">
                {{ track.Metadata.description }}
              </span>
            </div>

            <!-- Track Stats -->
            <div class="stats-grid">
              <div class="stat-item">
                <v-icon size="14">mdi-ruler</v-icon>
                <span class="stat-value">{{ track.Distance }}m</span>
              </div>
              <div class="stat-item">
                <v-icon size="14">mdi-map-marker-path</v-icon>
                <span class="stat-value">{{ track.Checkpoints.length }} {{ translate('checkpoints') }}</span>
              </div>
              <div class="stat-item" v-if="track.Access?.race?.length > 0">
                <v-icon size="14">mdi-account-group</v-icon>
                <span class="stat-value">{{ track.Access.race.length }} {{ translate('racers') }}</span>
              </div>
            </div>
          </div>

          <!-- Action Menu -->
          <div class="action-menu">
            <div class="action-group primary">
              <v-btn
                variant="flat"
                color="primary"
                density="compact"
                class="main-action"
                @click="editRecords = true"
              >
                <v-icon start size="16">mdi-trophy</v-icon>
                {{ translate('view_records') }}
              </v-btn>

              <v-menu :close-on-content-click="false" location="bottom end">
                <template v-slot:activator="{ props }">
                  <v-btn
                    color="surface"
                    variant="flat"
                    density="compact"
                    class="menu-btn"
                    v-bind="props"
                  >
                    <v-icon size="16">mdi-dots-vertical</v-icon>
                  </v-btn>
                </template>
                
                <v-list density="compact" class="action-menu-list" nav>
                  <v-list-item
                    v-if="globalStore.baseData.data.auth.curateTracks"
                    @click="curateDialog = true"
                    prepend-icon="mdi-star-settings"
                    :title="translate('curation')"
                  />
                  <v-list-item
                    v-if="!track.Curated"
                    @click="editDialog = true"
                    prepend-icon="mdi-pencil"
                    :title="translate('edit_track')"
                  />
                  <v-divider/>
                  <v-list-item
                    @click="openEditMetadata"
                    prepend-icon="mdi-cog"
                    :title="translate('edit_settings')"
                  />
                  <v-list-item
                    @click="openEditAccess"
                    prepend-icon="mdi-account-cog"
                    :title="translate('edit_access')"
                  />
                  <v-divider/>
                  <v-list-item
                    @click="lbDialog = true"
                    prepend-icon="mdi-trash-can"
                    :title="translate('clear_lead')"
                    class="warning"
                  />
                  <v-list-item
                    @click="deleteDialog = true"
                    prepend-icon="mdi-delete"
                    :title="translate('delete_track')"
                    color="error"
                  />
                </v-list>
              </v-menu>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Dialogs -->
    <!-- Curation Dialog -->
    <v-dialog 
      attach=".app-container" 
      v-model="curateDialog"
      width="400"
      class="curation-dialog"
    >
      <v-card rounded="lg" class="dialog-card">
        <div class="dialog-header">
          <h3 class="dialog-title">
            {{ translate('handle_curation_for') }} {{ track.RaceName }}
          </h3>
          <v-btn
            variant="text"
            icon="mdi-close"
            size="small"
            @click="curateDialog = false"
          />
        </div>

        <v-card-actions class="dialog-actions">
          <v-spacer/>
          <v-btn
            color="error"
            variant="tonal"
            @click="setCuration(0)"
          >
            <v-icon start>mdi-star-off</v-icon>
            {{ translate('set_uncurated') }}
          </v-btn>
          <v-btn
            color="success"
            variant="flat"
            @click="setCuration(1)"
          >
            <v-icon start>mdi-star</v-icon>
            {{ translate('set_curated') }}
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Settings Dialog -->
    <v-dialog 
      attach=".app-container" 
      v-model="settingsDialog"
      width="500"
      class="settings-dialog"
    >
      <v-card rounded="lg" class="dialog-card">
        <div class="dialog-header">
          <h3 class="dialog-title">
            {{ translate('edit_settings') }}
            <v-chip
              size="small"
              color="primary"
              variant="flat"
              class="track-chip"
            >
              {{ track.RaceName }}
            </v-chip>
          </h3>
          <v-btn
            variant="text"
            icon="mdi-close"
            size="small"
            @click="settingsDialog = false"
          />
        </div>

        <v-card-text class="dialog-content">
          <v-textarea
            class="description-field"
            :maxlength="200"
            :counter="200"
            :label="translate('description')"
            :hint="translate('description_hint')"
            color="primary"
            density="comfortable"
            variant="outlined"
            v-model="settings.description"
          />
          
          <v-select
            :label="translate('race_type')"
            :items="trackTypes"
            item-value="value"
            item-title="label"
            color="primary"
            density="comfortable"
            variant="outlined"
            v-model="settings.raceType"
            hide-details
          />
        </v-card-text>

        <v-card-actions class="dialog-actions">
          <v-spacer/>
          <v-btn
            variant="tonal"
            @click="settingsDialog = false"
          >
            {{ translate('cancel') }}
          </v-btn>
          <v-btn
            color="success"
            variant="flat"
            @click="confirmSettings"
          >
            {{ translate('save') }}
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Access Dialog -->
    <v-dialog 
      attach=".app-container" 
      v-model="accessDialog"
      width="500"
      class="access-dialog"
    >
      <v-card rounded="lg" class="dialog-card">
        <div class="dialog-header">
          <h3 class="dialog-title">
            {{ translate('editing_access_for') }} {{ track.RaceName }}
          </h3>
          <v-btn
            variant="text"
            icon="mdi-close"
            size="small"
            @click="accessDialog = false"
          />
        </div>

        <v-card-text class="dialog-content">
          <v-text-field
            :label="translate('access_list')"
            v-model="access.race"
            :hint="translate('editing_access_info')"
            persistent-hint
            color="primary"
            density="comfortable"
            variant="outlined"
          />
        </v-card-text>

        <v-card-actions class="dialog-actions">
          <v-spacer/>
          <v-btn
            variant="tonal"
            @click="accessDialog = false"
          >
            {{ translate('cancel') }}
          </v-btn>
          <v-btn
            color="success"
            variant="flat"
            @click="editAccess"
          >
            {{ translate('save') }}
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Records Dialog -->
    <v-dialog 
      attach=".app-container" 
      v-model="editRecords"
      width="600"
      class="records-dialog"
    >
      <v-card rounded="lg" class="dialog-card">
        <div class="dialog-header">
          <div class="header-content">
            <h3 class="dialog-title">
              {{ translate('view_records') }} {{ track.RaceName }}
            </h3>
            <p class="dialog-subtitle">
              {{ translate('view_records_modal_desc') }}
            </p>
          </div>
          <v-btn
            variant="text"
            icon="mdi-close"
            size="small"
            @click="editRecords = false"
          />
        </div>

        <v-card-text class="dialog-content">
          <v-alert
            type="warning"
            variant="tonal"
            border="start"
            density="comfortable"
            class="warning-alert"
          >
            {{ translate('changes_are_permanent') }}
          </v-alert>

          <v-list class="records-list">
            <v-list-item
              v-for="(record, i) in track.Records"
              :key="i"
              :title="record.Holder + ' | ' + msToHMS(record.Time)"
              :subtitle="record.Vehicle + ' | ' + record.Class"
            >
              <template v-slot:append>
                <v-btn
                  variant="text"
                  color="error"
                  density="comfortable"
                  @click="openConfirmRecordModal(record)"
                >
                  <v-icon start>mdi-delete</v-icon>
                  {{ translate('remove_record') }}
                </v-btn>
              </template>
            </v-list-item>
          </v-list>
        </v-card-text>
      </v-card>

      <!-- Record Removal Confirmation -->
      <v-dialog
        attach=".app-container"
        v-model="confirmRecordRemovalDialog"
        width="400"
        class="record-removal-dialog"
      >
        <v-card rounded="lg" class="dialog-card">
          <div class="dialog-header">
            <h3 class="dialog-title">
              {{ translate('confirm_record_removal') }}
            </h3>
            <v-btn
              variant="text"
              icon="mdi-close"
              size="small"
              @click="resetRecordRemoval"
            />
          </div>

          <v-card-text v-if="selectedRecord" class="dialog-content">
            <div class="record-details">
              <div class="detail-item">
                <span class="label">{{ translate('racer_name') }}:</span>
                <span class="value">{{ selectedRecord.Holder }}</span>
              </div>
              <div class="detail-item">
                <span class="label">{{ translate('best_lap') }}:</span>
                <span class="value">{{ selectedRecord.Time }}</span>
              </div>
              <div class="detail-item">
                <span class="label">{{ translate('vehicle') }}:</span>
                <span class="value">{{ selectedRecord.Vehicle }}</span>
              </div>
              <div class="detail-item">
                <span class="label">{{ translate('class') }}:</span>
                <span class="value">{{ selectedRecord.Class }}</span>
              </div>
              <div class="detail-item">
                <span class="label">{{ translate('race_type') }}:</span>
                <span class="value">{{ selectedRecord.RaceType }}</span>
              </div>
              <div class="detail-item">
                <span class="label">{{ translate('reversed') }}:</span>
                <span class="value">{{ selectedRecord.Reversed }}</span>
              </div>
            </div>
          </v-card-text>

          <v-card-actions class="dialog-actions">
            <v-spacer/>
            <v-btn
              variant="tonal"
              @click="resetRecordRemoval"
            >
              {{ translate('cancel') }}
            </v-btn>
            <v-btn
              color="error"
              variant="flat"
              :loading="loadingRecordRemoval"
              @click="handleRemoveRecord"
            >
              {{ translate('confirm') }}
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-dialog>

    <!-- Delete Track Dialog -->
    <v-dialog 
      attach=".app-container" 
      v-model="deleteDialog"
      width="400"
      class="delete-dialog"
    >
      <v-card rounded="lg" class="dialog-card">
        <div class="dialog-header">
          <h3 class="dialog-title">
            {{ translate('delete_track') }} {{ track.RaceName }}?
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
            @click="deleteTrack"
          >
            {{ translate('confirm') }}
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Edit Track Dialog -->
    <v-dialog 
      attach=".app-container" 
      v-model="editDialog"
      width="400"
      class="edit-dialog"
    >
      <v-card rounded="lg" class="dialog-card">
        <div class="dialog-header">
          <h3 class="dialog-title">
            {{ translate('open_track_editor_for') }} {{ track.RaceName }}?
          </h3>
          <v-btn
            variant="text"
            icon="mdi-close"
            size="small"
            @click="editDialog = false"
          />
        </div>

        <v-card-actions class="dialog-actions">
          <v-spacer/>
          <v-btn
            variant="tonal"
            @click="editDialog = false"
          >
            {{ translate('cancel') }}
          </v-btn>
          <v-btn
            color="success"
            variant="flat"
            @click="editTrack"
          >
            {{ translate('confirm') }}
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Clear Leaderboard Dialog -->
    <v-dialog 
      attach=".app-container" 
      v-model="lbDialog"
      width="400"
      class="leaderboard-dialog"
    >
      <v-card rounded="lg" class="dialog-card">
        <div class="dialog-header">
          <h3 class="dialog-title">
            {{ translate('clear_lead_for') }} {{ track.RaceName }}?
          </h3>
          <v-btn
            variant="text"
            icon="mdi-close"
            size="small"
            @click="lbDialog = false"
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
            @click="lbDialog = false"
          >
            {{ translate('cancel') }}
          </v-btn>
          <v-btn
            color="error"
            variant="flat"
            @click="clearLB"
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
import { closeApp } from "@/helpers/closeApp";
import { Track, TrackMetadata, TrackRecord } from "@/store/types";
import { Ref, ref } from "vue";
import { translate } from "@/helpers/translate";
import { useGlobalStore } from "@/store/global";
import { msToHMS } from "@/helpers/msToHMS";

const trackTypes = [
  { value: 'any', label: translate('any') },
  { value: 'circuit_only', label: translate('circuit_only') },
  { value: 'sprint_only', label: translate('sprint_only') },
]

const globalStore = useGlobalStore();

const props = defineProps<{
  track: Track;
}>();
const emit = defineEmits(['update'])

const curateDialog = ref(false);
const lbDialog = ref(false);
const editDialog = ref(false);
const accessDialog = ref(false);
const settingsDialog = ref(false);
const editRecords = ref(false);

const confirmRecordRemovalDialog = ref(false);
const selectedRecord: Ref<TrackRecord | undefined> = ref(undefined)
const loadingRecordRemoval = ref(false)

const openConfirmRecordModal = (record: TrackRecord) => {
  selectedRecord.value = record
  confirmRecordRemovalDialog.value = true
}

const resetRecordRemoval = () => {
  selectedRecord.value = undefined
  confirmRecordRemovalDialog.value = false

}
const handleRemoveRecord = async () => {
  loadingRecordRemoval.value = true  
  if (selectedRecord.value) {
    const response = await api.post('UiRemoveRecord',  JSON.stringify({ trackId: props.track.TrackId, record: selectedRecord.value }))
    if (response) {
      emit('update')      
      resetRecordRemoval()
      loadingRecordRemoval.value = false
    }
  }
  
  
}

const settings: Ref<TrackMetadata> = ref({
  description: undefined,
  raceType: undefined,
});
const deleteDialog = ref(false);
const access: any = ref(undefined)

const copyToClipboard = () => {
  const el = document.createElement("textarea");
  el.value = JSON.stringify(props.track.Checkpoints);
  document.body.appendChild(el);
  el.select();
  document.execCommand("copy");
  document.body.removeChild(el);
};

const clearLB = () => {
    api.post('UiClearLeaderboard', JSON.stringify({ RaceName: props.track.RaceName, TrackId: props.track.TrackId }))
    lbDialog.value = false;
};
const editTrack = () => {
    api.post('UiEditTrack', JSON.stringify({ RaceName: props.track.RaceName, TrackId: props.track.TrackId }))
    editDialog.value = false;
    closeApp()
};
const openEditAccess = async () => {
    const response = await api.post('UiGetAccess',  JSON.stringify({ RaceName: props.track.RaceName, TrackId: props.track.TrackId }))
    access.value = response.data
    accessDialog.value = true;
};

const openEditMetadata = async () => {
    Object.keys(settings.value).forEach((key) => {
      settings.value[key as keyof TrackMetadata] = props.track.Metadata[key as keyof TrackMetadata] || undefined
    })
    settingsDialog.value = true;
};

const editAccess = () => {
    api.post('UiEditAccess', JSON.stringify({RaceName: props.track.RaceName, TrackId: props.track.TrackId, NewAccess: access.value}))
    accessDialog.value = false;
}

const confirmSettings = async () => {
    if (settings.value.raceType === 'any') settings.value.raceType = undefined
    await api.post('UiConfirmSettings', JSON.stringify({ TrackId: props.track.TrackId, Metadata: settings.value }))
    settingsDialog.value = false;
    emit('update')
};

const deleteTrack = () => {
    api.post('UiDeleteTrack', JSON.stringify({ RaceName: props.track.RaceName, TrackId: props.track.TrackId }))
    deleteDialog.value = false;
    setTimeout(() => {
      emit('update')      
    }, 1500);
};

const setCuration = async (curated: number) => {
    const response = await api.post('UiSetCurated', JSON.stringify({ curated, trackId: props.track.TrackId }))
    if (response.data) {
      // eslint-disable-next-line vue/no-mutating-props
      props.track.Curated = curated;
      curateDialog.value = false;
    }
};

const showRace = async () => {
    const res = await api.post("UiShowTrack", JSON.stringify(props.track.TrackId));
    if (res.data) closeApp()
}


</script>

<style scoped lang="scss">
@use "@/styles/variables" as v;

.track-card {
  width: 100%;
  background: linear-gradient(165deg, rgba(18, 18, 20, 0.95), rgba(13, 13, 15, 0.9));
  border: 1px solid rgba(255, 255, 255, 0.06);
  overflow: hidden;
  backdrop-filter: blur(10px);
  transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);

  &:hover {
    border-color: rgba(v.$primary-color, 0.15);
    background: linear-gradient(165deg, rgba(22, 22, 24, 0.95), rgba(17, 17, 19, 0.9));
    transform: translateY(-1px);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  }
}

.card-wrapper {
  padding: 0.75rem;
}

.track-info {
  min-width: 0;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.track-header {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.title-section {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  min-width: 0;
}

.name-group {
  display: flex;
  align-items: center;
  gap: 0.625rem;
  min-width: 0;
  flex: 1;
}

.track-badge {
  background: rgba(v.$primary-color, 0.15);
  color: rgba(v.$primary-color, 0.9);
  padding: 0.25rem 0.5rem;
  border-radius: 4px;
  font-size: 0.75rem;
  font-weight: 600;
  letter-spacing: 0.5px;
  flex-shrink: 0;
}

.track-name {
  margin: 0;
  font-size: 0.9375rem;
  font-weight: 600;
  color: rgba(255, 255, 255, 0.95);
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  min-width: 0;
  letter-spacing: 0.25px;
}

.badge-group {
  display: flex;
  gap: 0.375rem;
  align-items: center;
  flex-shrink: 0;

  .status-chip {
    font-size: 0.75rem;
    font-weight: 500;
    letter-spacing: 0.25px;
    height: 20px;
  }
}

.quick-actions {
  display: flex;
  gap: 0.25rem;
  flex-shrink: 0;

  .action-btn {
    width: 28px;
    height: 28px;
    border-radius: 4px;
    background: rgba(255, 255, 255, 0.05);
    color: rgba(255, 255, 255, 0.7);

    &:hover {
      background: rgba(255, 255, 255, 0.1);
      color: rgba(255, 255, 255, 0.95);
    }
  }
}

.track-meta {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  background: rgba(255, 255, 255, 0.03);
  border-radius: 6px;
  padding: 0.625rem;
}

.meta-group {
  display: flex;
  flex-wrap: wrap;
  gap: 0.75rem;
  align-items: center;
  border-bottom: 1px solid rgba(255, 255, 255, 0.06);
  padding-bottom: 0.5rem;

  .meta-item {
    display: flex;
    align-items: center;
    gap: 0.375rem;
    font-size: 0.8125rem;

    &.creator {
      color: rgba(255, 255, 255, 0.85);
      font-weight: 500;
    }

    &.description {
      color: rgba(255, 255, 255, 0.6);
      font-style: italic;
    }
  }
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
  gap: 0.5rem;
  padding-top: 0.25rem;
}

.stat-item {
  display: flex;
  align-items: center;
  gap: 0.375rem;
  font-size: 0.8125rem;

  .v-icon {
    color: rgba(v.$primary-color, 0.9);
  }

  .stat-value {
    color: rgba(255, 255, 255, 0.9);
    font-weight: 500;
    letter-spacing: 0.25px;
  }
}

.action-menu {
  margin-top: 0.5rem;
}

.action-group {
  display: flex;
  gap: 0.375rem;
  align-items: center;

  &.primary {
    justify-content: space-between;

    .main-action {
      flex: 1;
      font-size: 0.8125rem;
      letter-spacing: 0.25px;
      height: 32px;
      font-weight: 500;
      text-transform: none;
      background: linear-gradient(45deg, rgba(v.$primary-color, 0.9), rgba(v.$primary-color, 0.8));
      
      &:hover {
        background: linear-gradient(45deg, rgba(v.$primary-color, 1), rgba(v.$primary-color, 0.9));
      }

      .v-icon {
        margin-right: 4px;
      }
    }

    .menu-btn {
      width: 32px;
      height: 32px;
      border-radius: 4px;
    }
  }
}

.action-menu-list {
  padding: 0.25rem;
  background: rgba(28, 28, 30, 0.98);
  border: 1px solid rgba(255, 255, 255, 0.08);
  backdrop-filter: blur(10px);
  border-radius: 6px;
  min-width: 180px;

  .v-list-item {
    border-radius: 4px;
    min-height: 36px;

    &.warning {
      color: rgb(255, 152, 0);
    }
  }
}

// Dialog styles remain unchanged...
</style>

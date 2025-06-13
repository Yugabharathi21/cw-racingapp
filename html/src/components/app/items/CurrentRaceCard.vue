<template>
    <v-card 
        rounded="lg" 
        class="race-card" 
        :elevation="0"
    >
        <div class="card-wrapper">
            <!-- Race Info Section -->
            <div class="race-info">
                <div class="race-header">
                    <!-- Title and Status -->
                    <div class="name-section">
                        <h3 class="track-name">{{ race.trackName }}</h3>
                        <div class="badge-row">
                            <v-chip
                                v-if="race.ranked"
                                color="warning"
                                variant="flat"
                                size="small"
                                density="comfortable"
                            >
                                <v-icon start size="14">mdi-podium-gold</v-icon>
                                {{ translate('ranked') }}
                            </v-chip>
                        </div>
                    </div>

                    <!-- Race Stats -->
                    <div class="race-stats">
                        <span class="stat">
                            <v-icon size="16" color="primary">mdi-flag-checkered</v-icon>
                            {{ lapsText }}
                        </span>
                        <span class="stat-divider">•</span>
                        <span class="stat">
                            <v-icon size="16" color="primary">mdi-account-group</v-icon>
                            {{ race.racers }} {{ translate('racers') }}
                        </span>
                        <span class="stat-divider">•</span>
                        <span class="stat">
                            <v-icon size="16" color="primary">mdi-car</v-icon>
                            {{ translate('class') }} {{ race.class }}
                        </span>
                    </div>

                    <!-- Race Settings -->
                    <div class="race-meta">
                        <div class="meta-group">
                            <span class="meta-item" :class="{ 'active': race.ghosting }">
                                <v-icon size="16">mdi-ghost</v-icon>
                                {{ translate(race.ghosting ? 'on' : 'off') }}
                            </span>
                            <span class="meta-item" v-if="race.reversed">
                                <v-icon size="16">mdi-backup-restore</v-icon>
                                {{ translate('reversed') }}
                            </span>
                        </div>
                        
                        <!-- Host Info -->
                        <div class="meta-group">
                            <span class="meta-item host">
                                <v-icon size="16">mdi-account-star</v-icon>
                                {{ race.hostName }}
                            </span>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Action Buttons -->
            <div class="action-bar">
                <div class="action-group danger">
                    <v-btn
                        v-if="race.canStart"
                        variant="tonal"
                        color="error"
                        density="comfortable"
                        class="action-btn"
                        :loading="loadingCancel"
                        @click="cancelRace"
                    >
                        <v-icon start size="18">mdi-close-circle</v-icon>
                        {{ translate('cancel_race') }}
                    </v-btn>
                    <v-btn
                        v-if="!race.canStart && globalStore.baseData.data.auth.cancelAll"
                        variant="tonal"
                        color="error"
                        density="comfortable"
                        class="action-btn"
                        :loading="loadingCancel"
                        @click="cancelRace"
                    >
                        <v-icon start size="18">mdi-close-circle</v-icon>
                        {{ translate('cancel_race_forced') }}
                    </v-btn>
                    <v-btn
                        variant="text"
                        color="error"
                        density="comfortable"
                        class="action-btn"
                        @click="leaveRace"
                    >
                        <v-icon start size="18">mdi-exit-to-app</v-icon>
                        {{ translate('leave_race') }}
                    </v-btn>
                </div>

                <v-btn
                    v-if="race.canStart || (!race.canStart && globalStore.baseData.data.auth.startAll)"
                    variant="flat"
                    color="success"
                    density="comfortable"
                    class="action-btn start-btn"
                    @click="startRace"
                >
                    <v-icon start size="18">mdi-flag-checkered</v-icon>
                    {{ translate(race.canStart ? 'start_race' : 'start_race_forced') }}
                </v-btn>
            </div>
        </div>
    </v-card>
</template>

<script setup lang="ts">
import { CurrentRace } from "@/store/types";
import { computed, ref } from "vue";
import { translate } from "@/helpers/translate";
import { useGlobalStore } from "@/store/global";
import api from "@/api/axios";
const globalStore = useGlobalStore();

const props = defineProps<{
  race: CurrentRace
}>()
const emits = defineEmits(['start', 'leave', 'cancel'])

const lapsText = computed(() => {
    let lapsText = translate('sprint')
    if (props.race.laps == -1) {
        lapsText = translate('elimination') + ' '
    }
    else if (props.race.laps > 0) {
        lapsText = props.race.laps + ' ' + translate('laps')
    }
    return lapsText
})

const startRace = async () => {
    emits('start', props.race)
}

const leaveRace = async () => {
    emits('leave', props.race)
}

const loadingCancel = ref(false)

const cancelRace = async () => {
    loadingCancel.value = true
    const res = await api.post("UiCancelRace", JSON.stringify(props.race.raceId));
    if (res && res.data) {
        emits('cancel')
    }
    loadingCancel.value = false
}

</script>

<style scoped lang="scss">
@use "@/styles/variables" as v;

.race-card {
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

    @media (max-width: 700px) {
        flex-direction: column;
        align-items: stretch;
        gap: 0.75rem;
    }
}

.race-info {
    flex: 1;
    min-width: 0;
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
}

.race-header {
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
        font-size: 0.8rem;
        font-weight: 500;
    }
}

.race-stats {
    display: flex;
    align-items: center;
    flex-wrap: wrap;
    gap: 0.5rem;
    font-size: 0.85rem;
    color: rgba(255, 255, 255, 0.7);

    .stat {
        display: flex;
        align-items: center;
        gap: 0.25rem;
    }

    .stat-divider {
        color: rgba(255, 255, 255, 0.2);
        font-size: 0.75rem;
    }
}

.race-meta {
    display: flex;
    flex-wrap: wrap;
    gap: 1rem;
    margin-top: 0.25rem;

    .meta-group {
        display: flex;
        gap: 1rem;
        align-items: center;
    }

    .meta-item {
        display: flex;
        align-items: center;
        gap: 0.35rem;
        font-size: 0.85rem;
        color: rgba(255, 255, 255, 0.6);

        &.active {
            color: v.$primary-color;
        }

        &.host {
            color: rgba(255, 255, 255, 0.7);
        }
        
        .v-icon {
            opacity: 0.9;
        }
    }
}

.action-bar {
    display: flex;
    gap: 0.75rem;
    flex-shrink: 0;

    @media (max-width: 700px) {
        border-top: 1px solid rgba(255, 255, 255, 0.06);
        padding-top: 0.75rem;
        flex-direction: column;
        gap: 0.5rem;
    }

    .action-group {
        display: flex;
        gap: 0.5rem;

        &.danger {
            @media (max-width: 500px) {
                flex-direction: column;
            }
        }
    }
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

    &.start-btn {
        min-width: 120px;
    }

    @media (max-width: 700px) {
        width: 100%;
    }
}
</style>

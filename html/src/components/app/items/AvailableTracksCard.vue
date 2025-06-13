<template>
    <v-card 
        rounded="lg" 
        class="track-card" 
        :elevation="0"
    >
        <div class="card-wrapper">
            <!-- Track Info Section -->
            <div class="track-info">
                <!-- Track Header -->
                <div class="track-header">
                    <div class="name-badges">
                        <h3 class="track-name">{{ track.RaceName }}</h3>
                        <div class="badge-row">
                            <v-icon 
                                v-if="track.Curated" 
                                color="success"
                                size="16"
                                icon="mdi-star"
                                class="status-icon"
                            />
                            <v-icon 
                                v-if="track.Metadata?.raceType && track.Metadata.raceType !== 'any_type'"
                                color="primary"
                                size="16"
                                icon="mdi-go-kart-track"
                                class="status-icon"
                            />
                        </div>
                    </div>

                    <div class="quick-stats">
                        <span class="stat">
                            <v-icon size="16" color="primary">mdi-ruler</v-icon>
                            {{ track.Distance }}m
                        </span>
                        <span class="stat-divider">•</span>
                        <span class="stat">
                            <v-icon size="16" color="primary">mdi-account-star</v-icon>
                            {{ track.CreatorName }}
                        </span>
                    </div>
                </div>

                <!-- Track Description -->
                <p v-if="track.Metadata?.description" class="track-desc">
                    {{ track.Metadata.description }}
                </p>
            </div>

            <!-- Action Buttons -->
            <div class="action-bar">
                <v-tooltip location="top" text="Preview Track">
                    <template v-slot:activator="{ props: tooltip }">
                        <v-btn
                            density="comfortable"
                            variant="text"
                            class="preview-btn"
                            v-bind="tooltip"
                            @click="showRace"
                        >
                            <v-icon>mdi-eye</v-icon>
                        </v-btn>
                    </template>
                </v-tooltip>

                <div class="main-actions">
                    <v-btn
                        density="comfortable"
                        variant="tonal"
                        class="action-btn"
                        @click="startRace"
                    >
                        <v-icon start size="18">mdi-cog</v-icon>
                        {{ translate('setup_race') }}
                    </v-btn>
                    <v-btn
                        density="comfortable"
                        color="primary"
                        variant="flat"
                        class="action-btn"
                        @click="quickHost"
                    >
                        <v-icon start size="18">mdi-play</v-icon>
                        {{ translate('quick_host') }}
                    </v-btn>
                </div>
            </div>
        </div>
    </v-card>
</template>

<script setup lang="ts">
import api from "@/api/axios";
import { closeApp } from "@/helpers/closeApp";
import { Track } from "@/store/types";
import { translate } from "@/helpers/translate";

const props = defineProps<{
  track: Track
}>()
const emits = defineEmits(['select'])

const showRace = async () => {
    const res = await api.post("UiShowTrack", JSON.stringify(props.track.TrackId));
    if (res.data) closeApp()
}

const quickHost = async () => {
    const res = await api.post("UiQuickHost", JSON.stringify(props.track));
    if (res.data) closeApp()
}

const startRace = async () => {
    emits('select', props.track)
}
</script>

<style scoped lang="scss">
@use "@/styles/variables" as v;

.track-card {
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

.track-info {
    flex: 1;
    min-width: 0;
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
}

.track-header {
    display: flex;
    flex-direction: column;
    gap: 0.35rem;
}

.name-badges {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    min-width: 0;
}

.track-name {
    margin: 0;
    font-size: 1rem;
    font-weight: 600;
    color: rgba(255, 255, 255, 0.95);
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    flex: 1;
    min-width: 0;
}

.badge-row {
    display: flex;
    gap: 0.35rem;
    flex-shrink: 0;

    .status-icon {
        padding: 3px;
        border-radius: 4px;
        background: rgba(255, 255, 255, 0.05);
    }
}

.quick-stats {
    display: flex;
    align-items: center;
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

.track-desc {
    margin: 0;
    font-size: 0.85rem;
    color: rgba(255, 255, 255, 0.6);
    line-height: 1.4;
    display: -webkit-box;
    -webkit-line-clamp: 2;
    line-clamp: 2;
    -webkit-box-orient: vertical;
    overflow: hidden;
}

.action-bar {
    display: flex;
    align-items: center;
    gap: 0.75rem;
    flex-shrink: 0;

    @media (max-width: 600px) {
        border-top: 1px solid rgba(255, 255, 255, 0.06);
        padding-top: 0.75rem;
    }

    .preview-btn {
        width: 36px;
        height: 36px;
        border-radius: 4px;
        background: rgba(255, 255, 255, 0.05);

        &:hover {
            background: rgba(255, 255, 255, 0.1);
        }
    }
}

.main-actions {
    display: flex;
    gap: 0.5rem;

    @media (max-width: 400px) {
        flex: 1;
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

        .v-icon {
            margin-right: 4px;
        }

        @media (max-width: 400px) {
            width: 100%;
        }
    }
}
</style>

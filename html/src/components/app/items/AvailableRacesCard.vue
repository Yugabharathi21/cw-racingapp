<template>
    <v-card 
        rounded="lg" 
        class="race-card" 
        :elevation="0"
        v-if="!hasExpired && !props.race.RaceData.Started"
    >
        <div class="card-wrapper">
            <!-- Race Info Section -->
            <div class="race-info">
                <div class="race-header">
                    <!-- Title and Status -->
                    <div class="name-section">
                        <h3 class="track-name">{{ props.race.TrackData.RaceName }}</h3>
                        <div class="badge-row">
                            <v-chip
                                v-if="props.race?.Ranked"
                                color="warning"
                                variant="flat"
                                size="small"
                                density="comfortable"
                            >
                                <v-icon start size="14">mdi-podium-gold</v-icon>
                                {{ translate('ranked') }}
                            </v-chip>
                            <v-chip
                                :color="props.race.Automated ? 'primary' : 'error'"
                                variant="flat"
                                size="small"
                                density="comfortable"
                            >
                                <v-icon start size="14">mdi-clock-outline</v-icon>
                                {{ `${minutes}:${seconds}` }}
                            </v-chip>
                        </div>
                    </div>

                    <!-- Race Stats -->
                    <div class="race-stats">
                        <span class="stat">
                            <v-icon size="16" color="primary">mdi-go-kart-track</v-icon>
                            {{ `${lapsText} ${props.race.TrackData.Distance}m` }}
                        </span>
                        <span class="stat-divider">•</span>
                        <span class="stat">
                            <v-icon size="16" color="primary">mdi-account-group</v-icon>
                            {{ `${props.race.racers} ${translate('racers')}` }}
                        </span>
                        <template v-if="props.race.MaxClass">
                            <span class="stat-divider">•</span>
                            <span class="stat">
                                <v-icon size="16" color="primary">mdi-car</v-icon>
                                {{ translate('class') }} {{ props.race.MaxClass }}
                            </span>
                        </template>
                    </div>

                    <!-- Money Info -->
                    <div class="race-meta">
                        <div class="meta-group" v-if="props.race.ParticipationAmount || buyInText">
                            <span v-if="props.race.ParticipationAmount" class="meta-item price">
                                <v-icon size="16">mdi-hand-coin</v-icon>
                                {{ participationText }}
                            </span>
                            <span v-if="buyInText" class="meta-item price">
                                <v-icon size="16">mdi-cash</v-icon>
                                {{ buyInText }}
                            </span>
                            <span v-if="potText && props.race.racers > 1" class="meta-item price">
                                <v-icon size="16">mdi-cash-multiple</v-icon>
                                {{ potText }}
                            </span>
                        </div>
                        
                        <!-- Settings Info -->
                        <div class="meta-group settings">
                            <span v-if="props.race.Ghosting" class="meta-item">
                                <v-icon size="16">mdi-ghost</v-icon>
                                {{ ghostingText }}
                            </span>
                            <span v-if="props.race.FirstPerson" class="meta-item">
                                <v-icon size="16">mdi-eye-lock</v-icon>
                                {{ translate('first_person') }}
                            </span>
                            <span v-if="props.race.Reversed" class="meta-item">
                                <v-icon size="16">mdi-backup-restore</v-icon>
                                {{ translate('reversed') }}
                            </span>
                        </div>

                        <!-- Host Info -->
                        <div class="meta-group">
                            <span class="meta-item host">
                                <v-icon size="16">mdi-account-star</v-icon>
                                {{ props.race.SetupRacerName }}
                            </span>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Action Buttons -->
            <div class="action-bar">
                <v-btn
                    class="action-btn preview"
                    variant="text"
                    density="comfortable"
                    @click="showRace"
                >
                    <v-icon>mdi-eye</v-icon>
                </v-btn>
                <v-btn
                    variant="tonal"
                    color="primary"
                    density="comfortable"
                    class="action-btn"
                    @click="joinRace"
                >
                    <v-icon start size="18">mdi-play</v-icon>
                    {{ translate('join_race') }}
                </v-btn>
            </div>
        </div>
    </v-card>
</template>

<script setup lang="ts">
import api from "@/api/axios";
import { closeApp } from "@/helpers/closeApp";
import { useGlobalStore } from "@/store/global";
import { translate } from "@/helpers/translate";
import { computed, onMounted, onUnmounted, ref } from "vue";

const props = defineProps<{
  race: any
}>()
const globalStore = useGlobalStore();

const joinRace = async () => {
    const res = await api.post("UiJoinRace", JSON.stringify(props.race.RaceId));
    if (res.data) closeApp()
}

const participationText = computed(() => {
    if(props.race.ParticipationCurrency === 'racingcrypto') {
        return  props.race.ParticipationAmount + ' ' + globalStore.baseData.data.payments.cryptoType
    }

    return '$' + props.race.ParticipationAmount

})

const lapsText = computed(() => {
    let lapsText = 'Sprint | '
    if (props.race.laps == -1) {
        lapsText = 'Elimination | '
    }
    else if (props.race.laps > 0) {
        lapsText = props.race.Laps + " "+  translate('laps') + ' | '
    }
    return lapsText
})

const buyInText = computed(() => {
    let text = undefined
    if (props.race.BuyIn > 0 ) {
        if (globalStore.baseData.data.payments.racing === 'racingcrypto') {
            text = props.race.BuyIn + " " + globalStore.baseData.data.payments.cryptoType + " " + translate('buy_in')
        } else {
            text = '$' + props.race.BuyIn + " " + translate('buy_in')
        }
    }
    return text
})

const potText = computed(() => {
    let text = undefined
    if (props.race.BuyIn > 0 ) {
        text = translate('pot') + ': ' + props.race.racers * props.race.BuyIn
    }
    return text
})

const ghostingText = computed(() => {
    let ghostingText = ''
    if (props.race.Ghosting) {
        ghostingText = translate('active')
        if (props.race.GhostingTime) {
            ghostingText = props.race.GhostingTime+'s)'
        }
    }
    return ghostingText
})

const showRace = async () => {
    const res = await api.post("UiShowTrack", JSON.stringify(props.race.TrackId));
    if (res.data) closeApp()
}

const futureTimestamp = ref<number>(props.race.ExpirationTime)

const remainingTime = ref<number>(futureTimestamp.value ? futureTimestamp.value - Math.floor(Date.now() / 1000): 0)

const minutes = computed(() => Math.floor(remainingTime.value / 60).toString().padStart(2, '0'))
const seconds = computed(() => (remainingTime.value % 60).toString().padStart(2, '0'))

const hasExpired = computed(() => remainingTime.value <= 0 )

const startCountdown = () => {
    if (futureTimestamp.value) {
        const interval = setInterval(() => {
          remainingTime.value -= 1
          if (remainingTime.value <= 0) {
            remainingTime.value = 0
            clearInterval(interval)
          }
        }, 1000)
        
        onUnmounted(() => {
          startCountdown()
          clearInterval(interval)
        })
    }
}

onMounted(() => {
    startCountdown()
})

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

    @media (max-width: 600px) {
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
    flex-direction: column;
    gap: 0.5rem;
    margin-top: 0.25rem;

    .meta-group {
        display: flex;
        flex-wrap: wrap;
        gap: 1rem;
        align-items: center;
        
        &.settings {
            color: rgba(255, 255, 255, 0.6);
        }
    }

    .meta-item {
        display: flex;
        align-items: center;
        gap: 0.35rem;
        font-size: 0.85rem;

        &.price {
            color: #4CAF50;
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
    gap: 0.5rem;
    flex-shrink: 0;

    @media (max-width: 600px) {
        border-top: 1px solid rgba(255, 255, 255, 0.06);
        padding-top: 0.75rem;
    }

    .action-btn {
        height: 36px;
        font-size: 0.9rem;
        font-weight: 500;
        border-radius: 4px;
        text-transform: none;

        &.preview {
            width: 36px;
            padding: 0;
            background: rgba(255, 255, 255, 0.05);

            &:hover {
                background: rgba(255, 255, 255, 0.1);
            }
        }

        &:not(.preview) {
            padding: 0 1rem;
            
            .v-icon {
                margin-right: 4px;
            }
        }

        @media (max-width: 600px) {
            flex: 1;
        }
    }
}
</style>

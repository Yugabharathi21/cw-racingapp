<template>
  <div class="topbar">
    <div class="status-items">
      <!-- User Info -->
      <div class="status-group user-info" v-if="globalStore.baseData?.data?.currentRacerName">
        <div class="status-item">
          <v-icon size="small">mdi-account</v-icon>
          <span class="item-text">{{ globalStore.baseData.data.currentRacerName }}</span>
        </div>
        <div class="status-item" v-if="globalStore.baseData?.data?.currentRacerAuth">
          <v-icon size="small">mdi-shield-check</v-icon>
          <span class="item-text auth">{{ globalStore.baseData.data.currentRacerAuth }}</span>
        </div>
      </div>

      <!-- Crew Info -->
      <div class="status-item" v-if="globalStore.baseData?.data?.currentCrewName">
        <v-icon size="small">mdi-account-group</v-icon>
        <span class="item-text">{{ globalStore.baseData.data.currentCrewName }}</span>
      </div>

      <!-- Rank -->
      <div class="status-item" v-if="!!globalStore.baseData?.data?.currentRanking">
        <v-icon size="small">mdi-trophy</v-icon>
        <span class="item-text">#{{ globalStore.baseData.data.currentRanking }}</span>
      </div>

      <!-- Vehicle -->
      <div class="status-item vehicle" v-if="!!globalStore.baseData?.data?.currentVehicle">
        <v-icon size="small">mdi-car-sports</v-icon>
        <span class="item-text">
          {{ globalStore.baseData.data.currentVehicle.model }}
          <span class="vehicle-class">[{{ globalStore.baseData.data.currentVehicle.class }}]</span>
        </span>
      </div>

      <!-- Crypto -->
      <div 
        class="status-item crypto" 
        v-if="globalStore.baseData.data.currentRacerName && !!globalStore.baseData?.data?.isUsingRacingCrypto"
        :class="{ 'clickable': allowOpenRacingCrypto }"
        @click="allowOpenRacingCrypto ? globalStore.showCryptoModal = true : undefined"
      >
        <v-icon size="small">mdi-currency-btc</v-icon>
        <span class="item-text">{{ globalStore.baseData.data.currentCrypto }} {{ globalStore.baseData.data.cryptoType }}</span>
      </div>
    </div>

    <!-- System Status -->
    <div class="system-status">
      <div class="status-item">
        <v-icon size="small">mdi-signal</v-icon>
        <v-icon size="small">mdi-battery-70</v-icon>
        <span class="item-text">75%</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { useGlobalStore } from "@/store/global";
import { translate } from "@/helpers/translate";
import { computed } from "vue";
const globalStore = useGlobalStore();

const allowOpenRacingCrypto = computed(() => {
  return globalStore.baseData.data.allowBuyingCrypto || 
         globalStore.baseData.data.allowSellingCrypto || 
         globalStore.baseData.data.allowTransferCrypto
})
</script>

<style scoped lang="scss">
@use '@/styles/variables' as v;

.topbar {
  user-select: none;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.35rem 0.75rem;
  background: rgba(0, 0, 0, 0.85);
  backdrop-filter: blur(10px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
  font-family: 'Rajdhani', sans-serif;
  font-size: 0.85rem;
  letter-spacing: 0.02em;
  height: 2.5rem;
}

.status-items {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.status-group {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  
  &::after {
    content: '';
    height: 1rem;
    width: 1px;
    background: rgba(255, 255, 255, 0.1);
    margin-left: 0.5rem;
  }
}

.status-item {
  display: flex;
  align-items: center;
  gap: 0.4rem;
  padding: 0.2rem 0.4rem;
  border-radius: 4px;
  background: rgba(255, 255, 255, 0.05);
  transition: all 0.2s ease;

  .v-icon {
    opacity: 0.7;
    color: v.$primary-color-light;
  }

  .item-text {
    color: rgba(255, 255, 255, 0.9);
    font-weight: 500;
    
    &.auth {
      color: v.$primary-color-light;
      font-weight: 600;
    }
  }

  &.vehicle {
    .vehicle-class {
      opacity: 0.6;
      font-size: 0.9em;
      margin-left: 0.2rem;
    }
  }

  &.crypto {
    background: rgba(v.$primary-color, 0.15);
    
    &.clickable {
      cursor: pointer;
      
      &:hover {
        background: rgba(v.$primary-color, 0.25);
        transform: translateY(-1px);
      }
    }
    
    .v-icon {
      color: v.$primary-color-light;
      opacity: 0.9;
    }
    
    .item-text {
      color: v.$primary-color-light;
      font-weight: 600;
    }
  }
}

.system-status {
  .status-item {
    gap: 0.3rem;
    
    .v-icon {
      font-size: 1rem;
    }
    
    .item-text {
      opacity: 0.7;
    }
  }
}
</style>

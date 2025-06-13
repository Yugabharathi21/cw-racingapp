<template>
  <div id="RacersPage" class="modern-page">
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
      <!-- My Racers Tab -->
      <v-window-item value="myRacers" class="tab-content">
        <div class="page-header">
          <div class="header-content">
            <h2>{{ translate('my_racers') }}</h2>
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

        <div class="racers-section" v-if="myRacers && myRacers.length > 0">
          <div class="racers-grid">
            <MyRacerCard 
              v-for="racer in filteredRacers" 
              :key="racer.id" 
              @triggerReload="getMyRacers()" 
              :racer="racer"
            ></MyRacerCard>
          </div>
        </div>
        <InfoText v-else :title="translate('no_racers')"></InfoText>
      </v-window-item>

      <!-- Create Racer Tab -->
      <v-window-item value="create" class="tab-content">
        <div v-if="globalStore.baseData?.data?.auth?.create" class="create-section">
          <div class="creation-card">
            <div class="card-header">
              <v-icon size="24" color="primary">mdi-account-plus</v-icon>
              <h3>{{ translate('create_racer') }}</h3>
            </div>
            <div class="card-content">
              <v-text-field
                v-model="create.racerName"
                :placeholder="translate('racer_name')"
                density="comfortable"
                variant="outlined"
                bg-color="rgba(0, 0, 0, 0.2)"
                hide-details
                prepend-inner-icon="mdi-account"
              ></v-text-field>
              
              <v-text-field
                v-model="create.racerId"
                :placeholder="translate('racer_id')"
                density="comfortable"
                variant="outlined"
                bg-color="rgba(0, 0, 0, 0.2)"
                hide-details
                prepend-inner-icon="mdi-identifier"
              ></v-text-field>

              <v-select
                v-if="creationTypes && creationTypes.length > 0"
                v-model="selectedAuth"
                :items="Object.values(creationTypes)"
                item-title="label"
                return-object
                density="comfortable"
                variant="outlined"
                bg-color="rgba(0, 0, 0, 0.2)"
                hide-details
                prepend-inner-icon="mdi-shield-account"
              ></v-select>
              <InfoText v-else :title="translate('not_auth')"></InfoText>

              <v-btn
                block
                color="primary"
                height="48"
                :loading="loading"
                :disabled="shouldDisableButton"
                @click="createUser()"
              >
                <v-icon start>mdi-plus-circle-outline</v-icon>
                {{ translate('create_racer') }}
              </v-btn>
            </div>
          </div>

          <div class="creation-card">
            <div class="card-header">
              <v-icon size="24" color="primary">mdi-account-multiple-plus</v-icon>
              <h3>{{ translate('create_racer_closest') }}</h3>
            </div>
            <div class="card-content">
              <v-text-field
                v-model="create.racerName"
                :placeholder="translate('racer_name')"
                density="comfortable"
                variant="outlined"
                bg-color="rgba(0, 0, 0, 0.2)"
                hide-details
                prepend-inner-icon="mdi-account"
              ></v-text-field>

              <v-select
                v-if="creationTypes && creationTypes.length > 0"
                v-model="selectedAuth"
                :items="Object.values(creationTypes)"
                item-title="label"
                return-object
                density="comfortable"
                variant="outlined"
                bg-color="rgba(0, 0, 0, 0.2)"
                hide-details
                prepend-inner-icon="mdi-shield-account"
              ></v-select>
              <InfoText v-else :title="translate('not_auth')"></InfoText>

              <v-btn
                block
                color="primary"
                height="48"
                :loading="loading"
                :disabled="shouldDisableButton"
                @click="createUserClosest()"
              >
                <v-icon start>mdi-plus-circle-outline</v-icon>
                {{ translate('create_racer') }}
              </v-btn>
            </div>
          </div>
        </div>
        <InfoText v-else :title="translate('not_auth')"></InfoText>
      </v-window-item>
    </v-window>
  </div>
</template>

<script setup lang="ts">
import api from "@/api/axios";
import { useGlobalStore } from "@/store/global";
import { MyRacer } from "@/store/types";
import { Ref } from "vue";
import { computed, onMounted } from "vue";
import { ref } from "vue";
import MyRacerCard from "../items/MyRacerCard.vue";
import InfoText from "../components/InfoText.vue";
import { translate } from "@/helpers/translate";

const globalStore = useGlobalStore();
const tab = ref(globalStore.currentPage);
const myRacers: Ref<MyRacer[] | undefined> = ref(undefined);
const creationTypes: Ref<any | undefined> = ref(undefined);
const search = ref('');
const selectedAuth = ref(undefined);

const filteredRacers = computed(() => {
  if (myRacers.value && search.value !== '') return myRacers.value.filter((racer) => racer.racername.toLowerCase().includes(search.value.toLowerCase()) || racer.citizenid.toLowerCase().includes(search.value.toLowerCase()) )
  return myRacers.value
})


const create = ref({
  racerName: '',
  racerId: ''
})
const loading = ref(false)

const shouldDisableButton = computed(() => {
  if (loading.value ) return true
  if (create.value.racerName === '') return true
  if (!selectedAuth.value) return true

  return false
})

const getMyRacers = async () => {
  const response = await api.post('UiGetRacersCreatedByUser')
  if (response.data) {
    myRacers.value = response.data
  }
  const typeResults = await api.post('UiGetPermissionedUserTypes')
  if (typeResults.data) {
    creationTypes.value = typeResults.data
  }
}

const createUser = async () => {
  loading.value = true
  const response = await api.post("UiCreateUser", JSON.stringify({ racerName: create.value.racerName, racerId: create.value.racerId, selectedAuth: selectedAuth.value }));
  setTimeout(() => {
    loading.value = false
    if (response.data) create.value.racerName = '';  create.value.racerId = '' 
  }, 1000);
};
const createUserClosest = async () => {
  loading.value = true
  const response = await api.post("UiCreateUserClosest", JSON.stringify({ racerName: create.value.racerName, selectedAuth: selectedAuth.value}));
  setTimeout(() => {
    if (response.data) create.value.racerName = '';  create.value.racerId = '' 
    loading.value = false
  }, 1000);
};

onMounted(() => {
  getMyRacers();
});

const tabs = [
  { value: "myRacers", label: "my_racers", icon: "mdi-account-group" },
  { value: "create", label: "create_racer", icon: "mdi-account-plus" },
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

/* Racers Section
   ========================================================================== */
.racers-section {
  margin-bottom: 2rem;
}

.racers-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
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
  max-width: 900px;
  margin: 0 auto;
  
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

  &:disabled {
    opacity: 0.7;
    transform: none;
    box-shadow: none;
  }
}
</style>

<template>
  <div id="CrewPage" class="modern-page">
    <!-- Navigation Tabs -->
    <div class="tab-container">
      <div class="tab-wrapper">
        <div
          v-for="tabItem in tabs"
          :key="tabItem.value"
          class="tab-item"
          :class="{ 'active': tab === tabItem.value }"
          @click="switchTab(tabItem.value)"
        >
          <v-icon>{{ tabItem.icon }}</v-icon>
          <span>{{ translate(tabItem.label) }}</span>
          <div class="tab-highlight"></div>
        </div>
      </div>
    </div>

    <!-- Content Area -->
    <v-window v-model="tab" class="content-container">
      <!-- My Crew Tab -->
      <v-window-item value="myCrew" class="tab-content">
        <div class="crew-overview" v-if="myCrew">
          <div class="crew-header">
            <h2>{{ myCrew.crewName }}</h2>
            <div class="crew-stats">
              <div class="stat-item">
                <v-icon color="primary" size="24">mdi-trophy-variant</v-icon>
                <span class="stat-label">{{ translate("rank") }}</span>
                <span class="stat-value">#{{ myCrew.rank }}</span>
              </div>
              <div class="stat-item">
                <v-icon color="primary" size="24">mdi-flag-checkered-circle</v-icon>
                <span class="stat-label">{{ translate("races") }}</span>
                <span class="stat-value">{{ myCrew.races }}</span>
              </div>
              <div class="stat-item">
                <v-icon color="primary" size="24">mdi-medal-outline</v-icon>
                <span class="stat-label">{{ translate("wins") }}</span>
                <span class="stat-value">{{ myCrew.wins }}</span>
              </div>
            </div>
          </div>

          <div class="crew-members" v-if="myCrew?.members?.length > 0">
            <div class="section-header">
              <h3>{{ translate("members") }}</h3>
              <span class="member-count">{{ myCrew.members.length }}</span>
            </div>
            <div class="members-grid">
              <MyCrewMemberCard
                v-for="member in myCrew.members"
                :isFounder="isFounder"
                :memberIsFounder="member.racername === myCrew.founderName"
                @triggerReload="getMyCrew()"
                :member="member"
                :key="member.citizenID"
              ></MyCrewMemberCard>
            </div>
          </div>
        </div>
        <InfoText v-else-if="!myCrew" :title="translate('not_in_crew')"></InfoText>
        <InfoText v-else :title="translate('no_members_in_crew')"></InfoText>
      </v-window-item>

      <!-- Manage Tab -->
      <v-window-item value="create" class="tab-content">
        <div class="manage-section">
          <div class="section-header">
            <h3>{{ translate("manage") }}</h3>
          </div>

          <div class="action-cards">
            <!-- Create Crew Card -->
            <div
              class="action-card"
              v-if="globalStore?.baseData?.data?.auth?.createCrew && !isFounder"
            >
              <div class="card-header">
                <v-icon size="24" color="primary">mdi-plus-circle-outline</v-icon>
                <h4>{{ translate("create_crew") }}</h4>
              </div>
              <div class="card-content">
                <v-text-field
                  v-model="creationCrewName"
                  :placeholder="translate('crew_name')"
                  variant="outlined"
                  density="comfortable"
                  bg-color="transparent"
                  hide-details
                ></v-text-field>
                <v-btn
                  block
                  color="primary"
                  height="42"
                  :loading="loading"
                  @click="createCrew()"
                >
                  {{ translate("confirm") }}
                </v-btn>
              </div>
            </div>

            <!-- Leave Crew Card -->
            <div
              class="action-card danger"
              v-if="globalStore.baseData.data.currentCrewName && !isFounder"
            >
              <div class="card-header">
                <v-icon size="24" color="error">mdi-exit-to-app</v-icon>
                <h4>{{ translate("leave_crew") }}</h4>
              </div>
              <div class="card-content">
                <p>{{ globalStore.baseData.data.currentCrewName }}</p>
                <v-btn
                  block
                  color="error"
                  height="42"
                  :loading="loading"
                  @click="leaveCrew()"
                >
                  {{ translate("leave") }}
                </v-btn>
              </div>
            </div>

            <!-- Disband Crew Card -->
            <div
              class="action-card danger"
              v-if="globalStore.baseData.data.currentCrewName && isFounder"
            >
              <div class="card-header">
                <v-icon size="24" color="error">mdi-close-circle-outline</v-icon>
                <h4>{{ translate("disband_crew") }}</h4>
              </div>
              <div class="card-content">
                <p>{{ globalStore.baseData.data.currentCrewName }}</p>
                <v-btn
                  block
                  color="error"
                  height="42"
                  :loading="loading"
                  @click="disbandCrew()"
                >
                  {{ translate("confirm") }}
                </v-btn>
              </div>
            </div>
          </div>
        </div>
      </v-window-item>

      <!-- Invites Tab -->
      <v-window-item value="invites" class="tab-content">
        <div class="invites-section">
          <div class="section-header">
            <h3>{{ translate("invites") }}</h3>
          </div>

          <!-- Pending Invites -->
          <div class="pending-invites" v-if="myInvites">
            <div class="invite-card">
              <div class="card-header">
                <v-icon size="24" color="primary">mdi-email-outline</v-icon>
                <h4>{{ translate("invite_from_crew") }}: {{ myInvites.crewName }}</h4>
              </div>
              <div class="card-actions">
                <v-btn
                  color="error"
                  variant="tonal"
                  height="42"
                  :loading="loading"
                  @click="denyCrew()"
                >
                  {{ translate("deny") }}
                </v-btn>
                <v-btn
                  color="success"
                  height="42"
                  :loading="loading"
                  @click="joinCrew()"
                >
                  {{ translate("accept") }}
                </v-btn>
              </div>
            </div>
          </div>
          <InfoText v-else :title="translate('no_invites')"></InfoText>

          <!-- Founder Invite Controls -->
          <div class="founder-controls" v-if="isFounder">
            <div class="section-divider"></div>

            <div class="action-cards">
              <!-- Send Invite Card -->
              <div class="action-card">
                <div class="card-header">
                  <v-icon size="24" color="primary">mdi-account-plus-outline</v-icon>
                  <h4>{{ translate("invite_member") }}</h4>
                </div>
                <div class="card-content">
                  <v-text-field
                    v-model="inviteText"
                    :placeholder="translate('citizen_id')"
                    variant="outlined"
                    density="comfortable"
                    bg-color="transparent"
                    hide-details
                  ></v-text-field>
                  <v-btn
                    block
                    color="primary"
                    height="42"
                    :loading="loading"
                    @click="inviteMember()"
                  >
                    {{ translate("send_invite") }}
                  </v-btn>
                </div>
              </div>

              <!-- Quick Invite Card -->
              <div class="action-card">
                <div class="card-header">
                  <v-icon size="24" color="primary">mdi-account-multiple-plus-outline</v-icon>
                  <h4>{{ translate("invite_closest") }}</h4>
                </div>
                <div class="card-content">
                  <v-btn
                    block
                    color="primary"
                    height="42"
                    :loading="loading"
                    @click="inviteMemberClosest()"
                  >
                    {{ translate("send_invite") }}
                  </v-btn>
                </div>
              </div>
            </div>
          </div>
        </div>
      </v-window-item>
    </v-window>
  </div>
</template>

<script setup lang="ts">
import api from "@/api/axios";
import { useGlobalStore } from "@/store/global";
import { Ref } from "vue";
import { onMounted } from "vue";
import { ref } from "vue";
import { Crew } from "@/store/types";
import MyCrewMemberCard from "../items/MyCrewMemberCard.vue";
import InfoText from "../components/InfoText.vue";
import { getBaseData } from "@/helpers/getBaseData";
import { computed } from "vue";

const globalStore = useGlobalStore();
const tab = ref(globalStore.currentTab);
const myCrew: Ref<Crew | undefined> = ref(globalStore.myCrew);
const myInvites: Ref<any | undefined> = ref(undefined);
const loading = ref(false);
import { translate } from "@/helpers/translate";

const tabs = [
  { value: "myCrew", label: "my_crew", icon: "mdi-account-group" },
  { value: "create", label: "manage", icon: "mdi-cog" },
  { value: "invites", label: "invites", icon: "mdi-email" },
];

const isFounder = computed(
  () => myCrew?.value?.founderName === globalStore.baseData.data.currentRacerName
);
const creationCrewName = ref("");
const inviteText = ref("");

const switchTab = (newTab: string) => {
  tab.value = newTab;
  getMyCrew();
};

const getMyCrew = async () => {
  const response = await api.post("UiGetCrewData");
  if (response.data) {
    globalStore.myCrew = response.data.crew;
    myCrew.value = response.data.crew;
    myInvites.value = response.data.invites;
  }
};

const createCrew = async () => {
  loading.value = true;
  await api.post(
    "UiCreateCrew",
    JSON.stringify({ crewName: creationCrewName.value })
  );
  setTimeout(() => {
    getBaseData();
    getMyCrew();
    loading.value = false;
  }, 1000);
};

const leaveCrew = async () => {
  loading.value = true;
  api.post(
    "UiLeaveCrew",
    JSON.stringify({ crewName: globalStore.baseData.data.currentCrewName })
  );
  setTimeout(() => {
    getBaseData();
    getMyCrew();
    loading.value = false;
  }, 1000);
};

const disbandCrew = async () => {
  loading.value = true;
  api.post(
    "UiDisbandCrew",
    JSON.stringify({ crewName: globalStore.baseData.data.currentCrewName })
  );
  setTimeout(() => {
    getBaseData();
    getMyCrew();
    loading.value = false;
  }, 1000);
};

const joinCrew = async () => {
  loading.value = true;
  api.post(
    "UiAcceptInvite",
    JSON.stringify({ crewName: myInvites.value.crewName })
  );
  setTimeout(() => {
    getBaseData();
    getMyCrew();
    loading.value = false;
  }, 1000);
};

const denyCrew = async () => {
  loading.value = true;
  api.post("UiDenyInvite", JSON.stringify({}));
  setTimeout(() => {
    getBaseData();
    getMyCrew();
    loading.value = false;
  }, 1000);
};

const inviteMember = async () => {
  loading.value = true;
  const response = await api.post(
    "UiSendInvite",
    JSON.stringify({ citizenId: inviteText.value })
  );
  setTimeout(() => {
    getBaseData();
    getMyCrew();
    loading.value = false;
    if (response.data) inviteText.value = "";
  }, 1000);
};
const inviteMemberClosest = async () => {
  loading.value = true;
  const response = await api.post("UiSendInviteClosest", JSON.stringify({}));
  setTimeout(() => {
    getBaseData();
    getMyCrew();
    loading.value = false;
  }, 1000);
};

onMounted(() => {
  getMyCrew();
});
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
    content: "";
    position: absolute;
    inset: 0;
    background: linear-gradient(
      120deg,
      transparent,
      rgba(255, 255, 255, 0.05),
      transparent
    );
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
      background: linear-gradient(
        120deg,
        transparent,
        rgba(v.$primary-color, 0.1),
        transparent
      );
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

  &:hover {
    transform: translateY(-2px);
    box-shadow: 0 6px 32px rgba(0, 0, 0, 0.3);
  }
}

.tab-content {
  padding: 2rem;
  min-height: 400px;

  @media (max-width: 768px) {
    padding: 1.5rem;
  }
}

/* Crew Overview
   ========================================================================== */
.crew-overview {
  display: flex;
  flex-direction: column;
  gap: 2.5rem;
}

.crew-header {
  background: linear-gradient(160deg, rgba(0, 0, 0, 0.5), rgba(0, 0, 0, 0.3));
  border-radius: 16px;
  padding: 2rem;
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
  border: 1px solid rgba(255, 255, 255, 0.05);
  box-shadow: 0 4px 24px rgba(0, 0, 0, 0.15);
  position: relative;
  overflow: hidden;

  &::before {
    content: "";
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    height: 1px;
    background: linear-gradient(
      90deg,
      transparent,
      rgba(v.$primary-color, 0.3),
      transparent
    );
  }

  h2 {
    font-size: 2rem;
    font-weight: 700;
    color: v.$primary-color-light;
    text-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
    letter-spacing: -0.5px;
  }
}

.crew-stats {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(160px, 1fr));
  gap: 1rem;
}

.stat-item {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 1rem 1.25rem;
  background: rgba(0, 0, 0, 0.3);
  border-radius: 12px;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  border: 1px solid rgba(255, 255, 255, 0.05);
  position: relative;
  overflow: hidden;

  &::before {
    content: "";
    position: absolute;
    inset: 0;
    background: linear-gradient(
      120deg,
      transparent,
      rgba(255, 255, 255, 0.05),
      transparent
    );
    transform: translateX(-100%);
    transition: transform 0.6s ease;
  }

  &:hover {
    background: rgba(0, 0, 0, 0.4);
    transform: translateY(-2px);
    border-color: rgba(v.$primary-color, 0.2);

    &::before {
      transform: translateX(100%);
    }

    .v-icon {
      transform: scale(1.1);
    }
  }

  .v-icon {
    transition: transform 0.3s ease;
  }

  .stat-label {
    color: rgba(255, 255, 255, 0.7);
    font-size: 0.9rem;
    font-weight: 500;
  }

  .stat-value {
    font-weight: 700;
    color: white;
    font-size: 1.1rem;
    margin-left: auto;
  }
}

/* Section Headers
   ========================================================================== */
.section-header {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-bottom: 1.5rem;
  position: relative;

  &::after {
    content: "";
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

  .member-count {
    background: rgba(v.$primary-color, 0.15);
    color: v.$primary-color-light;
    padding: 0.3rem 0.8rem;
    border-radius: 20px;
    font-size: 0.9rem;
    font-weight: 600;
    box-shadow: 0 0 15px rgba(v.$primary-color, 0.2);
  }
}

/* Members Grid
   ========================================================================== */
.members-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 1.5rem;
}

/* Manage Section
   ========================================================================== */
.manage-section {
  max-width: 900px;
  margin: 0 auto;
  width: 100%;

  .action-cards {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(400px, 1fr));
    gap: 1.5rem;
    margin-top: 1.5rem;

    @media (max-width: 768px) {
      grid-template-columns: 1fr;
    }
  }

  .action-card {
    background: rgba(0, 0, 0, 0.3);
    border-radius: 16px;
    border: 1px solid rgba(255, 255, 255, 0.05);
    overflow: hidden;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    position: relative;

    &::before {
      content: "";
      position: absolute;
      inset: 0;
      background: linear-gradient(
        120deg,
        transparent,
        rgba(255, 255, 255, 0.03),
        transparent
      );
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

    &.danger {
      &:hover {
        border-color: rgba(v.$negative-color, 0.2);

        .card-header {
          background: rgba(v.$negative-color, 0.1);
        }
      }
    }

    .card-header {
      display: flex;
      align-items: center;
      gap: 1rem;
      padding: 1.25rem;
      background: rgba(255, 255, 255, 0.03);
      transition: background 0.3s ease;

      .v-icon {
        transition: transform 0.3s ease;
      }

      h4 {
        font-size: 1.1rem;
        font-weight: 600;
        color: white;
        letter-spacing: -0.3px;
      }
    }

    .card-content {
      padding: 1.25rem;
      display: flex;
      flex-direction: column;
      gap: 1.25rem;

      p {
        color: rgba(255, 255, 255, 0.7);
        font-size: 0.95rem;
        line-height: 1.5;
      }

      .v-btn {
        height: 42px;
        letter-spacing: 0.5px;
        font-weight: 600;
        text-transform: none;
      }
    }
  }
}

/* Invite Section
   ========================================================================== */
.section-divider {
  height: 1px;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.1), transparent);
  margin: 2.5rem 0;
  opacity: 0.5;
}

.invite-card {
  background: rgba(0, 0, 0, 0.3);
  border-radius: 16px;
  border: 1px solid rgba(255, 255, 255, 0.05);
  overflow: hidden;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  position: relative;

  &::before {
    content: "";
    position: absolute;
    inset: 0;
    background: linear-gradient(
      120deg,
      transparent,
      rgba(255, 255, 255, 0.03),
      transparent
    );
    transform: translateX(-100%);
    transition: transform 0.6s ease;
  }

  &:hover {
    transform: translateY(-2px);
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

    .v-icon {
      transition: transform 0.3s ease;
    }

    h4 {
      font-size: 1.1rem;
      font-weight: 600;
      color: white;
    }
  }

  .card-actions {
    padding: 1.25rem;
    display: flex;
    gap: 1rem;
    justify-content: flex-end;

    .v-btn {
      height: 42px;
      min-width: 120px;
      letter-spacing: 0.5px;
      font-weight: 600;
      text-transform: none;
    }
  }
}

/* Input Fields Styling
   ========================================================================== */
.v-text-field {
  .v-field {
    border-radius: 12px !important;
    transition: all 0.3s ease;
    background: rgba(0, 0, 0, 0.2) !important;

    &:hover {
      background: rgba(0, 0, 0, 0.3) !important;
    }

    &--focused {
      background: rgba(0, 0, 0, 0.4) !important;
      border-color: rgba(v.$primary-color, 0.5) !important;
    }
  }

  input {
    font-size: 0.95rem !important;
    font-weight: 500 !important;
  }
}
</style>

<template>
  <v-navigation-drawer
    expand-on-hover
    rail
    elevation="1"
    class="modern-sidebar"
    :class="{ 'hover-active': isHovered }"
    @mouseenter="isHovered = true"
    @mouseleave="isHovered = false"
  >
    <!-- Header Section -->
    <div class="sidebar-header">
      <div class="logo-container">
        <v-icon size="28" color="primary">mdi-racing-helmet</v-icon>
        <div class="highlight-bar"></div>
      </div>
    </div>

    <!-- Navigation Links -->
    <v-list class="sidebar-nav" density="compact" nav>
      <!-- Main Navigation Group -->
      <div class="nav-group">
        <!-- Racing -->
        <v-list-item
          class="nav-item"
          :class="{ 'item-active': globalStore.currentPage === 'racing' }"
          :active="globalStore.currentPage === 'racing'"
          :title="translate('racing')"
          @click="openPage('racing')"
        >
          <template #prepend>
            <div class="icon-container">
              <v-icon>mdi-racing-helmet</v-icon>
            </div>
          </template>
        </v-list-item>

        <!-- Results -->
        <v-list-item
          class="nav-item"
          :class="{ 'item-active': globalStore.currentPage === 'results' }"
          :active="globalStore.currentPage === 'results'"
          :title="translate('results')"
          @click="openPage('results')"
        >
          <template #prepend>
            <div class="icon-container">
              <v-icon>mdi-trophy</v-icon>
            </div>
          </template>
        </v-list-item>

        <!-- Crew -->
        <v-list-item
          class="nav-item"
          :class="{ 'item-active': globalStore.currentPage === 'crew' }"
          :active="globalStore.currentPage === 'crew'"
          :title="translate('crew')"
          @click="openPage('crew')"
        >
          <template #prepend>
            <div class="icon-container">
              <v-icon>mdi-account-group</v-icon>
            </div>
          </template>
        </v-list-item>
      </div>

      <!-- Admin Navigation Group -->
      <div class="nav-group admin-group" v-if="hasAdminAccess">
        <!-- My Tracks -->
        <v-list-item
          v-if="globalStore.baseData.data?.auth?.create"
          class="nav-item"
          :class="{ 'item-active': globalStore.currentPage === 'mytracks' }"
          :active="globalStore.currentPage === 'mytracks'"
          :title="translate('my_tracks')"
          @click="openPage('mytracks')"
        >
          <template #prepend>
            <div class="icon-container">
              <v-icon>mdi-go-kart-track</v-icon>
            </div>
          </template>
        </v-list-item>

        <!-- Racers Admin -->
        <v-list-item
          v-if="globalStore.baseData.data?.auth?.control"
          class="nav-item"
          :class="{ 'item-active': globalStore.currentPage === 'racers' }"
          :active="globalStore.currentPage === 'racers'"
          :title="translate('racers')"
          @click="openPage('racers')"
        >
          <template #prepend>
            <div class="icon-container">
              <v-icon>mdi-account-multiple</v-icon>
            </div>
          </template>
        </v-list-item>

        <!-- Admin Panel -->
        <v-list-item
          v-if="globalStore.baseData.data?.auth?.adminMenu"
          class="nav-item"
          :class="{ 'item-active': globalStore.currentPage === 'admin' }"
          :active="globalStore.currentPage === 'admin'"
          :title="translate('race_admin')"
          @click="openPage('admin')"
        >
          <template #prepend>
            <div class="icon-container">
              <v-icon>mdi-shield-crown</v-icon>
            </div>
          </template>
        </v-list-item>
      </div>
    </v-list>

    <!-- Footer Settings -->
    <template v-slot:append>
      <v-list density="compact" nav class="footer-nav">
        <v-list-item
          class="nav-item"
          :class="{ 'item-active': globalStore.currentPage === 'settings' }"
          :active="globalStore.currentPage === 'settings'"
          :title="translate('settings')"
          @click="openPage('settings')"
        >
          <template #prepend>
            <div class="icon-container">
              <v-icon>mdi-cog</v-icon>
            </div>
          </template>
        </v-list-item>
      </v-list>
    </template>
  </v-navigation-drawer>
</template>

<script setup lang="ts">
import { useGlobalStore } from "@/store/global";
import { translate } from "@/helpers/translate";
import { ref, computed } from 'vue';

const globalStore = useGlobalStore();
const isHovered = ref(false);

const hasAdminAccess = computed(() => {
  const auth = globalStore.baseData.data?.auth;
  return auth?.create || auth?.control || auth?.adminMenu;
});

const openPage = (page: string) => {
  globalStore.$state.currentPage = page;
};
</script>

<style scoped lang="scss">
@use '@/styles/variables' as v;

/* Main Sidebar Container
   ========================================================================== */
.modern-sidebar {
  background: rgba(0, 0, 0, 0.95) !important;
  backdrop-filter: blur(10px);
  border-right: 1px solid rgba(255, 255, 255, 0.1) !important;
  display: flex;
  flex-direction: column;
  transition: width 0.3s cubic-bezier(0.4, 0, 0.2, 1);

  &.hover-active {
    :deep(.icon-container) {
      margin-right: 0.5rem;
    }
  }
}

/* Scrollbar Styling
   ========================================================================== */
:deep(.v-navigation-drawer__content) {
  background: transparent;
  scrollbar-width: thin;
  scrollbar-color: rgba(255, 255, 255, 0.1) transparent;
  display: flex;
  flex-direction: column;

  &::-webkit-scrollbar {
    width: 4px;
  }

  &::-webkit-scrollbar-thumb {
    background: rgba(255, 255, 255, 0.1);
    border-radius: 4px;
  }
}

/* Header Section
   ========================================================================== */
.sidebar-header {
  padding: 1rem 0.5rem;
  margin-bottom: 0.5rem;
  flex-shrink: 0;

  .logo-container {
    position: relative;
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 0.5rem;

    &::after {
      content: '';
      position: absolute;
      bottom: -0.5rem;
      left: 25%;
      right: 25%;
      height: 1px;
      background: linear-gradient(
        90deg,
        transparent,
        rgba(v.$primary-color, 0.3),
        transparent
      );
    }
  }
}

/* Navigation Section
   ========================================================================== */
.sidebar-nav {
  background: transparent !important;
  padding: 0.25rem !important;
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.nav-group {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
  padding: 0.25rem 0;

  &.admin-group {
    position: relative;
    margin-top: 0.5rem;
    padding-top: 0.5rem;

    &::before {
      content: '';
      position: absolute;
      top: 0;
      left: 25%;
      right: 25%;
      height: 1px;
      background: linear-gradient(
        90deg,
        transparent,
        rgba(255, 255, 255, 0.1),
        transparent
      );
    }
  }
}

/* List Items Styling
   ========================================================================== */
:deep(.v-list-item) {
  margin-bottom: 0.25rem;
  border-radius: 6px;
  min-height: 2.5rem !important;
  transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
  overflow: hidden;
}

.icon-container {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 2rem;
  height: 2rem;
  border-radius: 6px;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

:deep(.item-active) {
  background: rgba(v.$primary-color, 0.15) !important;
  
  &::before {
    content: '';
    position: absolute;
    left: 0;
    top: 0;
    bottom: 0;
    width: 3px;
    background: linear-gradient(
      to bottom,
      v.$primary-color,
      v.$primary-color-light
    );
    box-shadow: 0 0 10px rgba(v.$primary-color, 0.5);
  }

  .icon-container {
    background: rgba(v.$primary-color, 0.2);
    transform: scale(1.1);
  }

  .v-icon {
    color: v.$primary-color-light !important;
    filter: drop-shadow(0 0 5px rgba(v.$primary-color-light, 0.5));
  }
}

:deep(.v-list-item:hover) {
  background: rgba(255, 255, 255, 0.05) !important;

  .icon-container {
    background: rgba(255, 255, 255, 0.1);
  }

  .v-icon {
    color: v.$primary-color-light !important;
    transform: scale(1.1);
  }
}

/* Content Elements
   ========================================================================== */
:deep(.v-list-item__content) {
  font-family: 'Rajdhani', sans-serif;
  letter-spacing: 0.02em;
  font-weight: 500;
}

/* Icon Styling
   ========================================================================== */
:deep(.v-icon) {
  transition: all 0.2s ease;
  opacity: 0.9;
}

/* Footer Section
   ========================================================================== */
.footer-nav {
  border-top: 1px solid rgba(255, 255, 255, 0.1);
  padding-top: 0.5rem !important;
  background: transparent !important;
}
</style>

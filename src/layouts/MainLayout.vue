<template>
  <q-layout view="hHh lpR fFf">
    <q-header :class="`bg-${currentTheme.primary}`">
      <q-toolbar class="q-px-lg">
        <q-btn
          flat
          dense
          round
          icon="menu"
          aria-label="Menu"
          @click="toggleLeftDrawer"
        />
        
        <q-toolbar-title class="text-weight-bold">
          Sistema ERP
        </q-toolbar-title>

        <q-select
          v-model="selectedBranch"
          :options="branches"
          dense
          outlined
          class="branch-select q-mr-md"
          label="Sucursal"
          bg-color="white"
        />

        <q-btn-dropdown
          flat
          dense
          :label="currentTheme.name"
          class="q-mr-md text-white"
        >
          <q-list style="min-width: 200px">
            <q-item
              v-for="theme in themes"
              :key="theme.name"
              clickable
              v-close-popup
              @click="setTheme(theme)"
              class="q-py-md"
            >
              <q-item-section avatar>
                <div class="theme-preview">
                  <div class="color-strip" :style="{ background: `#${theme.primary}` }"></div>
                  <div class="color-strip" :style="{ background: `#${theme.secondary}` }"></div>
                  <div class="color-strip" :style="{ background: `#${theme.accent}` }"></div>
                </div>
              </q-item-section>
              <q-item-section>
                <q-item-label>{{ theme.name }}</q-item-label>
                <q-item-label caption>
                  <span class="text-primary">#{{ theme.primary }}</span> • 
                  <span class="text-secondary">#{{ theme.secondary }}</span> • 
                  <span class="text-grey">#{{ theme.accent }}</span>
                </q-item-label>
              </q-item-section>
            </q-item>
          </q-list>
        </q-btn-dropdown>

        <q-btn
          flat
          dense
          :icon="isDark ? 'light_mode' : 'dark_mode'"
          @click="toggleDarkMode"
          class="q-mr-md"
        />

        <q-btn-dropdown flat dense class="q-mr-md text-white" label="ESPAÑOL">
          <q-list>
            <q-item clickable v-close-popup>
              <q-item-section>Español</q-item-section>
            </q-item>
            <q-item clickable v-close-popup>
              <q-item-section>English</q-item-section>
            </q-item>
          </q-list>
        </q-btn-dropdown>

        <q-btn-dropdown flat dense>
          <template v-slot:label>
            <div class="row items-center no-wrap">
              <q-avatar size="28px" class="q-mr-xs">
                <img src="https://cdn.quasar.dev/img/avatar.png" />
              </q-avatar>
              <div class="text-white">{{ auth.user || 'ADMIN' }}</div>
              
            </div>
          </template>

          <q-list>
            <q-item clickable v-close-popup @click="logout">
              <q-item-section>Cerrar Sesión</q-item-section>
            </q-item>
          </q-list>
        </q-btn-dropdown>
      </q-toolbar>
    </q-header>

    <q-drawer
      v-model="leftDrawerOpen"
      show-if-above
      bordered
      :class="`bg-${currentTheme.accent}`"
      :width="240"
    >
      <q-scroll-area class="fit">
        <q-list padding>
          <q-item-label header class="text-weight-bold" :class="`text-${currentTheme.primary}`">
            MENÚ
          </q-item-label>

          <q-tree
            :nodes="menuItems"
            node-key="path"
            default-expand-all
            no-connectors
            @click.native.stop
          >
            <template v-slot:default-header="prop">
              <div 
                class="row items-center full-width cursor-pointer"
                @click="openInTab(prop.node)"
              >
                <q-icon :name="prop.node.icon" size="24px" class="q-mr-sm" />
                <div class="text-weight-medium">{{ prop.node.label }}</div>
              </div>
            </template>
          </q-tree>
        </q-list>
      </q-scroll-area>
    </q-drawer>

    <q-page-container class="q-pb-xl">
      <router-view />
    </q-page-container>

    <q-footer class="bg-gray-1">
      <div class="row items-center q-px-md">
        <q-tabs
          v-model="currentTab"
          dense
          class="col"
          active-color="red"
          indicator-color="transparent"
          align="left"
        >
          <q-tab
            v-for="tab in openTabs"
            :key="tab.path"
            :name="tab.path"
            class="tab-button"
            @click="router.push(tab.path)"
          >
            <div class="row items-center no-wrap q-gutter-x-xs">
              <q-icon :name="tab.icon" size="14px" />
              <div class="tab-label text-caption text-weight-medium">{{ tab.label }}</div>
              <q-btn
                v-if="tab.path !== '/'"
                flat
                dense
                round
                icon="close"
                size="xs"
                class="close-btn"
                @click.stop="closeTab(tab.path)"
              />
            </div>
          </q-tab>
        </q-tabs>

        <q-btn
          flat
          dense
          icon="close"
          label="CERRAR TODO"
          class="close-all-btn q-ml-sm"
          size="sm"
          @click="closeAllTabs"
          :disable="openTabs.length <= 1"
        />
      </div>
    </q-footer>
  </q-layout>
</template>

<script setup lang="ts">
import { ref, computed, watch, onMounted } from 'vue';
import { useRouter, useRoute } from 'vue-router';
import { useQuasar } from 'quasar';
import { useAuthStore } from '../stores/auth';
import { menuService } from 'src/services/menuServices';
import { MenuNode } from 'src/types/menu';
import { useTabStore } from 'src/stores/tab';

let menuItems = ref<MenuNode[]>([])

const $q = useQuasar();
const router = useRouter();
const route = useRoute();
const auth = useAuthStore();
const tabStore = useTabStore();

const leftDrawerOpen = ref(false);
const currentTab = ref('/');
const openTabs = ref([
  { path: '/', label: 'INICIO', icon: 'home' }
]);

const branches = [
  { label: 'Principal', value: 'main' },
  { label: 'Sucursal 1', value: 'branch1' },
  { label: 'Sucursal 2', value: 'branch2' }
];

const selectedBranch = ref(branches[0].value);

const themes = [
  {
    name: 'Por Defecto',
    primary: '3498db',
    secondary: '73c6b6',
    accent: 'd7dbdd'
  },
  {
    name: 'Corporativo',
    primary: '2c3e50',
    secondary: '2980b9',
    accent: 'ecf0f1'
  },
  {
    name: 'Clásico y Serio',
    primary: '34495e',
    secondary: 'e74c3c',
    accent: 'bdc3c7'
  },
  {
    name: 'Minimalista y Moderno',
    primary: '2ecc71',
    secondary: '3498db',
    accent: 'e67e22'
  },
  {
    name: 'Sobrio y Profesional',
    primary: '555555',
    secondary: '95a5a6',
    accent: 'f39c12'
  },
  {
    name: 'Elegante y Contrastante',
    primary: '9b59b6',
    secondary: '1abc9c',
    accent: 'f1c40f'
  }
];


const currentTheme = ref(themes[0]);

const isDark = computed({
  get: () => $q.dark.isActive,
  set: (value) => $q.dark.set(value)
});

function toggleLeftDrawer() {
  leftDrawerOpen.value = !leftDrawerOpen.value;
}

function toggleDarkMode() {
  isDark.value = !isDark.value;
}

function setTheme(theme: typeof themes[0]) {
  currentTheme.value = theme;
  document.documentElement.style.setProperty('--q-primary', `#${theme.primary}`);
  document.documentElement.style.setProperty('--q-secondary', `#${theme.secondary}`);
  document.documentElement.style.setProperty('--q-accent', `#${theme.accent}`);
}

function logout() {
  auth.logout();
  router.push('/login');
}

function openTab(tab: any) {
  const exists = openTabs.value.find(t => t.path === tab.path);
  if (!exists) {
    openTabs.value.push({
      path: tab.path,
      label: tab.label.toUpperCase(),
      icon: tab.icon
    });
  }
  currentTab.value = tab.path;
  router.push(tab.path);
}

function closeTab(path: string) {
  const index = openTabs.value.findIndex(tab => tab.path === path);
  if (index > 0) {
    openTabs.value.splice(index, 1);
    if (currentTab.value === path) {
      const newPath = openTabs.value[openTabs.value.length - 1].path;
      currentTab.value = newPath;
      router.push(newPath);
    }
  }
}

function closeAllTabs() {
  openTabs.value = [openTabs.value[0]];
  currentTab.value = '/';
  router.push('/');
}

const openInTab = (node: MenuNode) => {
  if (node.path) {
    openTab(node);
  } 
}

  onMounted(async () => {
    //menuItems.value = await menuService.getMenuItems();
    menuItems.value = await menuService.getMenuRol(Number (auth.rol));

    console.log('menuItems.value :::', menuItems.value );
    setTheme(themes[0]);
    if (!auth.isAuthenticated) {
      router.push('/login');
    }
  });

</script>

<style lang="scss" scoped>
.branch-select {
  width: 200px;
  background: rgb(243, 227, 4);
  border-radius: 4px;
  
  :deep(.q-field__control) {
    height: 32px;
  }
}

.theme-preview {
  display: flex;
  width: 32px;
  height: 32px;
  border-radius: 4px;
  border: 1px solid rgba(0, 0, 0, 0.1);
  overflow: hidden;
  
  .color-strip {
    flex: 1;
    height: 100%;
  }
}

.q-toolbar {
  height: 64px;
}

.tab-button {
  min-height: 32px;
  padding: 0 12px;
  margin: 0 2px;
  border-radius: 16px;
  font-size: 11px;
  background: #5dade2;
  color: black;
  transition: all 0.3s ease;

  &.q-tab--active {
    background: #5dade2;
    
    .tab-label {
      color: red;
    }
  }

  .tab-label {
    max-width: 120px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    color: black;
  }

  .close-btn {
    opacity: 0.7;
    transition: opacity 0.2s ease;
    
    .q-icon {
      font-size: 12px;
    }
  }

  &:hover {
    background: #3498db;
    .close-btn {
      opacity: 1;
    }
  }
}

.close-all-btn {
  font-size: 11px;
  height: 32px;
  border-radius: 16px;
  color: black;
  background: #5dade2;
  font-weight: 500;
  padding: 0 12px;

  &:hover:not(:disabled) {
    background: #3498db;
  }

  &:disabled {
    opacity: 0.7;
  }
}

.q-footer {
  background: #9ea49d !important;
  padding: 4px 0;
}



.q-drawer {
  .q-item {
    border-radius: 8px;
    margin: 4px 8px;
    height: 40px;

    &:hover {
      background: rgba(54, 233, 9, 0.05);
    }

    &--active {
      background: var(--q-primary);
      color: rgba(255, 255, 255, 0.453);
    }
  }
}

.body--dark {
  .theme-preview {
    border-color: rgba(255, 255, 255, 0.2);
  }
  
  .q-drawer {
    background: #121212;
    
    .q-item {
      &:hover {
        background: rgba(255, 255, 255, 0.1);
      }
    }
  }

  .tab-button {
    background: #5dade2;
    color: white;

    &.q-tab--active {
      background: #3498db;
    }
  }

  .close-all-btn {
    background: #5dade2;
    color: white;

    &:hover:not(:disabled) {
      background: #3498db;
    }
  }
}
</style>
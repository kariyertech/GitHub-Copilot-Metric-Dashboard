<template>
  <div>
    <v-toolbar 
      class="modern-toolbar" 
      elevation="0" 
      height="80">
      <v-btn icon>
        <img 
          src="/logo.png" 
          alt="GitHub Copilot Logo" 
          class="toolbar-logo ml-12"
        >
      </v-btn>

      <v-toolbar-title class="toolbar-title ml-10">{{ displayName }}</v-toolbar-title>
      <h2 class="warning-message"> {{ mockedDataMessage }} </h2>
      <v-spacer />

      <!-- Refresh Button -->
      <v-btn 
        icon 
        class="refresh-button mr-4" 
        @click="refreshMetrics"
        :loading="isRefreshing"
        :disabled="isRefreshing"
        title="Metrikleri Yenile">
        <v-icon>mdi-refresh</v-icon>
      </v-btn>

      <!-- Conditionally render the logout button -->
      <AuthState>
        <template #default="{ loggedIn, user }">
          <div v-show="loggedIn" class="user-info">
            Hoş geldiniz,
            <v-avatar class="user-avatar">
              <v-img :alt="user?.name" :src="user?.avatarUrl" />
            </v-avatar> {{ user?.name }}
          </div>
          <v-btn v-if="showLogoutButton && loggedIn" class="logout-button" @click="logout">Çıkış</v-btn>
        </template>
      </AuthState>

      <template #extension>
        <v-tabs 
          v-model="tab" 
          align-tabs="title"
          class="modern-tabs"
          bg-color="transparent">
          <v-tab v-for="item in tabItems" :key="item" :value="item">
            {{ item }}
          </v-tab>
        </v-tabs>

      </template>

    </v-toolbar>

    <!-- API Error Message -->
    <div v-show="apiError && !signInRequired" class="error-message" v-text="apiError" />
    <AuthState>
      <template #default="{ loggedIn }">
        <div v-show="signInRequired" class="github-login-container">
          <NuxtLink v-if="!loggedIn && signInRequired" to="/auth/github" external class="github-login-button"> <v-icon
              left>mdi-github</v-icon>
            GitHub ile Giriş Yap</NuxtLink>
        </div>
      </template>
      <template #placeholder>
        <button disabled>Loading...</button>
      </template>
    </AuthState>


    <div v-show="!apiError">
      <v-progress-linear v-show="!metricsReady" indeterminate color="indigo" />
      <v-window v-show="metricsReady && metrics.length" v-model="tab">
        <v-window-item v-for="item in tabItems" :key="item" :value="item">
          <v-card flat>
            <MetricsViewer v-if="item === displayItemName" :metrics="metrics" />
            <SeatsAnalysisViewer v-if="item === 'kullanıcı durumu'" :seats="seats" />
            <BreakdownComponent v-if="item === 'kullanılan diller'" :metrics="metrics" :breakdown-key="'language'" />
            <BreakdownComponent v-if="item === 'editörler'" :metrics="metrics" :breakdown-key="'editor'" />
            <CopilotChatViewer v-if="item === 'copilot kullanım'" :metrics="metrics" />
          </v-card>
        </v-window-item>
        <v-alert
v-show="metricsReady && metrics.length == 0" density="compact" text="Gösterilecek veri bulunmuyor"
          title="Veri Yok" type="warning" />
      </v-window>

    </div>

  </div>
</template>
<script lang='ts'>
import type { Metrics } from '@/model/Metrics';
import type { CopilotMetrics } from '@/model/Copilot_Metrics';
import type { MetricsApiResponse } from '@/types/metricsApiResponse';
import type { Seat } from "@/model/Seat";
import type { H3Error } from 'h3'

//Components
import MetricsViewer from './MetricsViewer.vue'
import BreakdownComponent from './BreakdownComponent.vue'
import CopilotChatViewer from './CopilotChatViewer.vue'
import SeatsAnalysisViewer from './SeatsAnalysisViewer.vue'

function getDisplayName(config: any) {
  if (config.githubOrg) {
    return `${config.githubOrg} - GitHub Copilot Metric Dashboard`;
  } else if (config.githubEnt) {
    return `${config.githubEnt} - GitHub Copilot Metric Dashboard`;
  } else {
    return 'GitHub Copilot Metric Dashboard';
  }
}

export default defineNuxtComponent({
  name: 'MainComponent',
  components: {
    MetricsViewer,
    BreakdownComponent,
    CopilotChatViewer,
    SeatsAnalysisViewer,
  },
  methods: {
    logout() {
      const { clear } = useUserSession()
      this.metrics = [];
      this.seats = [];
      // console.log('metrics are now', this.metrics);
      clear();
    },
    async refreshMetrics() {
      this.isRefreshing = true;
      this.apiError = undefined;
      
      try {
        // Reset states
        this.metricsReady = false;
        this.seatsReady = false;
        
        // Refresh metrics
        const metricsResponse = await $fetch('/api/metrics');
        
        if (metricsResponse) {
          this.metrics = metricsResponse.metrics || [];
          this.originalMetrics = metricsResponse.usage || [];
          this.metricsReady = true;
        } else {
          throw new Error('Metrik verileri alınamadı');
        }

        // Refresh seats
        const seatsResponse = await $fetch('/api/seats');
        if (seatsResponse) {
          this.seats = seatsResponse || [];
          this.seatsReady = true;
        } else {
          throw new Error('Kullanıcı durum verileri alınamadı');
        }

      } catch (error: any) {
        console.error('Refresh error:', error);
        this.apiError = `Veriler yenilenirken hata oluştu: ${error.message || 'Bilinmeyen hata'}`;
      } finally {
        this.isRefreshing = false;
      }
    }
  },

  data() {
    return {
      tabItems: ['kullanıcı durumu', 'kullanılan diller', 'editörler', 'copilot kullanım'],
      tab: null,
      isRefreshing: false
    }
  },
  created() {
    this.tabItems.unshift(this.displayItemName);
  },
  async setup() {
    const { loggedIn, user } = useUserSession()
    const config = useRuntimeConfig();
    const showLogoutButton = computed(() => config.public.usingGithubAuth && loggedIn.value);
    const mockedDataMessage = computed(() => config.public.isDataMocked ? 'Using mock data - see README if unintended' : '');
    const itemName = computed(() => config.public.scope);
    const displayItemName = computed(() => 'organizasyon');
    const githubInfo = getDisplayName(config.public)
    const displayName = computed(() => githubInfo);

    const metricsReady = ref(false);
    const metrics = ref<Metrics[]>([]);
    const originalMetrics = ref<CopilotMetrics[]>([]);
    const seatsReady = ref(false);
    const seats = ref<Seat[]>([]);
    // API Error Message
    const apiError = ref<string | undefined>(undefined);
    const signInRequired = computed(() => {
      return config.public.usingGithubAuth && !loggedIn.value;
    });

    /**
     * Handles API errors by setting appropriate error messages.
     * @param {H3Error} error - The error object returned from the API call.
     */
    function processError(error: H3Error) {
      console.error(error || 'No data returned from API');
      // Check the status code of the error response
      if (error.statusCode) {
        switch (error.statusCode) {
          case 401:
            apiError.value = '401 Yetkisiz erişim - GitHub API tokeni kontrol edin. .env dosyasındaki PAT token ve GitHub izinlerini kontrol edin.';
            break;
          case 404:
            apiError.value = `404 Bulunamadı - ${config.public.scope || ''} org:"${config.public.githubOrg || ''}" ent:"${config.public.githubEnt || ''}" team:"${config.public.githubTeam}" doğru mu? ${error.message}`;
            break;
          case 500:
            apiError.value = `500 Sunucu Hatası - büyük ihtimalle uygulamada hata var. Hata: ${error.message}`;
            break;
          default:
            apiError.value = `${error.statusCode} Error: ${error.message}`;
            break;
        }
      }
    }

    const metricsFetch = useFetch('/api/metrics');
    const seatsFetch = useFetch('/api/seats');

    const { data: metricsData, error: metricsError } = await metricsFetch;
    if (metricsError.value || !metricsData.value) {
      processError(metricsError.value as H3Error);
    } else {
      const apiResponse = metricsData.value as MetricsApiResponse;
      metrics.value = apiResponse.metrics || [];
      originalMetrics.value = apiResponse.usage || [];
      metricsReady.value = true;
    }

    if (config.public.scope === 'team' && metrics.value.length === 0 && !apiError.value) {
      apiError.value = 'API\'den veri dönmedi - takımın var olduğunu, aktivitesi olduğunu ve en az 5 aktif üyesi olduğunu kontrol edin';
    }

    const { data: seatsData, error: seatsError } = await seatsFetch;
    if (seatsError.value) {
      processError(seatsError.value as H3Error);
    } else {
      seats.value = seatsData.value || [];
      seatsReady.value = true;
    }

    return {
      metricsReady,
      metrics,
      originalMetrics,
      seatsReady,
      seats,
      apiError,
      signInRequired,
      showLogoutButton,
      config,
      mockedDataMessage,
      itemName,
      displayItemName,
      displayName,
      user
    };
  },
})
</script>

<style scoped>
/* Modern Toolbar */
.modern-toolbar {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%) !important;
  backdrop-filter: blur(20px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1) !important;
}

.toolbar-logo {
  width: 90px;
  height: 55px;
  filter: drop-shadow(0 4px 8px rgba(0, 0, 0, 0.2));
  transition: all 0.3s ease;
  object-fit: contain;
}

.toolbar-logo:hover {
  transform: scale(1.05);
  filter: drop-shadow(0 6px 12px rgba(0, 0, 0, 0.3));
}

/* Modern Tabs */
.modern-tabs {
  background: rgba(255, 255, 255, 0.1) !important;
  backdrop-filter: blur(10px);
  border-radius: 0 0 20px 20px;
}

.modern-tabs .v-tab {
  color: rgba(255, 255, 255, 0.8) !important;
  font-weight: 500;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin: 0 8px;
  border-radius: 8px;
  transition: all 0.3s ease;
}

.modern-tabs .v-tab:hover {
  color: white !important;
  background: rgba(255, 255, 255, 0.1);
  transform: translateY(-2px);
}

.modern-tabs .v-tab--selected {
  color: white !important;
  background: rgba(255, 255, 255, 0.2) !important;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

.toolbar-title {
  white-space: nowrap;
  overflow: visible;
  text-overflow: clip;
  color: white !important;
  font-weight: 600;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
  font-size: 1.4rem;
}

.error-message {
  color: red;
}

.warning-message {
  background: linear-gradient(45deg, #ff9a56 0%, #ff6b6b 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  font-weight: 500;
}

.logout-button {
  margin-left: auto;
  background: linear-gradient(45deg, #667eea 0%, #764ba2 100%) !important;
  border: none;
  color: white !important;
  border-radius: 8px;
  transition: all 0.3s ease;
}

.logout-button:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(102, 126, 234, 0.3);
}

.refresh-button {
  background: rgba(255, 255, 255, 0.1) !important;
  color: white !important;
  border-radius: 8px;
  transition: all 0.3s ease;
}

.refresh-button:hover {
  background: rgba(255, 255, 255, 0.2) !important;
  transform: translateY(-2px) rotate(180deg);
  box-shadow: 0 4px 12px rgba(255, 255, 255, 0.2);
}

.refresh-button:disabled {
  opacity: 0.6;
}

.refresh-button .v-icon {
  transition: transform 0.3s ease;
}

.github-login-container {
  display: flex;
  justify-content: center;
  margin-top: 20px;
}

.github-login-button {
  display: flex;
  align-items: center;
  background: linear-gradient(45deg, #24292e 0%, #40505a 100%);
  color: white;
  padding: 10px 20px;
  border-radius: 12px;
  text-decoration: none;
  font-weight: bold;
  font-size: 14px;
  box-shadow: 0 4px 15px rgba(36, 41, 46, 0.2);
  transition: all 0.3s ease;
}

.github-login-button:hover {
  background: linear-gradient(45deg, #444d56 0%, #5a6870 100%);
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(36, 41, 46, 0.3);
}

.github-login-button v-icon {
  margin-right: 8px;
}

.user-info {
  display: flex;
  align-items: center;
  font-weight: 500;
  color: rgba(255, 255, 255, 0.9);
}

.user-avatar {
  margin-right: 8px;
  margin-left: 8px;
  border: 3px solid rgba(255, 255, 255, 0.8);
  transition: all 0.3s ease;
}

.user-avatar:hover {
  border-color: rgba(255, 255, 255, 1);
  transform: scale(1.05);
}
</style>
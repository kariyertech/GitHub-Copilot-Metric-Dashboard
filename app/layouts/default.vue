<template>
  <v-app>
    <v-main>
      <slot />
    </v-main>
    <v-footer class="bg-indigo-lighten-1 text-center d-flex flex-column fixed-footer">
      <div class="px-4 py-2 text-center w-100">
        {{ new Date().getFullYear() }} — <strong>GitHub Copilot Metrics Dashboard</strong> — 1.0.0
      </div>
    </v-footer>
  </v-app>
</template>

<script lang="ts" setup>
function getDisplayName(config: any) {
  if (config.githubOrg) {
    return `${config.githubOrg} - GitHub Copilot Metric Dashboard`;
  } else if (config.githubEnt) {
    return `${config.githubEnt} - GitHub Copilot Metric Dashboard`;
  } else {
    return 'GitHub Copilot Metric Dashboard';
  }
}

const config = useRuntimeConfig();
const version = computed(() => config.public.version);
const githubInfo = getDisplayName(config.public)
useHead({
  title: githubInfo,
  meta: [
    { name: 'description', content: 'GitHub Copilot Metrics Dashboard' }
  ]
});
</script>

<style scoped>
.fixed-footer {
  height: 50px;
  max-height: 50px;
}
</style>

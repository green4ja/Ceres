<template>
  <q-layout view="hHh lpR fFf">
    <q-header class="bg-primary text-white">
      <q-toolbar class="bg-primary text-white">
        <q-btn no-caps flat label="Ceres" to="/" icon="public"/>

        <q-tabs v-model="tab" shrink no-caps @update:model-value="onTabChange">
          <q-tab name="generate-meals" label="Generate Meals" />
          <q-tab name="view-meals" label="View Meals" />
        </q-tabs>
      </q-toolbar>
    </q-header>

      <q-page-container class="main-page-container">
        <router-view />
      </q-page-container>
  </q-layout>
</template>

<script setup>
import { ref, watch } from 'vue'
import { useRouter, useRoute } from 'vue-router'

const router = useRouter();
const route = useRoute();
const tab = ref('');

watch(() => route.path, (newPath) => {
  if(newPath.includes('generate-meals')) tab.value = 'generate-meals';
  else if(newPath.includes('view-meals')) tab.value = 'view-meals';
  else tab.value = '';
}, { immediate: true })

const onTabChange = (newTab) => {
  router.push(`/${newTab}`)
}
</script>

<style scoped>
.main-page-container {
  padding: 16px 16px 16px;
  margin-top: 16px;
  box-sizing: border-box;
}
</style>

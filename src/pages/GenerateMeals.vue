<template>
  <div class="page-container">
    <SearchTool @profile-generated="onProfileGenerated" />
    <NutrientProfile :profile-data="nutrientProfileData" />
  </div>
</template>

<style scoped>
.page-container {
  padding: var(--q-spacing-xl) var(--q-spacing-lg);
  display: flex;
  gap: 8px;
  align-items: flex-end;
}
</style>

<script setup>
import { ref } from 'vue'
import NutrientProfile from 'src/components/NutrientProfile.vue'
import SearchTool from 'src/components/SearchTool.vue'

const nutrientProfileData = ref([])

const onProfileGenerated = (data) => {
  nutrientProfileData.value = data
}
</script>

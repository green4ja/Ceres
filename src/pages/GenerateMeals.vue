<template>
  <div class="page-container">
    <SearchTool />
    <NutrientProfile />
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
import NutrientProfile from 'src/components/NutrientProfile.vue';
import SearchTool from 'src/components/SearchTool.vue';
</script>

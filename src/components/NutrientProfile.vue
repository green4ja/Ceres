<template>
  <div class="q-pa-md">
    <q-table title="Nutrient Profile" dense :rows="rows" :columns="columns" row-key="nutrientId" />
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  profileData: {
    type: Array,
    default: () => [],
  },
})

const columns = [
  { name: 'nutrientName', align: 'left', label: 'Nutrient', field: 'nutrientName', sortable: true },
  {
    name: 'total',
    align: 'center',
    label: 'Amount',
    field: 'total',
    sortable: true,
    format: (val) => val.toFixed(2),
  },
  { name: 'unit', label: 'Unit', field: 'unit', sortable: true },
]

const rows = computed(() => props.profileData)
</script>

<style scoped></style>

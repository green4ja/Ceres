<template>
  <div class="q-pa-md">
    <q-table
      title="Nutrient Profile"
      dense
      :rows="rows"
      :columns="columns"
      row-key="nutrientId"
    />
  </div>
</template>

<script setup>
import { ref } from 'vue'

// TODO:
// - Implement taking real nutrient profile data from SearchTool.vue

const columns = [
  { name: 'nutrientName', align: 'left', label: 'Nutrient', field: 'nutrientName', sortable: true },
  { name: 'total', align: 'center', label: 'Amount', field: 'total', sortable: true },
  { name: 'unit', label: 'Unit', field: 'unit', sortable: true },
];

const rows = ref(
  [
    {
        "nutrientId": 1051,
        "nutrientNumber": "255",
        "nutrientName": "Water",
        "unit": "g",
        "rank": 100,
        "total": 75.3
    },
    {
        "nutrientId": 2047,
        "nutrientNumber": "957",
        "nutrientName": "Energy (Atwater General Factors)",
        "unit": "kcal",
        "rank": 280,
        "total": 98
    },
    {
        "nutrientId": 2048,
        "nutrientNumber": "958",
        "nutrientName": "Energy (Atwater Specific Factors)",
        "unit": "kcal",
        "rank": 290,
        "total": 88
    },
    {
        "nutrientId": 1008,
        "nutrientNumber": "208",
        "nutrientName": "Energy",
        "unit": "kcal",
        "rank": 300,
        "total": 309.84000000000003
    },
  ]
);

</script>

<style scoped></style>

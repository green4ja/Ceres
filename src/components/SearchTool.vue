<template>
  <div class="search-tool-container">
    <!-- Top Interactions -->
    <div class="search-interactables">
      <q-input outlined v-model="ingredientTextInput" label="Ingredient" class="search-input" />
      <q-select outlined v-model="selectedOption" :options="options" label="Search Type" class="search-select" />
      <q-btn color="primary" no-caps label="Search" class="search-button" v-on:click="onSearch"/>
    </div>
    <!-- Search Results -->
    <q-table
      flat bordered
      title="Search Results"
      dense
      :rows="searchRows"
      :columns="searchColumns"
      row-key="fdcId"
      style="margin-top: 8px;"
    >
      <template v-slot:body-cell-actions="props">
        <q-td :props="props">
          <q-btn color="primary" no-caps label="Select" size="sm" @click="onSelect(props.row)" />
        </q-td>
      </template>
    </q-table>
    <!-- Selected Ingredients -->
    <q-table
      flat bordered
      title="Selected Ingredients"
      dense
      :rows="selectRows"
      :columns="selectColumns"
      row-key="fdcId"
      style="margin-top: 8px;"
    >
      <template v-slot:body-cell-grams="props">
        <q-td :props="props">
          <q-input
            outlined
            dense
            type="number"
            min="0"
            suffix="g"
            v-model.number="props.row.grams"
          />
        </q-td>
      </template>
    </q-table>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { api } from 'boot/axios'

const USDA_API_KEY = import.meta.env.VITE_USDA_API_KEY || 'DEMO_KEY';

// search-interactables
const ingredientTextInput = ref('');
const selectedOption = ref('');
const options = [
  'Branded',
  'Survey (FNDDS)',
  'SR Legacy',
  'Foundation',
  'Experimental',
];

// Search Results q-table
const searchRows = ref([]);
const searchColumns = [
  { name: 'description', align: 'left', label: 'description', field: 'description', sortable: true },
  { name: 'fdcId', align: 'center', label: 'fdcId', field: 'fdcId', sortable: true },
  { name: 'dataType', label: 'dataType', field: 'dataType', sortable: true },
  { name: 'brandOwner', label: 'brandOwner', field: 'brandOwner', sortable: true },
  { name: 'brandName', label: 'brandName', field: 'brandName', sortable: true },
  { name: 'actions', align: 'center', label: '', field: 'actions' },
];

// Selected Ingredients q-table
const selectRows = ref([]);
const selectColumns = [
  { name: 'ingredient', align: 'left', label: 'ingredient', field: 'ingredient', sortable: true },
  { name: 'fdcId', align: 'center', label: 'fdcId', field: 'fdcId', sortable: true },
  { name: 'grams', align: 'center', label: 'grams', field: 'grams' },
];

const onSearch = () => {
  const payload = {
    query: ingredientTextInput.value,
    dataType: [selectedOption.value],
  };

  api.post(`https://api.nal.usda.gov/fdc/v1/foods/search?api_key=${USDA_API_KEY}`, payload)
    .then(response => {
      searchRows.value = response.data.foods.map(food => ({
        description: food.description,
        fdcId: food.fdcId,
        dataType: food.dataType,
        brandOwner: food.brandOwner,
        brandName: food.brandName,
      }))
    })
    .catch(error => {
      if (error.response) {
      console.error('Server Error:', error.response.data);
      } else if (error.request) {
      console.error('No Response:', error.request);
      } else {
      console.error('Error:', error.message);
      }
    });
}

const onSelect = (row) => {
  selectRows.value.push({
    ingredient:  row.description,
    fdcId: row.fdcId,
    grams: null,
  })
  searchRows.value = [];
}
</script>

<style scoped>
.search-tool-container {
  width: 75%;
  padding: var(--q-spacing-md);
}

.search-interactables {
  display: flex;
  gap: 8px;
  align-items: flex-end;
}

.search-input {
  flex: 0 0 60%;
}

.search-select {
  flex: 0 0 30%;
}

.search-button {
  flex: 0 0 10%;
  height: 56px;
}
</style>

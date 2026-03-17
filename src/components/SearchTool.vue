<template>
  <div class="search-tool-container">
    <div class="search-interactables">
      <q-input outlined v-model="text" label="Ingredient" class="search-input" />
      <q-select outlined v-model="selectedOption" :options="options" label="Search Type" class="search-select" />
      <q-btn color="primary" no-caps label="Search" class="search-button" v-on:click="onSearch"/>
    </div>
    <q-table
      flat bordered
      title="Search Results"
      dense
      :rows="rows"
      :columns="columns"
      row-key="name"
      style="margin-top: 8px;"
    />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { api } from 'boot/axios'

// search-interactables
const text = ref('')
const selectedOption = ref('')
const options = [
  'Branded',
  'Survey (FNDDS)',
  'SR Legacy',
  'Foundation',
  'Experimental',
]

// q-table
const rows = ref([])
const columns = [
  { name: 'description', align: 'left', label: 'description', field: 'description', sortable: true },
  { name: 'fdcId', align: 'center', label: 'fdcId', field: 'fdcId', sortable: true },
  { name: 'dataType', label: 'dataType', field: 'dataType', sortable: true },
  { name: 'brandOwner', label: 'brandOwner', field: 'brandOwner', sortable: true },
  { name: 'brandName', label: 'brandName', field: 'brandName', sortable: true },
]

const onSearch = () => {
  const payload = {
    query: text.value,
    dataType: [selectedOption.value],
  };

  api.post('https://api.nal.usda.gov/fdc/v1/foods/search?api_key=DEMO_KEY', payload)
    .then(response => {
      rows.value = response.data.foods.map(food => ({
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

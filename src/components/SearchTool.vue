<template>
  <div class="search-tool-container">
    <!-- Top Interactions -->
    <div class="search-interactables">
      <q-input outlined v-model="ingredientTextInput" label="Ingredient" class="search-input" />
      <q-select
        outlined
        v-model="selectedOption"
        :options="searchOptions"
        label="Search Type"
        class="search-select"
      />
      <q-btn
        color="primary"
        no-caps
        label="Search"
        class="search-button"
        :disable="!canSearch"
        v-on:click="onSearch"
      />
    </div>
    <!-- Search Results -->
    <q-table
      flat
      bordered
      title="Search Results"
      dense
      :rows="searchRows"
      :columns="searchColumns"
      row-key="id"
      style="margin-top: 8px"
    >
      <template v-slot:body-cell-actions="props">
        <q-td :props="props">
          <q-btn color="primary" no-caps label="Select" size="sm" @click="onSelect(props.row)" />
        </q-td>
      </template>
    </q-table>
    <!-- Selected Ingredients -->
    <q-table
      flat
      bordered
      title="Selected Ingredients"
      dense
      :rows="selectRows"
      :columns="selectColumns"
      row-key="id"
      style="margin-top: 8px"
    />
    <!-- Generate Nutrient Profile -->
    <!-- <q-btn
      color="primary"
      no-caps
      label="Generate Nutrient Profile"
      :disable="!canGenerate"
      v-on:click="onGenerate"
    /> -->
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { api } from 'boot/axios'

const API_KEY = import.meta.env.VITE_SPOONACULAR_API_KEY

// search-interactables
const ingredientTextInput = ref('')
const selectedOption = ref('')
const searchOptions = ['Ingredient', 'Grocery Product']

// Search Results q-table
const searchRows = ref([])
const searchColumns = [
  {
    name: 'name',
    align: 'left',
    label: 'name',
    field: 'name',
    sortable: true,
  },
  { name: 'id', align: 'center', label: 'id', field: 'id', sortable: true },
  { name: 'actions', align: 'center', label: '', field: 'actions' },
]

// Selected Ingredients q-table
const selectRows = ref([])
const selectColumns = [
  { name: 'name', align: 'left', label: 'name', field: 'name', sortable: true },
  { name: 'id', align: 'center', label: 'id', field: 'id', sortable: true },
]

const onSearch = () => {
  const numExpectedResults = 100 // (1-100)
  const searchQuery = ingredientTextInput.value

  if (selectedOption.value == 'Grocery Product') {
    api
      .get('https://api.spoonacular.com/food/products/search', {
        params: {
          apiKey: API_KEY,
          query: searchQuery,
          number: numExpectedResults,
        },
      })
      .then((response) => {
        searchRows.value = response.data.products.map((product) => ({
          name: product.title,
          id: product.id,
        }))
      })
      .catch((error) => {
        if (error.response) {
          console.error('Server Error:', error.response.data)
        } else if (error.request) {
          console.error('No Response:', error.request)
        } else {
          console.error('Error:', error.message)
        }
      })
  } else {
    // 'Ingredient'
    const searchSort = 'calories' // (https://spoonacular.com/food-api/docs#Recipe-Sorting-Options)
    const searchSortDirection = 'desc' // 'asc' or 'desc'

    api
      .get('https://api.spoonacular.com/food/ingredients/search', {
        params: {
          apiKey: API_KEY,
          query: searchQuery,
          number: numExpectedResults,
          sort: searchSort,
          sortDirection: searchSortDirection,
        },
      })
      .then((response) => {
        searchRows.value = response.data.results.map((result) => ({
          name: result.name,
          id: result.id,
        }))
      })
      .catch((error) => {
        if (error.response) {
          console.error('Server Error:', error.response.data)
        } else if (error.request) {
          console.error('No Response:', error.request)
        } else {
          console.error('Error:', error.message)
        }
      })
  }
}

const onSelect = (row) => {
  selectRows.value.push({
    name: row.name,
    id: row.id,
  })

  searchRows.value = []
  ingredientTextInput.value = ''
  selectedOption.value = ''
}

// const onGenerate = () => {
//   const foodsToFetch = {
//     fdcIds: selectRows.value.map((row) => row.fdcId),
//   }

//   const gramsByFdcId = new Map(selectRows.value.map((row) => [row.fdcId, Number(row.grams)]))

//   // TEST CASE (REMOVE LATER)
//   // {
//   // "fdcIds": [
//   //     2641085, Branded
//   //     1105314, Foundation
//   //     2501940 Branded
//   //   ]
//   // }

//   // TODO:
//   // - Verify portal-data endpoint data is being scaled

//   api
//     .post(`https://api.nal.usda.gov/fdc/v1/foods?api_key=${API_KEY}`, foodsToFetch)
//     .then(async (response) => {
//       scaledIngredients.value = response.data.map((ingredient) => {
//         const grams = gramsByFdcId.get(ingredient.fdcId)

//         return {
//           ingredient: ingredient.description,
//           fdcId: ingredient.fdcId,
//           foodNutrients: scaleNutrients(ingredient.foodNutrients || [], grams),
//         }
//       })

//       const returnedFdcIds = new Set(scaledIngredients.value.map((ingredient) => ingredient.fdcId))

//       const missingFdcIds = foodsToFetch.fdcIds.filter((fdcId) => !returnedFdcIds.has(fdcId))

//       if (missingFdcIds.length === 0) {
//         return
//       }

//       const missingIngredients = await Promise.all(
//         missingFdcIds.map(async (fdcId) => {
//           try {
//             const response = await api.get(`/usda-portal-data/external/${fdcId}`)

//             const ingredient = response.data
//             const grams = gramsByFdcId.get(fdcId)

//             return {
//               ingredient: ingredient.description,
//               fdcId: ingredient.fdcId || fdcId,
//               foodNutrients: scaleNutrients(ingredient.foodNutrients || [], grams),
//             }
//           } catch (error) {
//             console.error(`Failed to fetch missing fdcId ${fdcId}:`, error)
//             return null
//           }
//         }),
//       )

//       scaledIngredients.value = [
//         ...scaledIngredients.value,
//         ...missingIngredients.filter((ingredient) => ingredient !== null),
//       ]

//       console.log(aggregateIngredientData())
//     })
//     .catch((error) => {
//       if (error.response) {
//         console.error('Server Error:', error.response.data)
//       } else if (error.request) {
//         console.error('No Response:', error.request)
//       } else {
//         console.error('Error:', error.message)
//       }
//     })
// }

// const canGenerate = computed(() => {
//   if (selectRows.value.length === 0) return false

//   return selectRows.value.every((row) => {
//     const grams = Number(row.grams)
//     return Number.isFinite(grams) && grams > 0
//   })
// })

const canSearch = computed(() => {
  return ingredientTextInput.value.trim().length > 0 && Boolean(selectedOption.value)
})
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

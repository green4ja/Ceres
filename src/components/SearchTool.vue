<template>
  <div class="search-tool-container">
    <!-- Top Interactions -->
    <div class="search-interactables">
      <q-input outlined v-model="ingredientTextInput" label="Ingredient" class="search-input" />
      <q-select
        outlined
        v-model="selectedOption"
        :options="options"
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
      row-key="fdcId"
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
      row-key="fdcId"
      style="margin-top: 8px"
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
    <!-- Generate Nutrient Profile -->
    <q-btn
      color="primary"
      no-caps
      label="Generate Nutrient Profile"
      :disable="!canGenerate"
      v-on:click="onGenerate"
    />
  </div>
</template>

<script setup>
import { ref, computed, defineEmits } from 'vue'
import { api } from 'boot/axios'

const emit = defineEmits(['profile-generated'])

const USDA_API_KEY = import.meta.env.VITE_USDA_API_KEY || 'DEMO_KEY'

// search-interactables
const ingredientTextInput = ref('')
const selectedOption = ref('')
const options = ['Branded', 'Survey (FNDDS)', 'SR Legacy', 'Experimental']

// Search Results q-table
const searchRows = ref([])
const searchColumns = [
  {
    name: 'description',
    align: 'left',
    label: 'description',
    field: 'description',
    sortable: true,
  },
  { name: 'fdcId', align: 'center', label: 'fdcId', field: 'fdcId', sortable: true },
  { name: 'dataType', label: 'dataType', field: 'dataType', sortable: true },
  { name: 'brandOwner', label: 'brandOwner', field: 'brandOwner', sortable: true },
  { name: 'brandName', label: 'brandName', field: 'brandName', sortable: true },
  { name: 'actions', align: 'center', label: '', field: 'actions' },
]

// Selected Ingredients q-table
const selectRows = ref([])
const selectColumns = [
  { name: 'ingredient', align: 'left', label: 'ingredient', field: 'ingredient', sortable: true },
  { name: 'fdcId', align: 'center', label: 'fdcId', field: 'fdcId', sortable: true },
  { name: 'grams', align: 'center', label: 'grams', field: 'grams' },
]

// Generating Nutrient Profile
const scaledIngredients = ref([])

const onSearch = () => {
  const numResults = 200
  const searchFilters = {
    query: ingredientTextInput.value,
    pageSize: numResults,
    dataType: [selectedOption.value],
  }

  api
    .post(`https://api.nal.usda.gov/fdc/v1/foods/search?api_key=${USDA_API_KEY}`, searchFilters)
    .then((response) => {
      searchRows.value = response.data.foods.map((food) => ({
        description: food.description,
        fdcId: food.fdcId,
        dataType: food.dataType,
        brandOwner: food.brandOwner,
        brandName: food.brandName,
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

const onSelect = (row) => {
  selectRows.value.push({
    ingredient: row.description,
    fdcId: row.fdcId,
    grams: null,
  })
  searchRows.value = []
  ingredientTextInput.value = ''
  selectedOption.value = ''
}

const scaleNutrients = (nutrientsToScale, gramScalar) => {
  return nutrientsToScale.map((nutrient) => {
    const amount = Number(nutrient.amount)

    if (!Number.isFinite(amount)) {
      return nutrient
    }

    return {
      ...nutrient,
      amount: (amount / 100) * gramScalar,
    }
  })
}

const onGenerate = () => {
  const foodsToFetch = {
    fdcIds: selectRows.value.map((row) => row.fdcId),
  }

  const gramsByFdcId = new Map(selectRows.value.map((row) => [row.fdcId, Number(row.grams)]))

  api
    .post(`https://api.nal.usda.gov/fdc/v1/foods?api_key=${USDA_API_KEY}`, foodsToFetch)
    .then(async (response) => {
      scaledIngredients.value = response.data.map((ingredient) => {
        const grams = gramsByFdcId.get(ingredient.fdcId)

        return {
          ingredient: ingredient.description,
          fdcId: ingredient.fdcId,
          foodNutrients: scaleNutrients(ingredient.foodNutrients || [], grams),
        }
      })

      emit('profile-generated', aggregateIngredientData())
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

const canGenerate = computed(() => {
  if (selectRows.value.length === 0) return false

  return selectRows.value.every((row) => {
    const grams = Number(row.grams)
    return Number.isFinite(grams) && grams > 0
  })
})

const canSearch = computed(() => {
  return ingredientTextInput.value.trim().length > 0 && Boolean(selectedOption.value)
})

const aggregateIngredientData = () => {
  const totals = new Map()

  for (const ingredient of scaledIngredients.value) {
    for (const foodNutrient of ingredient.foodNutrients || []) {
      const nutrient = foodNutrient.nutrient || {}

      // Prefer id; fallback to number+unit if needed
      const id = nutrient.id ?? null
      const number = nutrient.number ?? ''
      const name = nutrient.name ?? ''
      const unit = nutrient.unitName || nutrient.nutrientUnit?.name || ''

      if (!id && !number) continue
      if (!name) continue

      const nutrientQuantity = Number(foodNutrient.amount ?? foodNutrient.value)

      if (!Number.isFinite(nutrientQuantity)) continue

      const key = id ? `id:${id}` : `num:${number}|unit:${unit}`
      const existing = totals.get(key)

      if (!existing) {
        totals.set(key, {
          nutrientId: id,
          nutrientNumber: number,
          nutrientName: name,
          unit,
          rank: nutrient.rank ?? Number.MAX_SAFE_INTEGER,
          total: nutrientQuantity,
        })
      } else {
        if (existing.unit === unit) {
          existing.total += nutrientQuantity
        }
      }
    }
  }

  return [...totals.values()].sort((a, b) => a.rank - b.rank)
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

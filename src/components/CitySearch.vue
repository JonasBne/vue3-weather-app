<script setup lang="ts">
import { ref, watch } from 'vue'
import { useRoute, useRouter } from 'vue-router'

interface QueryResult {
  id: number
  name: string
  region: string
  country: string
  lat: number
  lon: number
  url: string
}

const apiKey = import.meta.env.VITE_API_KEY

const router = useRouter()
const route = useRoute()

const query = ref(route.query.search || '')
const queryResults = ref<QueryResult[] | null>(null)
const searchError = ref(false)

watch(query, (newQuery) => {
  if (newQuery.length === 0) {
    router.replace({ query: {} })
    return
  }
  router.push({ query: { search: newQuery } })
})

const handleChange = async () => {
  if (!query.value) {
    queryResults.value = null
    return
  }

  try {
    const response = await fetch(
      `https://api.weatherapi.com/v1/search.json?q=${query.value}&key=${apiKey}`
    )
    const data = await response.json()
    queryResults.value = data
  } catch (e) {
    searchError.value = true
  }
}

const handleRedirect = (queryResult: QueryResult) => {
  console.log('clicked')
  router.push({
    name: 'cityView',
    params: {
      city: queryResult.name
    },
    query: {
      lat: queryResult.lat,
      lon: queryResult.lon,
      preview: 'true'
    }
  })
}
</script>

<template>
  <input
    name="query"
    class="bg-transparent text-white border-b w-full p-2 focus:outline-none focus:shadow-xl"
    placeholder="Search for a city..."
    v-model="query"
    @input="handleChange"
  />
  <ul class="p-2 absolute w-full shadow-md bg-slate-600" v-if="query && queryResults">
    <p v-if="searchError">Something went wrong. Please try again.</p>
    <li v-if="!searchError && queryResults.length === 0 && query.length > 0">
      No results were found for "{{ query }}"
    </li>
    <template v-else>
      <li v-for="result in queryResults" :key="result.id" @click="handleRedirect(result)">
        {{ result.name }}
      </li>
    </template>
  </ul>
</template>

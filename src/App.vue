<script setup>
import AppHeader from './components/AppHeader.vue'
import AppCardList from './components/AppCardList.vue'
import AppDrawer from './components/AppDrawer.vue'
import { onMounted, reactive, ref, watch } from 'vue'
import axios from 'axios'

const items = ref([]) // value: {[]}

const filters = reactive({
  sortBy: 'title',
  searchQuery: ''
})

const onChangeSelect = (event) => {
  filters.sortBy = event.target.value
}
const onChangeSearchInput = (event) => {
  filters.searchQuery = event.target.value
}

const fetchFavorites = async () => {
  try {
    const { data: favorites } = await axios.get(`https://54bb1c1584149a00.mokky.dev/favorites`)
    items.value = items.value.map((item) => {
      const favorite = favorites.find((favorite) => favorite.parentId === item.id)
      if (!favorite) {
        return item
      }
      return {
        ...item,
        isFavorite: true,
        favoriteId: favorite.id
      }
    })
  } catch (e) {
    console.log(e)
  }
}
const addToFavorite = async (item) => {
  item.isFavorite = true
}
const fetchItems = async () => {
  try {
    const params = {
      sortBy: filters.sortBy
    }
    if (filters.searchQuery) {
      params.title = `*${filters.searchQuery}*`
    }
    const { data } = await axios.get('https://54bb1c1584149a00.mokky.dev/items', {
      params
    })
    items.value = data.map((obj) => ({
      ...obj,
      isFavorite: false,
      osAdded: false
    }))
    console.log(data)
  } catch (e) {
    console.log(e)
  }
}
onMounted(async () => {
  await fetchItems()
  await fetchFavorites()
})
watch(filters, fetchItems)
</script>

<template>
  <!-- <AppDrawer /> -->
  <div class="w-4/5 m-auto bg-white rounded-xl shadow-xl mt-14">
    <AppHeader />
    <div class="p-10">
      <div class="flex justify-between items-center">
        <h2 class="text-3xl font-bolt mb-8">Все кроссовки</h2>

        <div class="flex gap-4">
          <!--Select-->
          <select
            @change="onChangeSelect"
            class="py-2 px-3 border rounded-md outline-none bg-transparent"
          >
            <option value="name">Name</option>
            <option value="price">To Up</option>
            <option value="-price">To Down</option>
          </select>

          <!--Search-->
          <div class="relative">
            <img class="absolute left-4 top-3" src="/search.svg" alt="search" />
            <input
              @input="onChangeSearchInput"
              class="border rounded-md py-2 pl-11 pr-4 outline-none focus:border-gray-400"
              placeholder="Search..."
            />
          </div>
        </div>
      </div>
      <AppCardList :items="items" />
    </div>
  </div>
</template>

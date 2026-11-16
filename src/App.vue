<script setup>
import { ref, computed, watch } from 'vue'
import listingsData from './data/listings.json'
import ListingCard from './components/ListingCard.vue'
import ListingModal from './components/ListingModal.vue'

const FAVORITES_KEY = 'housing_board_favorites'

// Состояние фильтров
const searchQuery = ref('')
const minPrice = ref('')
const maxPrice = ref('')
const selectedRooms = ref('')
const selectedDistrict = ref('')
const activeTab = ref('all') // 'all' | 'favorites'
const selectedListing = ref(null)

// Избранное с сохранением в localStorage
const loadFavorites = () => {
  try {
    const saved = localStorage.getItem(FAVORITES_KEY)
    return saved ? JSON.parse(saved) : []
  } catch {
    return []
  }
}

const favorites = ref(loadFavorites())

watch(
  favorites,
  (newVal) => {
    localStorage.setItem(FAVORITES_KEY, JSON.stringify(newVal))
  },
  { deep: true }
)

const toggleFavorite = (id) => {
  const index = favorites.value.indexOf(id)
  if (index === -1) {
    favorites.value.push(id)
  } else {
    favorites.value.splice(index, 1)
  }
}

const isFavorite = (id) => favorites.value.includes(id)

// Список уникальных районов для выпадающего списка
const districts = computed(() => {
  const set = new Set()
  for (const item of listingsData) {
    if (item.district) set.add(item.district)
  }
  return Array.from(set).sort((a, b) => a.localeCompare(b, 'ru'))
})

// Итоговый отфильтрованный список
const filteredListings = computed(() => {
  return listingsData.filter((item) => {
    // Вкладка "Избранное"
    if (activeTab.value === 'favorites' && !favorites.value.includes(item.id)) {
      return false
    }

    // Поиск по названию
    if (searchQuery.value.trim()) {
      const title = (item.title || '').toLowerCase()
      if (!title.includes(searchQuery.value.trim().toLowerCase())) {
        return false
      }
    }

    // Фильтр по минимальной цене
    if (minPrice.value !== '' && minPrice.value !== null) {
      if (item.price === null || item.price === undefined || item.price < Number(minPrice.value)) {
        return false
      }
    }

    // Фильтр по максимальной цене
    if (maxPrice.value !== '' && maxPrice.value !== null) {
      if (item.price === null || item.price === undefined || item.price > Number(maxPrice.value)) {
        return false
      }
    }

    // Фильтр по количеству комнат
    if (selectedRooms.value !== '') {
      if (item.rooms === null || item.rooms === undefined) return false
      if (selectedRooms.value === '4+') {
        if (item.rooms < 4) return false
      } else if (item.rooms !== Number(selectedRooms.value)) {
        return false
      }
    }

    // Фильтр по району
    if (selectedDistrict.value && item.district !== selectedDistrict.value) {
      return false
    }

    return true
  })
})
</script>

<template>
  <div class="app-container">
    <!-- Заголовок и переключение вкладок -->
    <header class="header">
      <h1>🏠 Доска объявлений</h1>
      <div class="tabs">
        <button 
          :class="{ active: activeTab === 'all' }" 
          @click="activeTab = 'all'"
        >
          Все объявления
        </button>
        <button 
          :class="{ active: activeTab === 'favorites' }" 
          @click="activeTab = 'favorites'"
        >
          ♥ Избранное ({{ favorites.length }})
        </button>
      </div>
    </header>

    <!-- Панель фильтров -->
    <section class="filters">
      <input 
        v-model="searchQuery" 
        type="text" 
        placeholder="Поиск по названию..." 
        class="filter-input"
      />
      
      <div class="price-filters">
        <input v-model="minPrice" type="number" placeholder="Цена от" class="filter-input price-input" />
        <input v-model="maxPrice" type="number" placeholder="Цена до" class="filter-input price-input" />
      </div>

      <select v-model="selectedRooms" class="filter-input">
        <option value="">Любые комнаты</option>
        <option value="0">Студия</option>
        <option value="1">1 комната</option>
        <option value="2">2 комнаты</option>
        <option value="3">3 комнаты</option>
        <option value="4+">4+ комнат</option>
      </select>

      <select v-model="selectedDistrict" class="filter-input">
        <option value="">Любой район</option>
        <option v-for="d in districts" :key="d" :value="d">{{ d }}</option>
      </select>
    </section>

    <!-- Счетчик -->
    <div class="results-count">
      Найдено объявлений: <strong>{{ filteredListings.length }}</strong>
    </div>

    <!-- Сетка карточек -->
    <div v-if="filteredListings.length > 0" class="listings-grid">
      <ListingCard
        v-for="item in filteredListings"
        :key="item.id"
        :listing="item"
        :is-favorite="isFavorite(item.id)"
        @toggle-favorite="toggleFavorite"
        @open-details="selectedListing = $event"
      />
    </div>

    <!-- Заглушка, если ничего не найдено -->
    <div v-else class="empty-state">
      <p>😕 По выбранным условиям объявления не найдены.</p>
      <p class="empty-sub">Попробуйте изменить фильтры или сбросить поиск.</p>
    </div>

    <!-- Модальное окно с подробностями -->
    <ListingModal
      v-if="selectedListing"
      :listing="selectedListing"
      :is-favorite="isFavorite(selectedListing.id)"
      @close="selectedListing = null"
      @toggle-favorite="toggleFavorite"
    />
  </div>
</template>

<style>
/* Сброс стилей и базовые настройки */
body {
  margin: 0;
  background-color: #f8f9fa;
  font-family: 'Inter', system-ui, -apple-system, sans-serif;
  color: #1f2937;
}

.app-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

/* Заголовок и вкладки */
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24px;
  flex-wrap: wrap;
  gap: 16px;
}

.header h1 {
  margin: 0;
  font-size: 24px;
}

.tabs {
  display: flex;
  gap: 8px;
  background: #e5e7eb;
  padding: 4px;
  border-radius: 8px;
}

.tabs button {
  border: none;
  background: transparent;
  padding: 8px 16px;
  border-radius: 6px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
  color: #4b5563;
}

.tabs button.active {
  background: #ffffff;
  color: #111827;
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
}

/* Фильтры */
.filters {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  margin-bottom: 20px;
  background: #ffffff;
  padding: 16px;
  border-radius: 12px;
  border: 1px solid #e5e7eb;
}

.filter-input {
  padding: 10px 14px;
  border: 1px solid #d1d5db;
  border-radius: 8px;
  font-size: 14px;
  outline: none;
  transition: border-color 0.2s;
  background: #fff;
}

.filter-input:focus {
  border-color: #3b82f6;
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
}

.price-filters {
  display: flex;
  gap: 8px;
}

.price-input {
  width: 120px;
}

.results-count {
  margin-bottom: 16px;
  font-size: 15px;
  color: #4b5563;
}

/* Сетка карточек */
.listings-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 20px;
}

/* Пустое состояние */
.empty-state {
  text-align: center;
  padding: 60px 20px;
  background: #ffffff;
  border-radius: 12px;
  border: 1px dashed #d1d5db;
}

.empty-state p {
  margin: 0 0 8px;
  font-size: 18px;
  color: #374151;
}

.empty-sub {
  font-size: 14px !important;
  color: #9ca3af !important;
}
</style>
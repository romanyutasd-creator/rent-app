<script setup>
import { ref } from 'vue'

const props = defineProps({
  listing: {
    type: Object,
    required: true,
  },
  isFavorite: {
    type: Boolean,
    default: false,
  },
})

const emit = defineEmits(['toggle-favorite', 'open-details'])

const imageError = ref(false)

const formatPrice = (price) => {
  if (price === null || price === undefined || price === '') return 'Цена не указана'
  return `${Number(price).toLocaleString('ru-RU')} ₽/мес`
}

const formatRooms = (rooms) => {
  if (rooms === null || rooms === undefined) return 'Комнаты не указаны'
  if (rooms === 0) return 'Студия'
  return `${rooms}-комн.`
}
</script>

<template>
  <article class="card" @click="emit('open-details', listing)">
    <div class="card-image-wrap">
      <img
        v-if="listing.photos?.length && !imageError"
        :src="listing.photos[0]"
        :alt="listing.title || 'Фото квартиры'"
        class="card-image"
        @error="imageError = true"
      />
      
      <!-- Красивая заглушка, если фото нет -->
      <div v-else class="card-placeholder">
        <svg width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
          <rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect>
          <circle cx="8.5" cy="8.5" r="1.5"></circle>
          <polyline points="21 15 16 10 5 21"></polyline>
        </svg>
        <span>Нет фотографии</span>
      </div>

      <button
        type="button"
        class="fav-btn"
        :class="{ active: isFavorite }"
        :title="isFavorite ? 'Убрать из избранного' : 'Добавить в избранное'"
        @click.stop="emit('toggle-favorite', listing.id)"
      >
        {{ isFavorite ? '♥' : '♡' }}
      </button>
    </div>

    <div class="card-body">
      <div class="card-price">{{ formatPrice(listing.price) }}</div>
      <h3 class="card-title">{{ listing.title || 'Без названия' }}</h3>

      <div class="card-meta">
        <span>{{ formatRooms(listing.rooms) }}</span>
        <span>•</span>
        <span>{{ listing.area ? `${listing.area} м²` : 'Площадь не указана' }}</span>
      </div>

      <div class="card-district">
        📍 {{ listing.district || 'Район не указан' }}
      </div>

      <p class="card-desc">
        {{ listing.description || 'Описание отсутствует.' }}
      </p>
    </div>
  </article>
</template>

<style scoped>
.card {
  background: #ffffff;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  cursor: pointer;
  transition: transform 0.15s ease, box-shadow 0.15s ease;
}

.card:hover {
  transform: translateY(-3px);
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.08);
}

.card-image-wrap {
  position: relative;
  height: 200px;
  background: #f3f4f6;
}

.card-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

/* Стили для красивой заглушки */
.card-placeholder {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 8px;
  color: #9ca3af;
  background: linear-gradient(135deg, #f9fafb 0%, #f3f4f6 100%);
}

.card-placeholder span {
  font-size: 13px;
  font-weight: 500;
}

.fav-btn {
  position: absolute;
  top: 10px;
  right: 10px;
  width: 36px;
  height: 36px;
  border-radius: 50%;
  border: none;
  background: rgba(255, 255, 255, 0.92);
  font-size: 20px;
  line-height: 1;
  cursor: pointer;
  color: #6b7280;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.15);
}

.fav-btn.active {
  color: #ef4444;
}

.card-body {
  padding: 14px 16px 16px;
  display: flex;
  flex-direction: column;
  gap: 6px;
  flex: 1;
}

.card-price {
  font-size: 18px;
  font-weight: 700;
  color: #111827;
}

.card-title {
  margin: 0;
  font-size: 16px;
  font-weight: 600;
  color: #1f2937;
}

.card-meta {
  display: flex;
  gap: 6px;
  font-size: 13px;
  color: #4b5563;
}

.card-district {
  font-size: 13px;
  color: #2563eb;
}

.card-desc {
  margin: 4px 0 0;
  font-size: 13px;
  color: #6b7280;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
</style>
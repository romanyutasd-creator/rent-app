<script setup>
import { ref, watch, onMounted, onUnmounted } from 'vue'

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

const emit = defineEmits(['close', 'toggle-favorite'])

const activePhotoIndex = ref(0)

watch(
  () => props.listing,
  () => {
    activePhotoIndex.value = 0
  }
)

const handleKeydown = (e) => {
  if (e.key === 'Escape') emit('close')
}

onMounted(() => window.addEventListener('keydown', handleKeydown))
onUnmounted(() => window.removeEventListener('keydown', handleKeydown))

const prevPhoto = () => {
  const len = props.listing.photos?.length || 0
  if (len > 1) {
    activePhotoIndex.value = (activePhotoIndex.value - 1 + len) % len
  }
}

const nextPhoto = () => {
  const len = props.listing.photos?.length || 0
  if (len > 1) {
    activePhotoIndex.value = (activePhotoIndex.value + 1) % len
  }
}

const formatPrice = (price) => {
  if (price === null || price === undefined || price === '') return 'Цена не указана'
  return `${Number(price).toLocaleString('ru-RU')} ₽/мес`
}

const formatRooms = (rooms) => {
  if (rooms === null || rooms === undefined) return 'Не указано'
  if (rooms === 0) return 'Студия'
  return String(rooms)
}
</script>

<template>
  <div class="modal-backdrop" @click.self="emit('close')">
    <div class="modal-content">
      <button type="button" class="close-btn" @click="emit('close')">✕</button>

      <div class="gallery">
        <template v-if="listing.photos?.length">
          <img
            :src="listing.photos[activePhotoIndex]"
            :alt="listing.title || 'Фотография'"
            class="gallery-img"
          />
          <template v-if="listing.photos.length > 1">
            <button type="button" class="nav-btn prev" @click="prevPhoto">‹</button>
            <button type="button" class="nav-btn next" @click="nextPhoto">›</button>
            <div class="photo-counter">
              {{ activePhotoIndex + 1 }} / {{ listing.photos.length }}
            </div>
          </template>
        </template>
        
        <div v-else class="gallery-empty">
          <svg width="64" height="64" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.2" stroke-linecap="round" stroke-linejoin="round">
            <rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect>
            <circle cx="8.5" cy="8.5" r="1.5"></circle>
            <polyline points="21 15 16 10 5 21"></polyline>
          </svg>
          <span>Фотографии отсутствуют</span>
        </div>
      </div>

      <div class="modal-body">
        <div class="modal-header">
          <div>
            <div class="modal-price">{{ formatPrice(listing.price) }}</div>
            <h2 class="modal-title">{{ listing.title || 'Без названия' }}</h2>
          </div>

          <button
            type="button"
            class="fav-action-btn"
            :class="{ active: isFavorite }"
            @click="emit('toggle-favorite', listing.id)"
          >
            {{ isFavorite ? '♥ В избранном' : '♡ В избранное' }}
          </button>
        </div>

        <div class="specs-grid">
          <div class="spec-item">
            <span class="spec-label">Комнаты</span>
            <span class="spec-value">{{ formatRooms(listing.rooms) }}</span>
          </div>
          <div class="spec-item">
            <span class="spec-label">Площадь</span>
            <span class="spec-value">{{ listing.area ? `${listing.area} м²` : 'Не указана' }}</span>
          </div>
          <div class="spec-item">
            <span class="spec-label">Район</span>
            <span class="spec-value">{{ listing.district || 'Не указан' }}</span>
          </div>
        </div>

        <div class="description-block">
          <h3 class="section-title">Описание</h3>
          <p class="description-text">
            {{ listing.description || 'Описание отсутствует.' }}
          </p>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.modal-backdrop {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
  z-index: 1000;
}

.modal-content {
  background: #fff;
  border-radius: 16px;
  width: 100%;
  max-width: 700px;
  max-height: 90vh;
  overflow-y: auto;
  position: relative;
  box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1);
}

.close-btn {
  position: absolute;
  top: 16px;
  right: 16px;
  width: 32px;
  height: 32px;
  border-radius: 50%;
  border: none;
  background: rgba(0, 0, 0, 0.4);
  color: white;
  font-size: 16px;
  cursor: pointer;
  z-index: 10;
  display: flex;
  align-items: center;
  justify-content: center;
}

.close-btn:hover {
  background: rgba(0, 0, 0, 0.6);
}

.gallery {
  position: relative;
  height: 350px;
  background: #f3f4f6;
}

.gallery-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

/* Красивая заглушка в модалке */
.gallery-empty {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 12px;
  color: #9ca3af;
  background: linear-gradient(135deg, #f9fafb 0%, #f3f4f6 100%);
  font-size: 16px;
  font-weight: 500;
}

.nav-btn {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  width: 40px;
  height: 40px;
  border-radius: 50%;
  border: none;
  background: rgba(255, 255, 255, 0.8);
  font-size: 24px;
  cursor: pointer;
  color: #374151;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background 0.2s;
}

.nav-btn:hover {
  background: #fff;
}

.nav-btn.prev { left: 16px; }
.nav-btn.next { right: 16px; }

.photo-counter {
  position: absolute;
  bottom: 16px;
  right: 16px;
  background: rgba(0, 0, 0, 0.6);
  color: #fff;
  padding: 4px 10px;
  border-radius: 20px;
  font-size: 12px;
}

.modal-body {
  padding: 24px;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 16px;
  margin-bottom: 24px;
}

.modal-price {
  font-size: 24px;
  font-weight: 700;
  color: #111827;
  margin-bottom: 4px;
}

.modal-title {
  margin: 0;
  font-size: 20px;
  font-weight: 600;
  color: #1f2937;
}

.fav-action-btn {
  padding: 10px 16px;
  border-radius: 8px;
  border: 1px solid #d1d5db;
  background: #fff;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  color: #374151;
  white-space: nowrap;
  transition: all 0.2s;
}

.fav-action-btn.active {
  background: #fef2f2;
  border-color: #fecaca;
  color: #ef4444;
}

.specs-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 16px;
  margin-bottom: 24px;
  background: #f9fafb;
  padding: 16px;
  border-radius: 12px;
}

.spec-item {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.spec-label {
  font-size: 12px;
  color: #6b7280;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.spec-value {
  font-size: 16px;
  font-weight: 600;
  color: #111827;
}

.description-block {
  margin-top: 20px;
}

.section-title {
  margin: 0 0 8px;
  font-size: 16px;
  font-weight: 600;
  color: #374151;
}

.description-text {
  margin: 0;
  font-size: 15px;
  line-height: 1.6;
  color: #4b5563;
}
</style>
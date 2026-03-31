<template>
  <div class="result-page">
    <!-- Header Navigation -->
    <nav class="top-nav">
      <div class="nav-container">
        <div class="back-link" @click="goBack">
          <span class="icon">←</span> 返回搜索
        </div>
        <div class="nav-actions">
          <template v-if="tripPlan">
            <a-button v-if="!editMode" class="action-btn" @click="toggleEditMode">
              <span class="icon">✏️</span> 编辑
            </a-button>
            <div v-else class="edit-group">
              <a-button type="primary" class="action-btn save" @click="saveChanges">保存</a-button>
              <a-button class="action-btn" @click="cancelEdit">取消</a-button>
            </div>
            
            <a-dropdown>
              <template #overlay>
                <a-menu>
                  <a-menu-item key="image" @click="exportAsImage">📷 导出图片</a-menu-item>
                  <a-menu-item key="pdf" @click="exportAsPDF">📄 导出 PDF</a-menu-item>
                </a-menu>
              </template>
              <a-button class="action-btn export">
                📥 导出 <span class="chevron">▼</span>
              </a-button>
            </a-dropdown>
          </template>
        </div>
      </div>
    </nav>

    <div v-if="tripPlan" class="dashboard-container">
      <!-- Hero Banner -->
      <header class="hero-banner">
        <div class="hero-content">
          <div class="trip-meta">
            <h1 class="city-name">{{ tripPlan.city }} {{ tripPlan.days.length }}日游</h1>
            <p class="date-range">
              <span class="icon">📅</span> {{ tripPlan.start_date }} — {{ tripPlan.end_date }}
            </p>
          </div>
          
          <div v-if="tripPlan.budget" class="budget-summary-card">
            <div class="budget-header">
              <span class="label">预计总费用</span>
              <span class="total-amount">¥{{ tripPlan.budget.total }}</span>
            </div>
            <div class="budget-bar">
              <div class="bar-segment attractions" :style="{ width: (tripPlan.budget.total_attractions / tripPlan.budget.total * 100) + '%' }"></div>
              <div class="bar-segment hotels" :style="{ width: (tripPlan.budget.total_hotels / tripPlan.budget.total * 100) + '%' }"></div>
              <div class="bar-segment meals" :style="{ width: (tripPlan.budget.total_meals / tripPlan.budget.total * 100) + '%' }"></div>
              <div class="bar-segment transport" :style="{ width: (tripPlan.budget.total_transportation / tripPlan.budget.total * 100) + '%' }"></div>
            </div>
            <div class="budget-legend">
              <div class="legend-item"><span class="dot attractions"></span> 景点 ¥{{ tripPlan.budget.total_attractions }}</div>
              <div class="legend-item"><span class="dot hotels"></span> 酒店 ¥{{ tripPlan.budget.total_hotels }}</div>
              <div class="legend-item"><span class="dot meals"></span> 餐饮 ¥{{ tripPlan.budget.total_meals }}</div>
              <div class="legend-item"><span class="dot transport"></span> 交通 ¥{{ tripPlan.budget.total_transportation }}</div>
            </div>
          </div>
        </div>
      </header>

      <!-- Suggestions Strip -->
      <section class="suggestions-strip">
        <div class="suggestion-card">
          <span class="icon">💡</span>
          <p>{{ tripPlan.overall_suggestions }}</p>
        </div>
      </section>

      <!-- Weather Scroller -->
      <section v-if="tripPlan.weather_info && tripPlan.weather_info.length" class="weather-section">
        <h3 class="section-title">目的地天气</h3>
        <div class="weather-scroller">
          <div v-for="w in tripPlan.weather_info" :key="w.date" class="weather-mini-card">
            <div class="w-date">{{ formatSimpleDate(w.date) }}</div>
            <div class="w-main">
              <span class="w-icon">{{ getWeatherIcon(w.day_weather) }}</span>
              <span class="w-temp">{{ w.day_temp }}°/{{ w.night_temp }}°</span>
            </div>
            <div class="w-desc">{{ w.day_weather }}</div>
          </div>
        </div>
      </section>

      <div class="main-layout">
        <!-- Itinerary Timeline -->
        <div class="itinerary-column">
          <h3 class="section-title">每日行程安排</h3>
          <div class="timeline-container">
            <div v-for="(day, dIdx) in tripPlan.days" :key="dIdx" class="timeline-day" :id="`day-${dIdx}`">
              <div class="day-marker">
                <div class="marker-circle">{{ dIdx + 1 }}</div>
                <div class="marker-line"></div>
              </div>
              
              <div class="day-content">
                <div class="day-header">
                  <h4 class="day-title">Day {{ dIdx + 1 }}: {{ day.description || '精彩开启' }}</h4>
                  <span class="day-date-label">{{ day.date }}</span>
                </div>

                <!-- Attractions Cards -->
                <div class="attractions-list">
                  <div v-for="(attr, aIdx) in day.attractions" :key="aIdx" class="attraction-item-card">
                    <div class="attr-img">
                      <img :src="getAttractionImage(attr.name, aIdx)" @error="handleImageError" />
                      <div v-if="attr.ticket_price" class="attr-price">¥{{ attr.ticket_price }}</div>
                    </div>
                    <div class="attr-info">
                      <div class="attr-top">
                        <h5 class="attr-name">{{ attr.name }}</h5>
                        <div v-if="editMode" class="attr-actions">
                          <span @click="moveAttraction(dIdx, aIdx, 'up')" :class="{ disabled: aIdx === 0 }">🔼</span>
                          <span @click="moveAttraction(dIdx, aIdx, 'down')" :class="{ disabled: aIdx === day.attractions.length - 1 }">🔽</span>
                          <span @click="deleteAttraction(dIdx, aIdx)" class="delete">❌</span>
                        </div>
                      </div>
                      
                      <div v-if="!editMode" class="attr-details">
                        <p class="attr-addr"><span class="icon">📍</span> {{ attr.address }}</p>
                        <p class="attr-desc">{{ attr.description }}</p>
                        <div class="attr-meta">
                          <span class="tag time">⏳ {{ attr.visit_duration }}min</span>
                          <span v-if="attr.category" class="tag cat">🏷️ {{ attr.category }}</span>
                        </div>
                      </div>
                      <div v-else class="attr-edit">
                        <a-input v-model:value="attr.address" size="small" placeholder="地址" class="mb-1" />
                        <a-textarea v-model:value="attr.description" :rows="2" size="small" placeholder="描述" />
                      </div>
                    </div>
                  </div>
                </div>

                <!-- Hotel and Meals -->
                <div class="day-extras">
                  <div v-if="day.hotel" class="extra-item hotel">
                    <span class="extra-icon">🏨</span>
                    <div class="extra-text">
                      <div class="extra-title">住宿: {{ day.hotel.name }}</div>
                      <div class="extra-sub">{{ day.hotel.type }} · {{ day.hotel.price_range }} · {{ day.hotel.rating }}⭐</div>
                    </div>
                  </div>
                  <div class="extra-item meals">
                    <span class="extra-icon">🍱</span>
                    <div class="extra-text">
                      <div class="extra-title">今日推荐方案</div>
                      <div class="extra-sub">
                        <span v-for="m in day.meals" :key="m.type" class="meal-tag">
                          {{ getMealEmoji(m.type) }}{{ m.name }}
                        </span>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- Sticky Map Column -->
        <div class="map-column">
          <div class="sticky-map-wrapper">
            <h3 class="section-title">行程路线图</h3>
            <div class="map-card-container">
              <div id="amap-container" class="map-canvas"></div>
              <div class="map-controls">
                <div v-for="(_, idx) in tripPlan.days" :key="idx" 
                     class="day-filter" 
                     :class="{ active: activeSection === `day-${idx}` }"
                     @click="scrollToDay(idx)">
                  Day {{ idx + 1 }}
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <a-empty v-else class="empty-state">
      <template #description>
        <p>未找到有效的行程计划</p>
        <a-button type="primary" @click="goBack">回首页重新生成</a-button>
      </template>
    </a-empty>
    
    <a-back-top />
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, nextTick } from 'vue'
import { useRouter } from 'vue-router'
import { message } from 'ant-design-vue'
import AMapLoader from '@amap/amap-jsapi-loader'
import html2canvas from 'html2canvas'
import jsPDF from 'jspdf'
import type { TripPlan } from '@/types'

const router = useRouter()
const tripPlan = ref<TripPlan | null>(null)
const editMode = ref(false)
const originalPlan = ref<TripPlan | null>(null)
const attractionPhotos = ref<Record<string, string>>({})
const activeSection = ref('day-0')
let map: any = null

onMounted(async () => {
  const data = sessionStorage.getItem('tripPlan')
  if (data) {
    tripPlan.value = JSON.parse(data)
    await loadAttractionPhotos()
    await nextTick()
    initMap()
  }
})

const goBack = () => router.push('/')

const scrollToDay = (idx: number) => {
  activeSection.value = `day-${idx}`
  const el = document.getElementById(`day-${idx}`)
  if (el) el.scrollIntoView({ behavior: 'smooth', block: 'start' })
}

const toggleEditMode = () => {
  editMode.value = true
  originalPlan.value = JSON.parse(JSON.stringify(tripPlan.value))
}

const saveChanges = () => {
  editMode.value = false
  if (tripPlan.value) sessionStorage.setItem('tripPlan', JSON.stringify(tripPlan.value))
  message.success('修改已保存')
  if (map) map.destroy()
  nextTick(() => initMap())
}

const cancelEdit = () => {
  if (originalPlan.value) tripPlan.value = JSON.parse(JSON.stringify(originalPlan.value))
  editMode.value = false
}

const deleteAttraction = (dIdx: number, aIdx: number) => {
  if (!tripPlan.value) return
  const day = tripPlan.value.days[dIdx]
  if (day.attractions.length <= 1) return message.warning('每天至少保留一个景点')
  day.attractions.splice(aIdx, 1)
}

const moveAttraction = (dIdx: number, aIdx: number, dir: 'up' | 'down') => {
  if (!tripPlan.value) return
  const attractions = tripPlan.value.days[dIdx].attractions
  if (dir === 'up' && aIdx > 0) {
    [attractions[aIdx], attractions[aIdx - 1]] = [attractions[aIdx - 1], attractions[aIdx]]
  } else if (dir === 'down' && aIdx < attractions.length - 1) {
    [attractions[aIdx], attractions[aIdx + 1]] = [attractions[aIdx + 1], attractions[aIdx]]
  }
}

// Helpers
const formatSimpleDate = (dateStr: string) => {
  const d = new Date(dateStr)
  return `${d.getMonth() + 1}/${d.getDate()}`
}

const getWeatherIcon = (text: string) => {
  if (text.includes('晴')) return '☀️'
  if (text.includes('云')) return '☁️'
  if (text.includes('阴')) return '🌥️'
  if (text.includes('雨')) return '🌧️'
  if (text.includes('雪')) return '❄️'
  return '🌡️'
}

const getMealEmoji = (type: string) => {
  const map: any = { breakfast: '☕', lunch: '🍱', dinner: '🥘', snack: '🍡' }
  return map[type] || '🍴'
}

const loadAttractionPhotos = async () => {
  if (!tripPlan.value) return
  const promises = tripPlan.value.days.flatMap(day => 
    day.attractions.map(attr => 
      fetch(`http://localhost:8000/api/poi/photo?name=${encodeURIComponent(attr.name)}`)
        .then(res => res.json())
        .then(data => {
          if (data.success) attractionPhotos.value[attr.name] = data.data.photo_url
        }).catch(() => {})
    )
  )
  await Promise.all(promises)
}

const getAttractionImage = (name: string, index: number) => {
  if (attractionPhotos.value[name]) return attractionPhotos.value[name]
  const colors = ['#667eea', '#f093fb', '#4facfe', '#43e97b', '#fa709a']
  const color = colors[index % colors.length]
  const svg = `<svg xmlns="http://www.w3.org/2000/svg" width="200" height="150">
    <rect width="100%" height="100%" fill="${color}"/>
    <text x="50%" y="50%" dominant-baseline="middle" text-anchor="middle" font-size="16" fill="white">${name}</text>
  </svg>`
  return `data:image/svg+xml;base64,${btoa(unescape(encodeURIComponent(svg)))}`
}

const handleImageError = (e: any) => {
  e.target.src = 'data:image/svg+xml,%3Csvg xmlns="http://www.w3.org/2000/svg" width="200" height="150"%3E%3Crect width="100%25" height="100%25" fill="%23eee"/%3E%3C/svg%3E'
}

// Map Logic
const initMap = async () => {
  try {
    const AMap = await AMapLoader.load({
      key: import.meta.env.VITE_AMAP_WEB_JS_KEY,
      version: '2.0',
      plugins: ['AMap.Marker', 'AMap.Polyline']
    })
    map = new AMap.Map('amap-container', { zoom: 12, viewMode: '3D' })
    addMarkers(AMap)
  } catch (err) { console.error('Map failed', err) }
}

const addMarkers = (AMap: any) => {
  if (!tripPlan.value) return
  const points: any[] = []
  tripPlan.value.days.forEach((day, dIdx) => {
    day.attractions.forEach((attr, aIdx) => {
      if (attr.location?.longitude) {
        const marker = new AMap.Marker({
          position: [attr.location.longitude, attr.location.latitude],
          label: { content: `<div class="map-label">${dIdx + 1}-${aIdx + 1}</div>`, offset: new AMap.Pixel(0, -30) }
        })
        map.add(marker)
        points.push([attr.location.longitude, attr.location.latitude])
      }
    })
  })
  if (points.length > 1) {
    const polyline = new AMap.Polyline({ path: points, strokeColor: '#4f46e5', strokeWeight: 5, strokeOpacity: 0.8, showDir: true })
    map.add(polyline)
    map.setFitView()
  }
}

// Export Logic (Compact)
const exportAsImage = async () => {
  message.loading({ content: '生成图片中...', key: 'export' })
  const el = document.querySelector('.dashboard-container') as HTMLElement
  const canvas = await html2canvas(el, { useCORS: true, scale: 2 })
  const link = document.createElement('a')
  link.download = `Trip_${tripPlan.value?.city}.png`
  link.href = canvas.toDataURL()
  link.click()
  message.success({ content: '导出完成', key: 'export' })
}

const exportAsPDF = async () => {
  message.loading({ content: '生成 PDF 中...', key: 'export' })
  const el = document.querySelector('.dashboard-container') as HTMLElement
  const canvas = await html2canvas(el, { scale: 1.5 })
  const imgData = canvas.toDataURL('image/png')
  const pdf = new jsPDF('p', 'mm', 'a4')
  const w = 210
  const h = (canvas.height * w) / canvas.width
  pdf.addImage(imgData, 'PNG', 0, 0, w, h)
  pdf.save(`Trip_${tripPlan.value?.city}.pdf`)
  message.success({ content: '导出完成', key: 'export' })
}
</script>

<style scoped>
.result-page {
  background-color: #f6f8fb;
  min-height: 100vh;
  color: #1e293b;
  font-family: 'Inter', sans-serif;
}

/* Nav */
.top-nav {
  background: white;
  height: 64px;
  display: flex;
  align-items: center;
  position: sticky;
  top: 0;
  z-index: 100;
  box-shadow: 0 1px 3px rgba(0,0,0,0.05);
}
.nav-container {
  max-width: 1400px;
  margin: 0 auto;
  width: 100%;
  padding: 0 24px;
  display: flex;
  justify-content: space-between;
}
.back-link {
  cursor: pointer;
  font-weight: 500;
  color: #64748b;
  transition: color 0.2s;
}
.back-link:hover { color: #4f46e5; }
.nav-actions { display: flex; gap: 12px; }
.action-btn {
  border-radius: 8px;
  font-weight: 500;
}
.edit-group { display: flex; gap: 8px; }

/* Hero */
.dashboard-container {
  max-width: 1400px;
  margin: 0 auto;
  padding: 32px 24px;
}
.hero-banner {
  background: linear-gradient(135deg, #4f46e5 0%, #7c3aed 100%);
  border-radius: 24px;
  padding: 40px;
  color: white;
  margin-bottom: 24px;
  box-shadow: 0 10px 25px -5px rgba(79, 70, 229, 0.4);
}
.hero-content {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
}
.city-name { font-size: 48px; font-weight: 800; color: white; margin: 0; line-height: 1.2; }
.date-range { font-size: 18px; opacity: 0.9; margin: 12px 0 0 0; }

/* Budget Card */
.budget-summary-card {
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(10px);
  border-radius: 16px;
  padding: 24px;
  width: 360px;
  border: 1px solid rgba(255, 255, 255, 0.2);
}
.budget-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 16px; }
.budget-header .label { font-size: 14px; opacity: 0.8; }
.total-amount { font-size: 28px; font-weight: 700; }
.budget-bar { height: 8px; border-radius: 4px; display: flex; overflow: hidden; margin-bottom: 16px; background: rgba(0,0,0,0.1); }
.bar-segment.attractions { background: #fbbf24; }
.bar-segment.hotels { background: #60a5fa; }
.bar-segment.meals { background: #f87171; }
.bar-segment.transport { background: #34d399; }
.budget-legend { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; }
.legend-item { font-size: 12px; display: flex; align-items: center; gap: 6px; }
.dot { width: 8px; height: 8px; border-radius: 50%; }
.dot.attractions { background: #fbbf24; }
.dot.hotels { background: #60a5fa; }
.dot.meals { background: #f87171; }
.dot.transport { background: #34d399; }

/* Suggestions */
.suggestions-strip { margin-bottom: 32px; }
.suggestion-card {
  background: white;
  border-radius: 16px;
  padding: 20px 24px;
  display: flex;
  gap: 16px;
  align-items: flex-start;
  box-shadow: 0 4px 6px -1px rgba(0,0,0,0.05);
  border-left: 4px solid #4f46e5;
}
.suggestion-card p { margin: 0; line-height: 1.6; color: #475569; }

/* Weather */
.section-title { font-size: 20px; font-weight: 700; margin-bottom: 20px; }
.weather-section { margin-bottom: 40px; }
.weather-scroller { display: flex; gap: 16px; overflow-x: auto; padding-bottom: 8px; }
.weather-scroller::-webkit-scrollbar { height: 6px; }
.weather-scroller::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 3px; }
.weather-mini-card {
  background: white;
  min-width: 140px;
  padding: 20px;
  border-radius: 16px;
  text-align: center;
  box-shadow: 0 4px 6px -1px rgba(0,0,0,0.05);
}
.w-date { font-size: 14px; font-weight: 600; color: #64748b; margin-bottom: 8px; }
.w-main { display: flex; align-items: center; justify-content: center; gap: 8px; margin-bottom: 4px; }
.w-icon { font-size: 24px; }
.w-temp { font-weight: 700; font-size: 18px; }
.w-desc { font-size: 12px; color: #94a3b8; }

/* Main Layout */
.main-layout { display: flex; gap: 32px; }
.itinerary-column { flex: 1; }
.map-column { width: 480px; }

/* Timeline */
.timeline-container { position: relative; padding-left: 20px; }
.timeline-day { display: flex; gap: 24px; margin-bottom: 48px; position: relative; }
.day-marker { flex-shrink: 0; width: 40px; position: relative; }
.marker-circle {
  width: 40px; height: 40px; border-radius: 50%;
  background: #4f46e5; color: white;
  display: flex; align-items: center; justify-content: center;
  font-weight: 700; font-size: 18px;
  position: relative; z-index: 2;
  box-shadow: 0 4px 10px rgba(79, 70, 229, 0.3);
}
.marker-line {
  position: absolute; top: 40px; bottom: -48px; left: 50%;
  width: 2px; background: #e2e8f0; transform: translateX(-50%);
}
.timeline-day:last-child .marker-line { display: none; }

.day-content { flex: 1; min-width: 0; }
.day-header { margin-bottom: 24px; }
.day-title { font-size: 24px; font-weight: 700; margin: 0 0 4px 0; }
.day-date-label { color: #94a3b8; font-size: 14px; font-weight: 500; }

/* Attraction Card */
.attractions-list { display: flex; flex-direction: column; gap: 16px; margin-bottom: 24px; }
.attraction-item-card {
  background: white; border-radius: 16px; overflow: hidden; display: flex;
  box-shadow: 0 4px 6px -1px rgba(0,0,0,0.05); border: 1px solid #f1f5f9;
  transition: transform 0.2s;
}
.attraction-item-card:hover { transform: translateY(-2px); box-shadow: 0 10px 15px -3px rgba(0,0,0,0.1); }

.attr-img { width: 220px; height: 160px; position: relative; flex-shrink: 0; }
.attr-img img { width: 100%; height: 100%; object-fit: cover; }
.attr-price {
  position: absolute; top: 12px; left: 12px;
  background: rgba(0,0,0,0.6); color: white;
  padding: 4px 10px; border-radius: 8px; font-size: 12px; font-weight: 700;
}

.attr-info { flex: 1; padding: 20px; display: flex; flex-direction: column; }
.attr-top { display: flex; justify-content: space-between; align-items: flex-start; margin-bottom: 8px; }
.attr-name { font-size: 18px; font-weight: 700; margin: 0; color: #1e293b; }
.attr-actions { display: flex; gap: 8px; cursor: pointer; }
.attr-details p { font-size: 14px; color: #64748b; margin: 0 0 8px 0; line-height: 1.5; }
.attr-desc {
  display: -webkit-box; -webkit-line-clamp: 2; -webkit-box-orient: vertical;
  overflow: hidden; margin-bottom: 12px !important;
}
.attr-meta { display: flex; gap: 8px; }
.tag { font-size: 11px; padding: 4px 8px; border-radius: 6px; font-weight: 600; }
.tag.time { background: #f1f5f9; color: #475569; }
.tag.cat { background: #e0e7ff; color: #4338ca; }

/* Extras */
.day-extras { background: #f8fafc; border-radius: 16px; padding: 20px; display: flex; flex-direction: column; gap: 16px; }
.extra-item { display: flex; gap: 16px; align-items: flex-start; }
.extra-icon { font-size: 24px; }
.extra-title { font-weight: 600; font-size: 15px; }
.extra-sub { font-size: 13px; color: #64748b; margin-top: 2px; }
.meal-tag { margin-right: 12px; display: inline-flex; align-items: center; gap: 4px; }

/* Map Column */
.sticky-map-wrapper { position: sticky; top: 96px; }
.map-card-container {
  background: white; border-radius: 20px; overflow: hidden;
  box-shadow: 0 10px 15px -3px rgba(0,0,0,0.1); border: 1px solid #f1f5f9;
}
.map-canvas { height: 500px; width: 100%; }
.map-controls {
  display: flex; overflow-x: auto; padding: 12px; gap: 8px; background: white;
  border-top: 1px solid #f1f5f9;
}
.day-filter {
  padding: 6px 16px; border-radius: 20px; background: #f1f5f9;
  font-size: 13px; font-weight: 600; color: #64748b; cursor: pointer;
  white-space: nowrap; transition: all 0.2s;
}
.day-filter.active { background: #4f46e5; color: white; }
.day-filter:hover:not(.active) { background: #e2e8f0; }

:deep(.map-label) {
  background: #4f46e5; color: white; padding: 2px 8px;
  border-radius: 10px; font-size: 10px; font-weight: 800;
  box-shadow: 0 2px 4px rgba(0,0,0,0.2);
}

.empty-state { padding-top: 100px; }
.mb-1 { margin-bottom: 8px; }
</style>

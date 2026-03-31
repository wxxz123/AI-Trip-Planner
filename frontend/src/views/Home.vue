<template>
  <div class="home-container">
    <div class="stars"></div>
    <div class="twinkling"></div>

    <!-- Navigation Header -->
    <header class="app-header">
      <div class="logo">
        <span class="logo-icon">🌍</span>
        <span class="logo-text">AI Travel Planner</span>
      </div>
    </header>

    <main class="main-content">
      <div class="hero-section">
        <h1 class="hero-title">AI 智能行程规划专家</h1>
        <p class="hero-subtitle">只需几秒，开启您的梦想之旅</p>
      </div>

      <div class="glass-card">
        <div class="card-header">
          <h2>开始规划您的精彩旅程</h2>
        </div>

        <a-form :model="formData" layout="vertical" @finish="handleSubmit" class="premium-form" hideRequiredMark>
          
          <div class="form-group-title">基础信息</div>
          <a-row :gutter="24">
            <a-col :span="12">
              <a-form-item name="city" :rules="[{ required: true, message: '请输入目的地城市' }]">
                <template #label><span class="glass-label">目的地城市</span></template>
                <a-input v-model:value="formData.city" placeholder="例如: 东京、北京" size="large" class="glass-input city-input">
                  <template #prefix><span class="input-icon">📍</span></template>
                </a-input>
              </a-form-item>
            </a-col>
            <a-col :span="12">
              <a-form-item name="date_range" :rules="[{ required: true, message: '请选择往返日期' }]">
                <template #label><span class="glass-label">旅行日期范围 (<span class="highlight-days">{{ formData.travel_days || 1 }} 天</span>)</span></template>
                <a-range-picker v-model:value="dateRange" style="width: 100%" size="large" class="glass-picker" :placeholder="['开始日期', '结束日期']" format="YYYY-MM-DD" />
              </a-form-item>
            </a-col>
          </a-row>

          <div class="form-group-title st-mt">出行偏好</div>
          <a-row :gutter="24">
            <a-col :span="12">
              <a-form-item name="transportation">
                <template #label><span class="glass-label">交通方式</span></template>
                <a-select v-model:value="formData.transportation" size="large" class="glass-select" :dropdownClassName="'glass-dropdown'">
                  <a-select-option value="公共交通">🚇 公共交通</a-select-option>
                  <a-select-option value="自驾">🚗 自驾</a-select-option>
                  <a-select-option value="步行">🚶 步行</a-select-option>
                  <a-select-option value="混合">🔀 混合</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
            <a-col :span="12">
              <a-form-item name="accommodation">
                <template #label><span class="glass-label">住宿标准</span></template>
                <a-select v-model:value="formData.accommodation" size="large" class="glass-select" :dropdownClassName="'glass-dropdown'">
                  <a-select-option value="经济型酒店">💰 经济型酒店</a-select-option>
                  <a-select-option value="舒适型酒店">🏨 舒适型酒店</a-select-option>
                  <a-select-option value="豪华酒店">⭐ 豪华酒店</a-select-option>
                  <a-select-option value="民宿">🏡 特色民宿</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
          </a-row>

          <a-form-item name="preferences">
            <template #label>
              <div class="glass-label mb-1">旅行风格 (多选)</div>
            </template>
            <div class="pill-group">
              <div 
                v-for="opt in prefOptions" 
                :key="opt.value" 
                class="pill-toggle" 
                :class="{ active: formData.preferences.includes(opt.value) }"
                @click="togglePreference(opt.value)"
              >
                {{ opt.icon }} {{ opt.label }}
              </div>
            </div>
          </a-form-item>

          <a-form-item name="free_text_input">
            <template #label><span class="glass-label">特殊要求 (选填)</span></template>
            <a-textarea v-model:value="formData.free_text_input" placeholder="例如: 需要无障碍通道、带了老人小孩、对海鲜过敏..." :rows="2" size="large" class="glass-textarea" />
          </a-form-item>

          <a-form-item class="submit-item">
            <a-button type="primary" html-type="submit" :loading="loading" size="large" block class="glow-btn">
              <template v-if="!loading">
                生成我的专属行程 <span class="btn-arrow">↗</span>
              </template>
              <template v-else>
                AI 正在为您深度规划中...
              </template>
            </a-button>
          </a-form-item>
          
          <div v-if="loading" class="loading-container">
            <a-progress
              :percent="loadingProgress"
              status="active"
              :stroke-color="{
                '0%': '#8b5cf6',
                '100%': '#3b82f6',
              }"
              :stroke-width="8"
              class="glass-progress"
            />
            <div class="loading-status-text">
              {{ loadingStatus }}
            </div>
          </div>

        </a-form>
      </div>
    </main>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, watch } from 'vue'
import { useRouter } from 'vue-router'
import { message } from 'ant-design-vue'
import { generateTripPlan } from '@/services/api'
import type { TripFormData } from '@/types'
import type { Dayjs } from 'dayjs'

const router = useRouter()
const loading = ref(false)
const loadingStatus = ref('')
const loadingProgress = ref(0)

const dateRange = ref<[Dayjs, Dayjs] | null>(null)

const prefOptions = [
  { label: '文化游览', value: '历史文化', icon: '🏛️' },
  { label: '美食探索', value: '美食', icon: '🍜' },
  { label: '自然风光', value: '自然风光', icon: '🏞️' },
  { label: '购物休闲', value: '购物', icon: '🛍️' },
  { label: '艺术展览', value: '艺术', icon: '🎨' },
  { label: '家庭亲子', value: '亲子', icon: '👨‍👩‍👧' },
]

interface TripFormState extends TripFormData {
  date_range: [Dayjs, Dayjs] | null
}

const formData = reactive<TripFormState>({
  city: '',
  start_date: '',
  end_date: '',
  date_range: null,
  travel_days: 1,
  transportation: '公共交通',
  accommodation: '舒适型酒店',
  preferences: [],
  free_text_input: ''
})

const togglePreference = (val: string) => {
  const idx = formData.preferences.indexOf(val)
  if (idx > -1) {
    formData.preferences.splice(idx, 1)
  } else {
    formData.preferences.push(val)
  }
}

watch(dateRange, (val) => {
  if (val && val[0] && val[1]) {
    formData.date_range = val
    formData.start_date = val[0].format('YYYY-MM-DD')
    formData.end_date = val[1].format('YYYY-MM-DD')
    const days = val[1].diff(val[0], 'day') + 1
    if (days > 0 && days <= 30) {
      formData.travel_days = days
    } else if (days > 30) {
      message.warning('旅行天数不能超过30天')
      dateRange.value = null
      formData.date_range = null
      formData.travel_days = 1
    }
  } else {
    formData.date_range = null
    formData.start_date = ''
    formData.end_date = ''
    formData.travel_days = 1
  }
})

const handleSubmit = async () => {
  loading.value = true
  loadingStatus.value = '🔍 正在全网搜索最优质景点...'
  loadingProgress.value = 10
  
  const progressInterval = setInterval(() => {
    if (loadingProgress.value < 90) {
      loadingProgress.value += 5
      if (loadingProgress.value === 30) loadingStatus.value = '🌤️ 正在分析当地天气和最佳游览路线...'
      if (loadingProgress.value === 60) loadingStatus.value = '🏨 正在匹配高性价比住宿...'
      if (loadingProgress.value === 80) loadingStatus.value = '🤖 AI 正在合成最终行程表...'
    }
  }, 800)

  try {
    const response = await generateTripPlan(formData)
    clearInterval(progressInterval)
    loadingProgress.value = 100
    loadingProgress.value = 100
    loadingStatus.value = '✅ 规划完成!'

    if (response.success && response.data) {
      sessionStorage.setItem('tripPlan', JSON.stringify(response.data))
      message.success('旅行计划生成成功!')
      setTimeout(() => {
        router.push('/result')
      }, 500)
    } else {
      message.error(response.message || '生成失败')
    }
  } catch (error: any) {
    clearInterval(progressInterval)
    message.error(error.message || '生成旅行计划失败,请稍后重试')
  } finally {
    setTimeout(() => {
      loading.value = false
      loadingStatus.value = ''
      loadingProgress.value = 0
    }, 1000)
  }
}
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');

.home-container {
  min-height: 100vh;
  background-color: #0b071e;
  background-image: 
    radial-gradient(ellipse at top left, rgba(74, 30, 137, 0.4) 0%, transparent 40%),
    radial-gradient(ellipse at bottom right, rgba(29, 78, 216, 0.3) 0%, transparent 50%);
  position: relative;
  overflow: hidden;
  font-family: 'Inter', -apple-system, sans-serif;
  color: #fff;
}

/* 简单的微粒背景效果 */
.stars, .twinkling {
  position: absolute;
  top: 0; left: 0; right: 0; bottom: 0;
  width: 100%; height: 100%;
  pointer-events: none;
}
.stars {
  background: transparent url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="200" height="200"><circle cx="20" cy="20" r="1" fill="rgba(255,255,255,0.3)"/><circle cx="150" cy="80" r="1.5" fill="rgba(255,255,255,0.4)"/><circle cx="80" cy="180" r="1" fill="rgba(255,255,255,0.2)"/></svg>') repeat;
  z-index: 0;
}

.app-header {
  padding: 24px 40px;
  position: relative;
  z-index: 10;
}
.logo {
  display: flex;
  align-items: center;
  gap: 12px;
  font-weight: 700;
  font-size: 20px;
  letter-spacing: -0.5px;
}
.logo-icon { font-size: 24px; }

.main-content {
  position: relative;
  z-index: 10;
  max-width: 640px;
  margin: 40px auto 80px;
  padding: 0 24px;
}

.hero-section {
  text-align: center;
  margin-bottom: 40px;
  animation: fadeInDown 0.8s ease-out;
}
.hero-title {
  font-size: 42px;
  font-weight: 800;
  color: #ffffff;
  margin-bottom: 12px;
  letter-spacing: -1px;
}
.hero-subtitle {
  font-size: 16px;
  color: rgba(255, 255, 255, 0.7);
  font-weight: 400;
}

.glass-card {
  background: rgba(255, 255, 255, 0.03);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 24px;
  padding: 40px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.4), inset 0 1px 0 rgba(255,255,255,0.1);
  animation: fadeInUp 0.8s ease-out;
}

.card-header h2 {
  color: #fff;
  font-size: 20px;
  font-weight: 600;
  margin-bottom: 30px;
  text-align: center;
}

.form-group-title {
  font-size: 12px;
  text-transform: uppercase;
  letter-spacing: 1px;
  color: rgba(255,255,255,0.4);
  margin-bottom: 16px;
  font-weight: 600;
}
.st-mt {
  margin-top: 12px;
}

.glass-label {
  color: rgba(255, 255, 255, 0.85);
  font-size: 14px;
  font-weight: 500;
}
.highlight-days {
  color: #a78bfa;
  font-weight: 700;
}

/* 深色表单覆盖 */
:deep(.ant-input),
:deep(.ant-input-affix-wrapper),
:deep(.ant-picker),
:deep(.ant-select-selector) {
  background: rgba(0, 0, 0, 0.2) !important;
  border: 1px solid rgba(255, 255, 255, 0.1) !important;
  color: #fff !important;
  border-radius: 12px !important;
  transition: all 0.3s ease;
  padding-top: 8px !important;
  padding-bottom: 8px !important;
  height: auto !important;
}

:deep(.ant-input-affix-wrapper > input.ant-input) {
  background: transparent !important;
  border: none !important;
  color: #fff !important;
  padding: 0 !important;
  box-shadow: none !important;
}

:deep(.ant-input:focus), :deep(.ant-input:hover),
:deep(.ant-input-affix-wrapper:focus), :deep(.ant-input-affix-wrapper:hover),
:deep(.ant-picker:focus), :deep(.ant-picker:hover),
:deep(.ant-select-selector:hover), :deep(.ant-select-focused .ant-select-selector) {
  border-color: #a78bfa !important;
  background: rgba(0, 0, 0, 0.3) !important;
  box-shadow: 0 0 0 2px rgba(167, 139, 250, 0.2) !important;
}

:deep(.ant-input::placeholder),
:deep(.ant-picker-input > input::placeholder),
:deep(.ant-select-selection-placeholder) {
  color: rgba(255, 255, 255, 0.3) !important;
}

:deep(.ant-picker-separator), :deep(.ant-picker-suffix) {
  color: rgba(255, 255, 255, 0.5) !important;
}

/* 城市输入框前缀 icon */
:deep(.city-input .ant-input-prefix) {
  color: rgba(255, 255, 255, 0.6) !important;
}

/* 日期范围选择器输入框内文字 */
:deep(.ant-picker-input > input) {
  color: #fff !important;
  font-size: 14px !important;
}

:deep(.ant-select-selection-item) {
  color: #fff !important;
  line-height: normal !important;
  display: flex;
  align-items: center;
}
:deep(.ant-select-arrow) {
  color: rgba(255,255,255,0.5) !important;
}

/* 覆盖 textarea 高度补丁 */
:deep(.ant-input.glass-textarea) {
  padding: 12px !important;
}

/* Pill Group Toggle */
.mb-1 { margin-bottom: 8px; }
.pill-group {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}
.pill-toggle {
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  color: rgba(255, 255, 255, 0.7);
  padding: 8px 16px;
  border-radius: 20px;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.2s ease;
  user-select: none;
}
.pill-toggle:hover {
  background: rgba(255, 255, 255, 0.1);
  color: #fff;
}
.pill-toggle.active {
  background: linear-gradient(135deg, #7c3aed 0%, #3b82f6 100%);
  border-color: transparent;
  color: #fff;
  box-shadow: 0 4px 12px rgba(124, 58, 237, 0.3);
}

.submit-item {
  margin-top: 32px;
  margin-bottom: 24px;
}

.loading-container {
  margin-top: 24px;
  padding: 20px;
  background: rgba(255, 255, 255, 0.05);
  border-radius: 16px;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

:deep(.glass-progress .ant-progress-inner) {
  background: rgba(0, 0, 0, 0.2) !important;
}
:deep(.glass-progress .ant-progress-text) {
  color: #a78bfa !important;
  font-weight: 700;
}

.glow-btn {
  background: linear-gradient(135deg, #8b5cf6 0%, #3b82f6 100%) !important;
  border: none !important;
  height: 56px !important;
  border-radius: 28px !important;
  font-size: 16px !important;
  font-weight: 600 !important;
  color: #fff !important;
  box-shadow: 0 8px 20px rgba(139, 92, 246, 0.4) !important;
  transition: all 0.3s ease !important;
  display: flex;
  align-items: center;
  justify-content: center;
}
.glow-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 12px 28px rgba(139, 92, 246, 0.6) !important;
}
.btn-arrow {
  margin-left: 8px;
  transition: transform 0.3s ease;
}
.glow-btn:hover .btn-arrow {
  transform: translateX(4px) translateY(-4px);
}

.loading-status-text {
  text-align: center;
  color: #a78bfa;
  font-size: 14px;
  font-weight: 500;
  margin-top: 16px;
  animation: pulse 2s infinite;
}

@keyframes fadeInDown {
  from { opacity: 0; transform: translateY(-20px); }
  to { opacity: 1; transform: translateY(0); }
}
@keyframes fadeInUp {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}
@keyframes pulse {
  0% { opacity: 0.6; }
  50% { opacity: 1; }
  100% { opacity: 0.6; }
}

@media (max-width: 768px) {
  .hero-title { font-size: 32px; }
  .glass-card { padding: 24px; }
}
</style>

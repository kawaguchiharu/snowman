<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import SnowmanSvg from '../components/snowmansvg.vue'
import CityBackground from '../components/citybackground.vue'

const route = useRoute()
const router = useRouter()
const result = ref(null)
const isLoading = ref(true)
const errorMsg = ref('')

const STAGE_HEIGHT_PX = 450 // ステージの高さ

onMounted(async () => {
  const { pref, date, type } = route.query
  if (!pref) { isLoading.value = false; return; }

  try {
    const response = await fetch('http://localhost:8000/calculate', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ prefecture_id: parseInt(pref), target_date: date, period_type: type })
    })
    const data = await response.json()
    result.value = data
  } catch (e) { errorMsg.value = "エラー" } finally { isLoading.value = false }
})

// ★カメラのズーム倍率計算 (雪だるま主役モード)
const pixelPerMeter = computed(() => {
  if (!result.value) return 10
  
  const h = result.value.height_m
  
  // 雪だるまを画面の7〜8割くらいの高さに表示する設定
  // どんなに小さくても、最低「3メートル」分の空間は映す (寄りすぎ防止)
  const viewHeightMeters = Math.max(h * 1.3, 3.0) 
  
  return STAGE_HEIGHT_PX / viewHeightMeters
})

const goBack = () => router.push('/')
</script>

<template>
  <div class="result-page">
    <div v-if="isLoading" class="loading-view"><div class="spinner">❄️</div><h2>雪を集めています...</h2></div>
    <div v-else-if="errorMsg" class="error-view"><h2>⚠️ エラー</h2><p>{{ errorMsg }}</p><button @click="goBack">戻る</button></div>

    <div v-else-if="result" class="content-view">
      <div class="header-area">
        <h1>🎉 完成！</h1>
        <p class="pref-msg">{{ result.message }}</p>
      </div>

      <div class="visual-stage">
        <CityBackground :scale="pixelPerMeter" />

        <div class="figure-wrapper standard street-position">
          <SnowmanSvg :height-px="1.0 * pixelPerMeter" />
          <div class="label">基準(1m)</div>
        </div>

        <div class="figure-wrapper result giant-position">
          <SnowmanSvg :height-px="result.height_m * pixelPerMeter" />
          <div class="label highlight">{{ result.height_m.toFixed(1) }}m</div>
        </div>
      </div>

      <div class="stats-box">
        <div class="stat-item">
          <span class="sub">高さ</span>
          <span class="val">{{ result.height_m.toFixed(1) }}</span>
          <span class="unit">m</span>
        </div>
        <div class="stat-item">
          <span class="sub">体積</span>
          <span class="val">{{ result.volume_m3.toLocaleString() }}</span>
          <span class="unit">m³</span>
        </div>
      </div>
      <button class="back-btn" @click="goBack">もう一度作る</button>
    </div>
  </div>
</template>

<style scoped>
.visual-stage {
  position: relative;
  width: 100%;
  height: 450px;
  overflow: hidden; /* ★ここ重要！ これがないとビルが画面外にはみ出します */
  border-bottom: 10px solid #3e2723;
  margin-bottom: 30px;
  border-radius: 12px;
  background-color: #E0F7FA;
}
.figure-wrapper {
  position: absolute; bottom: 20px; display: flex; flex-direction: column; align-items: center; transition: all 0.5s ease-out;
}
.street-position { left: 15%; z-index: 20; }
.giant-position { left: 60%; transform: translateX(-50%); z-index: 15; }

/* 共通スタイル (省略なし版と同じ) */
.result-page { min-height: 100vh; background: #e1f5fe; padding: 20px; text-align: center; font-family: sans-serif; color: #333; }
.content-view { max-width: 900px; margin: 0 auto; animation: slideIn 0.5s; }
.loading-view { margin-top: 100px; color: #0288d1; }
.spinner { font-size: 3rem; animation: spin 1s infinite linear; display: block; margin-bottom: 20px;}
@keyframes spin { 100% { transform: rotate(360deg); } }
@keyframes slideIn { from { opacity: 0; transform: translateY(20px); } to { opacity: 1; transform: translateY(0); } }
.header-area h1 { color: #e65100; }
.pref-msg { background: white; padding: 5px 15px; border-radius: 15px; display: inline-block; color: #0277bd; font-weight: bold; margin-top: 10px;}
.label { margin-top: 5px; font-weight: bold; font-size: 0.8rem; background: rgba(255,255,255,0.9); padding: 4px 8px; border-radius: 4px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); white-space: nowrap;}
.label.highlight { color: #e65100; font-size: 1.2rem; border: 2px solid #e65100; }
.stats-box { background: white; padding: 20px; border-radius: 10px; margin-bottom: 20px; display: inline-flex; gap: 40px; }
.stat-item { display: flex; flex-direction: column; }
.val { font-size: 1.5rem; font-weight: bold; }
.back-btn { padding: 15px 40px; background: #0288d1; color: white; border: none; border-radius: 30px; font-size: 1.1rem; cursor: pointer; font-weight: bold; transition: transform 0.2s; }
.back-btn:hover { transform: scale(1.05); }
</style>
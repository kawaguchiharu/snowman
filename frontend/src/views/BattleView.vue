
<script setup>
import { ref, computed, nextTick } from 'vue'
import { useRouter } from 'vue-router'

// コンポーネント
import JapanMap3D from '../components/japanmap_3D.vue'
import SmartCalendar from '../components/calendar.vue'
import SnowEffect from '../components/Snoweffect.vue'
import OceanBackground from '../components/oceanbackground.vue'

const router = useRouter()

// ==========================================
// 状態管理
// ==========================================
const showModal = ref(false)
const currentSelectingPref = ref(null)
const calendarDate = ref(new Date('2024-01-01'))

const battleMode = ref('daily') 
const activePlayer = ref(1)

const player1 = ref({ pref: null, date: null, label: '日付未選択' })
const player2 = ref({ pref: null, date: null, label: '日付未選択' })

const errorMessage = ref('')

// ==========================================
// ロジック
// ==========================================

const goBack = () => {
  router.push('/')
}

const handleMapSelect = (pin) => {
  if (activePlayer.value === 1) {
    player1.value.pref = pin
    // 地図を選んだ直後は日付リセット
    player1.value.date = null
    player1.value.label = '日付を選んでね'
  } else {
    player2.value.pref = pin
    player2.value.date = null
    player2.value.label = '日付を選んでね'
  }
  currentSelectingPref.value = pin
  showModal.value = true
  errorMessage.value = ''
}

const closeModal = () => showModal.value = false

const handleDateSelect = async (type) => {
  await nextTick()
  
  let label = ''
  let dateVal = new Date(calendarDate.value)

  if (type === 'day') {
    label = `${dateVal.getFullYear()}年${dateVal.getMonth() + 1}月${dateVal.getDate()}日`
    battleMode.value = 'daily'
  } else if (type === 'month') {
    label = `${dateVal.getFullYear()}年 ${dateVal.getMonth() + 1}月`
    battleMode.value = 'monthly'
  } else if (type === 'year') {
    label = `${dateVal.getFullYear()}年`
    battleMode.value = 'yearly'
  }

  const data = {
    date: formatDate(dateVal),
    label: label,
    type: type
  }

  if (activePlayer.value === 1) {
    player1.value.date = data.date
    player1.value.label = data.label // ★ここでラベルを更新！
    closeModal()
    activePlayer.value = 2
  } else {
    player2.value.date = data.date
    player2.value.label = data.label // ★ここでラベルを更新！
    closeModal()
  }
}

const selectPlayer = (num) => {
  activePlayer.value = num
}

const formatDate = (date) => {
  if (!date) return ''
  const d = new Date(date)
  return `${d.getFullYear()}-${String(d.getMonth() + 1).padStart(2, '0')}-${String(d.getDate()).padStart(2, '0')}`
}

const startBattle = () => {
  if (!player1.value.pref || !player1.value.date) {
    errorMessage.value = '先攻（赤）の場所と日付が決まっていません！'
    return
  }
  if (!player2.value.pref || !player2.value.date) {
    errorMessage.value = '後攻（青）の場所と日付が決まっていません！'
    return
  }

  router.push({
    path: '/battle-result',
    query: {
      mode: battleMode.value,
      p1_id: player1.value.pref.id,
      p1_name: player1.value.pref.name,
      p1_date: player1.value.date,
      p1_label: player1.value.label, // ★表示用ラベルも渡す
      p2_id: player2.value.pref.id,
      p2_name: player2.value.pref.name,
      p2_date: player2.value.date,
      p2_label: player2.value.label // ★表示用ラベルも渡す
    }
  })
}

// 案内テキスト
const instructionText = computed(() => {
  if (activePlayer.value === 1) {
    if (!player1.value.pref) return '🔴 先攻の <b>都道府県</b> を選んでください'
    return '🔴 先攻の <b>日付</b> を選んでください'
  } else {
    // 1Pの選択が終わっていたら、その内容を表示
    const p1Text = player1.value.pref 
      ? `🔴 ${player1.value.pref.name} <small>(${player1.value.label})</small> 決定！<br>` 
      : ''
    
    if (!player2.value.pref) return `${p1Text}次は 🔵 後攻の <b>都道府県</b> を選んでください`
    return '🔵 後攻の <b>日付</b> を選んでください'
  }
})
</script>

<template>
  <SnowEffect />
  
  <div class="battle-container">
    <OceanBackground />

    <button class="back-btn" @click="goBack">⬅ TOP</button>

    <div class="status-header">
      
      <div 
        class="player-card p1" 
        :class="{ active: activePlayer === 1, 'is-set': player1.date }"
        @click="selectPlayer(1)"
      >
        <div class="card-icon">🔴</div>
        <div class="card-info">
          <span class="role-label">先攻 (RED)</span>
          <div class="main-display">
            <span class="pref-text">{{ player1.pref ? player1.pref.name : '場所未選択' }}</span>
            <div class="date-badge" :class="{ 'has-date': player1.date }">
              {{ player1.label }}
            </div>
          </div>
        </div>
      </div>

      <div class="vs-logo">VS</div>

      <div 
        class="player-card p2" 
        :class="{ active: activePlayer === 2, 'is-set': player2.date }"
        @click="selectPlayer(2)"
      >
        <div class="card-icon">🔵</div>
        <div class="card-info">
          <span class="role-label">後攻 (BLUE)</span>
          <div class="main-display">
            <span class="pref-text">{{ player2.pref ? player2.pref.name : '場所未選択' }}</span>
            <div class="date-badge" :class="{ 'has-date': player2.date }">
              {{ player2.label }}
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="battle-header">
      <h1>⚔️ 雪合戦バトルモード</h1>
    </div>

    <main>
      <p class="instruction-text" v-html="instructionText"></p>
      <JapanMap3D @select="handleMapSelect" />
    </main>

    <div class="footer-action">
      <p v-if="errorMessage" class="error-msg-bar">{{ errorMessage }}</p>
      <button 
        class="start-btn" 
        :disabled="!player1.date || !player2.date"
        @click="startBattle"
      >
        BATTLE START !!
      </button>
    </div>

    <div v-if="showModal" class="modal-overlay" @click.self="closeModal">
      <div class="modal-content">
        <button class="close-btn" @click="closeModal">×</button>
        <div class="modal-header">
          <h2>📍 {{ currentSelectingPref?.name }}</h2>
          <p class="guide-text">
            <span v-if="activePlayer === 1" style="color:#d32f2f;">🔴 先攻</span>
            <span v-else style="color:#1976d2;">🔵 後攻</span>
            の日付を決めてください
          </p>
        </div>
        <div class="calendar-wrapper">
          <SmartCalendar 
            v-model="calendarDate" 
            :prefId="currentSelectingPref?.id" 
            @search="handleDateSelect" 
          />
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* =======================================
   バックボタン
   ======================================= */
.back-btn {
  position: fixed; top: 20px; left: 20px; z-index: 100;
  background: rgba(255, 255, 255, 0.2); border: 1px solid rgba(255, 255, 255, 0.5); color: #fff;
  padding: 15px 30px; border-radius: 30px; font-weight: bold; cursor: pointer;
  backdrop-filter: blur(5px); transition: all 0.3s; box-shadow: 0 4px 6px rgba(0,0,0,0.1); font-size: 1.2rem;
}
.back-btn:hover { background: rgba(255, 255, 255, 0.4); transform: translateX(-3px); }

/* =======================================
   レイアウト
   ======================================= */
.battle-container {
  text-align: center; padding-top: 130px; padding-bottom: 100px;
  min-height: 100vh; position: relative; z-index: 10; color: #fff; box-sizing: border-box;
}
.battle-header { margin-top: 20px; margin-bottom: 10px; position: relative; z-index: 5; }
.battle-header h1 { margin: 0; color: #ffcc80; font-size: 2rem; text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.7); letter-spacing: 2px; }
main { position: relative; z-index: 5; }

/* =======================================
   ステータスパネル (日付表示)
   ======================================= */
.status-header {
  position: fixed; top: 0; left: 0; width: 100%; padding: 10px 2%;
  background: rgba(255, 255, 255, 0.25); backdrop-filter: blur(12px); -webkit-backdrop-filter: blur(12px);
  z-index: 50; display: flex; justify-content: center; align-items: center; box-shadow: 0 4px 15px rgba(0,0,0,0.2);
}

.player-card {
  flex: 1; max-width: 320px; /* 幅を広めに */
  background: rgba(255, 255, 255, 0.9); border-radius: 16px; padding: 10px 15px;
  display: flex; align-items: center; cursor: pointer; transition: all 0.3s;
  color: #333; border: 3px solid transparent; box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}
.player-card.is-set { background: #fff; border-color: #ddd; }
.player-card.p1.active { border-color: #ff5252; background: #ffebee; transform: scale(1.05); box-shadow: 0 0 20px rgba(255, 82, 82, 0.6); z-index: 10; }
.player-card.p2.active { border-color: #448aff; background: #e3f2fd; transform: scale(1.05); box-shadow: 0 0 20px rgba(68, 138, 255, 0.6); z-index: 10; }

.card-icon { font-size: 2rem; margin-right: 12px; }
.card-info { text-align: left; flex: 1; }
.role-label { font-size: 0.7rem; color: #666; font-weight: bold; display: block; text-transform: uppercase; margin-bottom: 2px; }

.main-display { display: flex; flex-direction: column; line-height: 1.2; }
.pref-text { font-weight: 900; font-size: 1.2rem; color: #222; }

/* 日付バッジ (未選択時) */
.date-badge {
  font-size: 0.85rem; color: #999; margin-top: 4px;
  background: rgba(0,0,0,0.05); padding: 2px 8px; border-radius: 10px;
  display: inline-block; align-self: flex-start;
}
/* 日付バッジ (選択済み) */
.date-badge.has-date {
  font-weight: bold; font-size: 1.1rem; /* 大きくする */
  background: transparent; padding: 0; margin-top: 2px;
}
.p1 .date-badge.has-date { color: #d32f2f; }
.p2 .date-badge.has-date { color: #1565c0; }

.vs-logo { font-size: 1.8rem; font-weight: 900; font-style: italic; margin: 0 15px; color: #fff; text-shadow: 0 2px 5px rgba(0,0,0,0.5); }

/* =======================================
   メインエリア
   ======================================= */
.instruction-text {
  font-size: 1.2rem; margin-bottom: 15px; text-shadow: 0 2px 4px rgba(0,0,0,0.6);
  line-height: 1.5; background: rgba(0,0,0,0.4); display: inline-block; padding: 8px 25px; border-radius: 30px;
}
:deep(.instruction-text b) { font-size: 1.4rem; color: #fffde7; text-decoration: underline; }
:deep(.instruction-text small) { font-size: 0.9em; opacity: 0.9; margin: 0 5px; }

/* =======================================
   フッター
   ======================================= */
.footer-action { position: fixed; bottom: 0; left: 0; width: 100%; padding: 20px; background: rgba(0,0,0,0.5); z-index: 50; display: flex; flex-direction: column; align-items: center; }
.start-btn { width: 90%; max-width: 400px; padding: 16px; border: none; border-radius: 50px; background: linear-gradient(45deg, #ff5252, #d32f2f, #b71c1c); background-size: 200% auto; color: white; font-weight: bold; font-size: 1.4rem; cursor: pointer; box-shadow: 0 4px 15px rgba(0,0,0,0.3); transition: transform 0.2s, box-shadow 0.2s; text-shadow: 0 1px 2px rgba(0,0,0,0.3); }
.start-btn:disabled { background: #9e9e9e; cursor: not-allowed; box-shadow: none; }
.start-btn:hover:not(:disabled) { transform: translateY(-3px); box-shadow: 0 8px 25px rgba(255, 82, 82, 0.6); }
.error-msg-bar { color: #ff8a80; font-weight: bold; margin-bottom: 10px; background: rgba(0,0,0,0.6); padding: 5px 15px; border-radius: 20px; }

/* =======================================
   モーダル
   ======================================= */
.modal-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0, 0, 0, 0.7); display: flex; justify-content: center; align-items: center; z-index: 3000; animation: fadeIn 0.3s; }
.modal-content { background: #fff; color: #333; padding: 25px; border-radius: 20px; width: 95%; max-width: 500px; position: relative; animation: popUp 0.4s; }
.close-btn { position: absolute; top: 15px; right: 20px; background: none; border: none; font-size: 2rem; color: #555; cursor: pointer; }
.modal-header { margin-bottom: 20px; }
.modal-header h2 { margin: 0; color: #1565c0; }
.guide-text { margin-top: 5px; font-weight: bold; }
.calendar-wrapper { margin-bottom: 10px; }

@keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
@keyframes popUp { from { transform: scale(0.9); opacity: 0; } to { transform: scale(1); opacity: 1; } }
</style>

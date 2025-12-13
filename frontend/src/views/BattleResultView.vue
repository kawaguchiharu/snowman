<script setup>
import { ref, onMounted, computed } from 'vue'
import { useRoute, useRouter } from 'vue-router'

// 共通コンポーネント
import SnowEffect from '../components/Snoweffect.vue'
import OceanBackground from '../components/oceanbackground.vue'
import snowmanImg from '/public/snowman.svg'

const route = useRoute()
const router = useRouter()

// パラメータ取得
const p1Data = computed(() => ({
  name: route.query.p1_name || 'Player 1',
  label: route.query.p1_label || '日付不明',
  dateRaw: route.query.p1_date || '2024-01-01',
  id: Number(route.query.p1_id) || 1
}))

const p2Data = computed(() => ({
  name: route.query.p2_name || 'Player 2',
  label: route.query.p2_label || '日付不明',
  dateRaw: route.query.p2_date || '2024-01-01',
  id: Number(route.query.p2_id) || 13
}))

// バトル状態管理
const battleState = ref('ready') 
const winner = ref(null)

// HP (スケーリング後の整数値)
const p1Hp = ref(0)
const p2Hp = ref(0)
const p1MaxHp = ref(1) 
const p2MaxHp = ref(1)

// 実体積（データ保持用）
const p1RealVolume = ref(0)
const p2RealVolume = ref(0)

// アニメーション管理
const p1Action = ref('idle')
const p2Action = ref('idle')
const logMessage = ref('データを取得中...') 

// ★決定した武器情報
const p1Weapon = ref(null)
const p2Weapon = ref(null)

// 武器名ポップアップ用 (攻撃時のみ表示)
const currentWeaponNameP1 = ref('')
const currentWeaponNameP2 = ref('')

// ==========================================
// 武器リスト (攻撃力は参照されませんが定義は残します)
// ==========================================
const weaponList = [
  { name: '人参の鼻', power: 30, icon: '🥕' },
  { name: '素手', power: 150, icon: '✊' },
  { name: '雪玉', power: 450, icon: '❄️' },
  { name: 'バケツ', power: 600, icon: '🪣' },
  { name: 'つらら', power: 900, icon: '🗡️' },
  { name: 'スコップ', power: 1500, icon: '🥄' },
  { name: 'スノーダンプ', power: 3000, icon: '🚜' },
  { name: '除雪車', power: 7500, icon: '🚛' },
  { name: '雪崩', power: 15000, icon: '🏔️' },
  { name: '氷河期', power: 30000, icon: '🥶' } 
]

const detectType = (label) => {
  if (label.includes('日')) return 'day'
  if (label.includes('月')) return 'month'
  return 'year'
}

// API連携
const fetchSnowData = async (playerData) => {
  try {
    const type = detectType(playerData.label)

    const res = await fetch('http://localhost:8000/calculate', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        prefecture_id: playerData.id,
        target_date: playerData.dateRaw,
        period_type: type
      })
    })

    if (res.ok) {
      const data = await res.json()
      const rawVolume = data.volume_m3 

      let divisor = 100000 
      if (type === 'month') divisor = 1500000
      else if (type === 'year') divisor = 5000000

      const battleHp = Math.floor(rawVolume / divisor)

      return {
        rawVolume: Math.floor(rawVolume), 
        battleHp: Math.max(50, battleHp) 
      }

    } else {
      console.error('API Error:', await res.text())
      return { rawVolume: 0, battleHp: 100 }
    }
  } catch (e) {
    console.error('Connection Error:', e)
    return { rawVolume: 0, battleHp: 100 }
  }
}

const sleep = (ms) => new Promise(resolve => setTimeout(resolve, ms))

const startBattleSequence = async () => {
  const [res1, res2] = await Promise.all([
    fetchSnowData(p1Data.value),
    fetchSnowData(p2Data.value)
  ])

  // HP設定
  p1RealVolume.value = res1.rawVolume
  p1MaxHp.value = res1.battleHp
  p1Hp.value = res1.battleHp
  
  p2RealVolume.value = res2.rawVolume
  p2MaxHp.value = res2.battleHp
  p2Hp.value = res2.battleHp

  await sleep(500)

  // HPチェック
  if (p1Hp.value <= 10 && p2Hp.value <= 10) {
    logMessage.value = '雪が足りず勝負になりません...'
    finishBattle()
    return
  }

  // ★ここで武器を決定して固定！
  p1Weapon.value = weaponList[Math.floor(Math.random() * weaponList.length)]
  p2Weapon.value = weaponList[Math.floor(Math.random() * weaponList.length)]
  
  logMessage.value = `1P:${p1Weapon.value.name} vs 2P:${p2Weapon.value.name}`
  await sleep(1500) // 武器決定のログを見せる時間

  battleState.value = 'fighting'
  logMessage.value = 'BATTLE START!'

  while (p1Hp.value > 0 && p2Hp.value > 0) {
    const isP1Turn = Math.random() > 0.5
    if (isP1Turn) {
      await performAttack(1)
    } else {
      await performAttack(2)
    }
    await sleep(1000)
  }
  finishBattle()
}

// 攻撃ロジック (ダメージを1000に固定)
const performAttack = async (attacker) => {
  
  // 決定済みの武器を取得
  const weapon = attacker === 1 ? p1Weapon.value : p2Weapon.value

  // モーションと名前ポップアップ表示
  if (attacker === 1) {
    p1Action.value = 'attack'
    currentWeaponNameP1.value = weapon.name
  } else {
    p2Action.value = 'attack'
    currentWeaponNameP2.value = weapon.name
  }

  await sleep(300) 

  // ★ダメージ計算を固定値 1000 に変更
  const damage = 1000

  if (attacker === 1) {
    p2Action.value = 'damage'
    p2Hp.value = Math.max(0, p2Hp.value - damage)
    logMessage.value = `${p1Data.value.name}の攻撃！ ${damage}ダメージ`
  } else {
    p1Action.value = 'damage'
    p1Hp.value = Math.max(0, p1Hp.value - damage)
    logMessage.value = `${p2Data.value.name}の攻撃！ ${damage}ダメージ`
  }

  await sleep(600)
  p1Action.value = 'idle'
  p2Action.value = 'idle'
  currentWeaponNameP1.value = ''
  currentWeaponNameP2.value = ''
}

const finishBattle = () => {
  battleState.value = 'finished'
  if (p1Hp.value <= 0 && p2Hp.value <= 0) {
    winner.value = 0 // 引き分け
    logMessage.value = '相打ち（引き分け）'
  } else {
    winner.value = p1Hp.value > 0 ? 1 : 2
    logMessage.value = winner.value === 1 ? `${p1Data.value.name}の勝利！` : `${p2Data.value.name}の勝利！`
  }
}

// スケール計算ロジック
const getScale = (battleHp) => {
  if (battleHp <= 0) return 0.5
  const logVal = Math.log10(battleHp)
  // スケールを控えめにし、最大サイズを抑える (最大 1.5倍 程度)
  const scale = 1.0 + (Math.max(0, logVal - 3) * 0.05) 
  return Math.min(1.5, Math.max(1.0, scale)) 
}

const p1HpPercent = computed(() => Math.max(0, p1Hp.value / p1MaxHp.value * 100))
const p2HpPercent = computed(() => Math.max(0, p2Hp.value / p2MaxHp.value * 100))

onMounted(() => {
  startBattleSequence()
})

const goBackHome = () => router.push('/home')
const goBackBattle = () => router.push('/battle')
const goTop = () => router.push('/')
</script>

<template>
  <SnowEffect />
  
  <div class="battle-result-container">
    <OceanBackground />

    <div class="header-status-bar">
      
      <div class="player-status-block p1-block">
        <div class="info-text">
          <h2 class="pref-name">{{ p1Data.name }}</h2>
          <p class="date-text">{{ p1Data.label }}</p>
          <span class="hp-value-label">HP: {{ p1Hp.toLocaleString() }}</span>
        </div>
        <div class="hp-bar-container">
          <div class="hp-bar-fill red" :style="{ width: p1HpPercent + '%' }"></div>
        </div>
      </div>

      <div class="ko-logo">
        <div class="ko-text">
          <span v-if="battleState === 'finished'">K.O.</span>
          <span v-else-if="battleState === 'fighting'">FIGHT!</span>
          <span v-else>VS</span>
        </div>
      </div>

      <div class="player-status-block p2-block">
        <div class="info-text">
          <h2 class="pref-name">{{ p2Data.name }}</h2>
          <p class="date-text">{{ p2Data.label }}</p>
          <span class="hp-value-label">HP: {{ p2Hp.toLocaleString() }}</span>
        </div>
        <div class="hp-bar-container">
          <div class="hp-bar-fill blue" :style="{ width: p2HpPercent + '%' }"></div>
        </div>
      </div>
    </div>

    <div class="avatar-stage">
        
      <div class="player-avatar-area p1" :class="{ 'loser-shake': battleState === 'finished' && winner === 2 }">
        <div v-if="p1Action === 'attack'" class="weapon-popup red-pop">
          {{ currentWeaponNameP1 }}
        </div>

        <img 
          :src="snowmanImg" 
          class="snowman-img red-tint" 
          :class="p1Action"
          :style="{ transform: `scale(${getScale(p1MaxHp)})` }" 
        />
        
        <div v-if="p1Weapon" class="equipped-weapon p1-weapon-icon">
          {{ p1Weapon.icon }}
        </div>
      </div>

      <div class="player-avatar-area p2" :class="{ 'loser-shake': battleState === 'finished' && winner === 1 }">
        <div v-if="p2Action === 'attack'" class="weapon-popup blue-pop">
          {{ currentWeaponNameP2 }}
        </div>

        <img 
          :src="snowmanImg" 
          class="snowman-img blue-tint flipped" 
          :class="p2Action" 
          :style="{ transform: `scaleX(-1) scale(${getScale(p2MaxHp)})` }"
        />

        <div v-if="p2Weapon" class="equipped-weapon p2-weapon-icon">
          {{ p2Weapon.icon }}
        </div>
      </div>
    </div>


    <div class="log-area-container">
      <div class="battle-log-box">
        <p class="log-message">▶ {{ logMessage }}</p>
        <div v-if="battleState === 'finished'" class="final-result">
           <span v-if="winner === 1" class="winner-text red-text">WINNER: {{ p1Data.name }}</span>
           <span v-else-if="winner === 2" class="winner-text blue-text">WINNER: {{ p2Data.name }}</span>
           <span v-else class="winner-text">DRAW</span>
        </div>
      </div>
    </div>


    <div v-if="battleState === 'finished'" class="action-footer">
      <button @click="goBackBattle" class="retry-btn">⚔️ 再戦する</button>
      <button @click="goBackHome" class="home-btn">🗺️ 地図へ</button>
      <button @click="goTop" class="top-btn">🏠 TOP</button>
    </div>

  </div>
</template>

<style scoped>
/* =======================================
   レイアウト
   ======================================= */
.battle-result-container {
  min-height: 100vh;
  position: relative;
  display: flex; flex-direction: column; justify-content: flex-start; 
  align-items: center;
  z-index: 10; overflow: hidden; color: #fff;
  padding-top: 100px; /* ステータスバーの高さ分を空ける */
  padding-bottom: 100px; /* ★フッターとログエリアが隠れない最低限のパディングに削減 */
}

/* =======================================
   HP & ステータスバー (上部)
   ======================================= */
.header-status-bar {
  position: fixed; top: 0; left: 0; width: 100%; height: 100px;
  background: rgba(0, 0, 0, 0.4); backdrop-filter: blur(5px);
  display: flex; justify-content: center; align-items: center;
  padding: 10px 20px; box-sizing: border-box;
  z-index: 100;
  box-shadow: 0 5px 15px rgba(0,0,0,0.5);
}

.player-status-block {
  flex: 1; max-width: 400px; height: 100%;
  display: flex; flex-direction: column; justify-content: center;
}
.p1-block { align-items: flex-start; }
.p2-block { align-items: flex-end; }

.info-text {
  width: 100%;
  padding: 0 10px;
  margin-bottom: 5px;
}
.p1-block .info-text { text-align: left; }
.p2-block .info-text { text-align: right; }

.pref-name { 
  margin: 0; font-size: 1.5rem; font-weight: 900; 
  text-shadow: 1px 1px 3px #000; letter-spacing: 1px;
}
.date-text { 
  margin: 0; font-size: 0.8rem; font-weight: bold; 
  color: #ffeb3b; text-shadow: 1px 1px 2px #000;
}
.hp-value-label {
  font-size: 0.9rem; font-weight: bold;
  color: #fff; text-shadow: 1px 1px 2px #000;
}

/* HPバー */
.hp-bar-container {
  width: 100%; height: 25px;
  background: #444; border-radius: 5px; overflow: hidden;
  border: 2px solid #fff; box-shadow: 0 2px 5px rgba(0,0,0,0.5);
}
.hp-bar-fill { height: 100%; transition: width 0.3s ease-out; }
.red { background: linear-gradient(to right, #ff5252, #d50000); }
.blue { background: linear-gradient(to left, #448aff, #2962ff); }
.p1-block .hp-bar-container { border-right: none; }
.p2-block .hp-bar-container { border-left: none; }

/* KO ロゴ */
.ko-logo {
  position: relative; width: 80px; height: 80px; margin: 0 10px;
  display: flex; justify-content: center; align-items: center;
  background: #ffcc00; border-radius: 50%;
  border: 5px solid #fff; box-shadow: 0 0 15px rgba(255, 255, 0, 0.8);
  font-weight: 900; color: #d50000; font-style: italic;
  transform: rotate(-10deg); z-index: 110;
}
.ko-text { font-size: 1.8rem; text-shadow: 1px 1px 2px #000; }

/* =======================================
   アバターエリア (中央)
   ======================================= */
.avatar-stage {
  flex: 1; 
  width: 100%;
  display: flex; justify-content: space-around;
  align-items: flex-end; /* 雪だるまを底に配置 */
  padding-bottom: 0px; /* ★パディングをさらに削減し、雪だるまを上へ */
  box-sizing: border-box;
  overflow: hidden; /* ★はみ出し対策 */
}

.player-avatar-area {
  position: relative;
  width: 45%; 
  max-width: 300px; /* ★ 300pxに戻して、画面に収まりやすくする */
  height: auto;
  display: flex; justify-content: center; align-items: flex-end;
  transition: filter 0.5s, transform 0.5s;
}

.loser-shake {
  animation: shake-loser 1s forwards infinite;
  opacity: 0.5;
}

.snowman-img {
  width: 100%; 
  height: auto; 
  object-fit: contain;
  animation: idle-bounce 1.5s infinite ease-in-out; 
  transform-origin: center bottom;
  transition: transform 0.3s, filter 0.3s; 
  position: relative; z-index: 20;
}

/* 画像の色付けと影 */
.red-tint { filter: drop-shadow(0 0 10px rgba(255, 0, 0, 0.5)); }
.blue-tint { filter: drop-shadow(0 0 10px rgba(0, 80, 255, 0.5)); }
.flipped { transform: scaleX(-1); }
.flipped.idle { animation: idle-bounce-flipped 1.5s infinite ease-in-out; }

/* 攻撃・ダメージアニメーション */
.p1 .snowman-img.attack { animation: lunge-right 0.3s forwards; }
.p2 .snowman-img.attack { animation: lunge-left-flipped 0.3s forwards; }
.damage {
  animation: shake-damage 0.4s forwards !important;
  filter: sepia(100%) saturate(1000%) hue-rotate(-50deg) drop-shadow(0 0 10px red) !important;
}

/* 武器名ポップアップ */
.weapon-popup {
  position: absolute; top: -50px; left: 50%; transform: translateX(-50%);
  font-weight: bold; font-size: 1.2rem; color: #fff; white-space: nowrap;
  padding: 5px 15px; border-radius: 15px;
  animation: popUpFade 0.5s ease-out forwards;
  z-index: 40; border: 3px solid #fff;
}
.red-pop { background: #ff5252; box-shadow: 0 4px 10px rgba(255, 0, 0, 0.6); }
.blue-pop { background: #448aff; box-shadow: 0 4px 10px rgba(0, 0, 255, 0.6); }

/* 装備武器アイコン (足元に配置) */
.equipped-weapon {
  position: absolute;
  font-size: 3rem;
  z-index: 30;
  filter: drop-shadow(0 2px 4px rgba(0,0,0,0.5));
  animation: floatWeapon 2s ease-in-out infinite;
  bottom: 10px;
}
.p1-weapon-icon { right: 10%; } 
.p2-weapon-icon { left: 10%; } 

/* =======================================
   バトルログ
   ======================================= */
.log-area-container {
  position: absolute; bottom: 80px; /* フッターの上に配置 */
  width: 90%; max-width: 600px;
  z-index: 50;
  display: flex; justify-content: center;
}

.battle-log-box {
  background: rgba(0, 0, 0, 0.7);
  border: 3px solid #ffcc00;
  padding: 15px 20px;
  border-radius: 10px;
  box-shadow: 0 4px 15px rgba(0,0,0,0.5);
  min-height: 50px;
  text-align: left;
  animation: fadeIn 0.5s;
}

.log-message {
  margin: 0;
  font-weight: bold; font-size: 1.1rem; color: #fff;
  text-shadow: 0 1px 2px #000;
}

.final-result { margin-top: 10px; border-top: 1px dashed rgba(255,255,255,0.3); padding-top: 8px; }
.winner-text { font-size: 1.5rem; font-weight: 900; letter-spacing: 1px; }
.red-text { color: #ff5252; }
.blue-text { color: #448aff; }

/* =======================================
   フッター (ボタン)
   ======================================= */
.action-footer { 
  position: fixed; bottom: 0; left: 0; width: 100%;
  padding: 15px; 
  background: rgba(0, 0, 0, 0.5); 
  display: flex; gap: 10px; justify-content: center; 
  z-index: 60; animation: fadeIn 1s; 
  box-shadow: 0 -5px 15px rgba(0,0,0,0.3);
}
button { 
  padding: 10px 20px; border: none; border-radius: 30px; font-weight: bold; cursor: pointer; 
  box-shadow: 0 4px 10px rgba(0,0,0,0.3); transition: transform 0.2s; 
}
button:hover { transform: translateY(-3px); }
.retry-btn { background: #ff9800; color: white; }
.home-btn { background: #03a9f4; color: white; }
.top-btn { background: #fff; color: #555; }


/* Keyframes */
@keyframes idle-bounce { 0%, 100% { transform: scale(var(--scale, 1.0)) translateY(0); } 50% { transform: scale(var(--scale, 1.0)) translateY(-5px); } }
@keyframes idle-bounce-flipped { 0%, 100% { transform: scaleX(-1) scale(var(--scale, 1.0)) translateY(0); } 50% { transform: scaleX(-1) scale(var(--scale, 1.0)) translateY(-5px); } }
@keyframes lunge-right { 0% { transform: scale(var(--scale, 1.0)) translateX(0); } 50% { transform: scale(var(--scale, 1.0)) translateX(80px) rotate(5deg); } 100% { transform: scale(var(--scale, 1.0)) translateX(0); } }
@keyframes lunge-left-flipped { 0% { transform: scaleX(-1) scale(var(--scale, 1.0)) translateX(0); } 50% { transform: scaleX(-1) scale(var(--scale, 1.0)) translateX(80px) rotate(-5deg); } 100% { transform: scaleX(-1) scale(var(--scale, 1.0)) translateX(0); } }
@keyframes shake-damage { 0% { transform: translate(0); } 25% { transform: translate(-10px, 0); } 50% { transform: translate(10px, 0); } 75% { transform: translate(-10px, 0); } 100% { transform: translate(0); } }
@keyframes shake-loser { 0%, 100% { transform: rotate(0deg); } 25% { transform: rotate(-1deg); } 75% { transform: rotate(1deg); } }
@keyframes popUpFade { 0% { opacity: 0; transform: translateX(-50%) translateY(10px); } 100% { opacity: 1; transform: translateX(-50%) translateY(0); } }
@keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
@keyframes floatWeapon {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-5px); }
}

.snowman-img {
    /* getScaleの計算結果をCSS変数として適用 */
    --scale: v-bind('getScale(p1MaxHp)');
}
.p2 .snowman-img {
    /* p2用にもCSS変数を適用 */
    --scale: v-bind('getScale(p2MaxHp)');
}
</style>
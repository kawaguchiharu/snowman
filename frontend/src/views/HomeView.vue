<script setup>

import { ref } from 'vue'

import { useRouter } from 'vue-router'



// 外部コンポーネントのインポート

import JapanMap3D from '../components/japanmap_3D.vue'

import SmartCalendar from '../components/calendar.vue'

import SnowEffect from '../components/Snoweffect.vue'

import OceanBackground from '../components/oceanbackground.vue' 



const router = useRouter() // ページ移動用



const selectedPref = ref(null)

const showModal = ref(false)

// 初期日付を本日以降に変更したい場合は適宜調整

const targetDate = ref(new Date('2024-01-01'))



// 地図クリック

const handleMapSelect = (pin) => {

selectedPref.value = pin

showModal.value = true

}



const closeModal = () => showModal.value = false



// 日付フォーマット

const formatDate = (date) => {

if (!date) return ''

const d = new Date(date)

return `${d.getFullYear()}-${String(d.getMonth() + 1).padStart(2, '0')}-${String(d.getDate()).padStart(2, '0')}`

}



// ページ移動

const handleSearch = (type) => {

router.push({

path: '/result',

query: {

pref: selectedPref.value.id,

prefName: selectedPref.value.name,

date: formatDate(targetDate.value),

type: type

}

})

}

</script>



<template>

<SnowEffect />



<div class="home-container">

<OceanBackground />


<header>

<h1>⛄ 雪だるまシミュレーター</h1>

<p>地図のピンをクリックしてね！</p>

</header>



<main>

<JapanMap3D @select="handleMapSelect" />

</main>



<div v-if="showModal" class="modal-overlay" @click.self="closeModal">

<div class="modal-content">

<button class="close-btn" @click="closeModal">×</button>

<div class="modal-header">

<h2>📍 {{ selectedPref.name }}</h2>

</div>

<p class="guide-text">クリックすると結果画面へ移動します👇</p>



<div class="calendar-wrapper">

<SmartCalendar v-model="targetDate" @search="handleSearch" />

</div>

</div>

</div>

</div>

</template>



<style scoped>

/* =======================================

ホーム画面全体のスタイル

======================================= */

.home-container {

text-align: center;

padding-bottom: 50px;

/* 背景色/グラデーションは oceanbackground.vue に移動 */

min-height: 100vh;

position: relative; /* 子要素 (OceanBackground, Header) の位置決め基準 */

z-index: 10;

}



/* =======================================

★ 修正点: main (日本地図を含む) の z-index 設定

OceanBackground (z-index: 1) より手前に配置し、地図が表示されるようにする。

======================================= */

main {

position: relative; /* z-indexを有効にするため */

z-index: 5; /* 海 (1) より高く、ヘッダー (20) より低い値 */

}



/* =======================================

ヘッダーのスタイル

======================================= */

header {

margin-bottom: 20px;

margin-top: 20px;

color: #e3f2fd;

position: relative;

z-index: 20;

}



h1 {

color: #bbdefb;

text-shadow: 1px 1px 3px rgba(0, 0, 0, 0.5);

}



/* =======================================

モーダルのスタイル

======================================= */

.modal-overlay {

position: fixed; top: 0; left: 0; width: 100%; height: 100%;

background: rgba(0, 0, 0, 0.7);

display: flex; justify-content: center; align-items: center;

z-index: 3000;

animation: fadeIn 0.3s;

}



.modal-content {

background: #4fc3f7;

padding: 25px;

border-radius: 20px;

width: 95%; max-width: 500px;

position: relative;

animation: popUp 0.4s;

box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);

}



.close-btn {

position: absolute; top: 15px; right: 20px;

background: none; border: none;

font-size: 2rem;

color: #1a237e;

cursor: pointer;

}



.modal-header { text-align: center; margin-bottom: 15px; }



.modal-header h2 {

color: #1a237e;

}



.guide-text {

font-size: 0.9rem;

color: #1a237e;

margin-bottom: 15px;

}



.calendar-wrapper { display: flex; justify-content: center; margin-bottom: 20px; }



@keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }

@keyframes popUp { from { transform: scale(0.9); opacity: 0; } to { transform: scale(1); opacity: 1; } }

</style>
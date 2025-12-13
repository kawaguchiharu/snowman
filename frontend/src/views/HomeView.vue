<script setup>

import { ref } from 'vue'

import { useRouter } from 'vue-router'

import JapanMap3D from '../components/japanmap_3D.vue'

import SmartCalendar from '../components/calendar.vue'



const router = useRouter() // ページ移動用



const selectedPref = ref(null)

const showModal = ref(false)

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



// ★ここが重要: 計算せずにページ移動！

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

<div class="home-container">

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

.home-container { text-align: center; padding-bottom: 50px; }

h1 { color: #0288d1; }

header { margin-bottom: 20px; margin-top: 20px; }

.modal-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0, 0, 0, 0.6); display: flex; justify-content: center; align-items: center; z-index: 999; animation: fadeIn 0.3s; }

.modal-content { background: rgb(66, 137, 243); padding: 25px; border-radius: 20px; width: 95%; max-width: 500px; position: relative; animation: popUp 0.4s; }

.close-btn { position: absolute; top: 15px; right: 20px; background: none; border: none; font-size: 2rem; color: #aaa; cursor: pointer; }

.modal-header { text-align: center; margin-bottom: 15px; }

.guide-text { font-size: 0.9rem; color: #666; margin-bottom: 15px; }

.calendar-wrapper { display: flex; justify-content: center; margin-bottom: 20px; }

@keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }

@keyframes popUp { from { transform: scale(0.9); opacity: 0; } to { transform: scale(1); opacity: 1; } }

</style>
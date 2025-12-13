<template>
  <div class="snow-area">
    <div class="snow_layer snow_layer1"></div>
    <div class="snow_layer snow_layer4"></div> <div class="snow_layer snow_layer2"></div>
    <div class="snow_layer snow_layer5"></div> <div class="snow_layer snow_layer3"></div>
    <div class="snow_layer snow_layer6"></div> </div>
</template>

<script setup>
// このエフェクトは純粋にCSSで動作するため、JavaScriptは不要です
</script>

<style scoped>
/* =======================================================
   1. コンテナ設定 (画面全体に固定配置)
   ======================================================= */
.snow-area {
  position: fixed;
  top: 0; left: 0;
  width: 100vw; height: 100vh;
  /* 全てのUIの上に表示 */
  z-index: 9000;
  pointer-events: none;
  overflow: hidden;
}

/* =======================================================
   2. 雪の基本的なスタイル (r=5 で粒の大きさを確保)
   ======================================================= */
.snow_layer {
  /* SVG内の円の半径を r="5" に設定し、粒を大きくしています */
  background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="100" height="100"><circle cx="50" cy="50" r="3" fill="white" /></svg>');
  width: 100%; height: 100%;
  background-position: 0% 0%;
  position: absolute;
  top: 0;
  left: 0;
}

/* =======================================================
   3. レイヤーグループ定義
   ======================================================= */

/* --- グループ A: 標準の速い雪 (粒は中程度、左揺れ) --- */
.snow_layer1, .snow_layer4 {
  background-size: 45px 45px; /* 粒の密度 */
  opacity: 0.8;
  animation: snow_fall_standard 20s linear infinite;
}
.snow_layer4 {
  animation-delay: -10s; /* 開始タイミングをずらす (20sの半分) */
}

/* --- グループ B: 遅い雪（遠景・粒が小さい、右揺れ） --- */
.snow_layer2, .snow_layer5 {
  background-size: 70px 70px; /* 粒を小さく、密度を低く */
  opacity: 0.4;
  animation: snow_fall_slow 40s linear infinite;
}
.snow_layer5 {
  animation-delay: -20s; /* 開始タイミングをずらす (40sの半分) */
}

/* --- グループ C: 強い揺らぎの雪（近景・粒が大きい、強い左揺れ） --- */
.snow_layer3, .snow_layer6 {
  background-size: 30px 30px; /* 粒を大きく、密度を高めに */
  opacity: 0.9;
  animation: snow_fall_wiggle 15s linear infinite; /* アニメーション時間を速く */
}
.snow_layer6 {
  animation-delay: -7.5s; /* 開始タイミングをずらす (15sの半分) */
}


/* =======================================================
   4. アニメーション定義 (3種類)
   ======================================================= */

/* パターン A: 標準的な左揺れ */
@keyframes snow_fall_standard {
  0% { background-position: 0% 0%; }
  100% { background-position: -10% 100%; } 
}

/* パターン B: 遅い右揺れ (風の方向が異なるように見せる) */
@keyframes snow_fall_slow {
  0% { background-position: 0% 0%; }
  100% { background-position: 5% 100%; } 
}

/* パターン C: 強い左揺れ */
@keyframes snow_fall_wiggle {
  0% { background-position: 0% 0%; }
  100% { background-position: -20% 100%; } 
}
</style>
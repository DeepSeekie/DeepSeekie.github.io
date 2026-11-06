<template>
  <div id="splash" :class="{ 'is-leaving': leaving }" @pointerdown="onTap">
    <div class="splash-frame"><i></i><i></i><i></i><i></i></div>
    <div class="splash-stage" :class="'is-' + phase">
      <svg class="rings" viewBox="0 0 300 300">
        <g class="ring ring-outer"><circle cx="150" cy="150" r="138"/></g>
        <g class="ring ring-mid"><circle cx="150" cy="150" r="112"/></g>
        <g class="ring ring-inner"><circle cx="150" cy="150" r="86"/></g>
        <g class="ring ring-core"><circle cx="150" cy="150" r="58"/></g>
        <g class="stream stream-1"><circle cx="150" cy="150" r="125" stroke-dasharray="22 600"/></g>
        <g class="stream stream-2"><circle cx="150" cy="150" r="99" stroke-dasharray="36 600"/></g>
        <g class="stream stream-3"><circle cx="150" cy="150" r="71" stroke-dasharray="14 600"/></g>
        <circle class="ring-dot" cx="150" cy="150" r="2.4"/>
      </svg>
      <div class="ring-data">
        <div class="data-item" style="top: 12%; left: 6%;"><span class="data-k">NEURAL SYNC</span><span class="data-v is-blue">{{ syncDisplay }}<span class="unit">%</span></span></div>
        <div class="data-item" style="top: 12%; right: 6%; text-align: right;"><span class="data-k">CLEARANCE</span><span class="data-v">LEVEL A</span></div>
      </div>
      <div class="verify-card" :class="{ 'is-open': phase === 'verify' || phase === 'confirm' }">
        <div class="verify-head">BIOMETRIC CONFIRMATION</div>
        <div class="verify-body">
          <div class="verify-id"><span>ID: DOCTOR</span><span class="tag">PRTS</span></div>
          <div class="verify-bar"><i :style="{ width: syncPct + '%' }"></i></div>
          <div class="verify-pct"><span>NEURAL LINK</span><span class="num">{{ syncPct }}%</span></div>
        </div>
        <div class="verify-action">
          <span class="led"></span>
          <span v-if="phase === 'verify'">AWAITING TAP</span><span v-else>SYNCING…</span>
        </div>
      </div>
      <div class="welcome" :class="{ 'is-open': phase === 'welcome' }">
        <div class="welcome-en">ACCESS GRANTED</div>
        <div class="welcome-cn">博士。神经链接已同步。<br><span class="accent">PRTS 为您开放全部权限。</span></div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const emit = defineEmits(['done']);
const leaving = ref(false);
const phase = ref('boot');
const syncPct = ref(0);
const syncDisplay = ref('00');
let dataTimer = null;

const onTap = () => {
  if (phase.value !== 'verify') return;
  phase.value = 'confirm';
  if (dataTimer) clearInterval(dataTimer);

  const from = 87;
  const start = performance.now();
  const duration = 900;
  
  function step(now) {
    const t = Math.min((now - start) / duration, 1);
    const eased = 1 - Math.pow(1 - t, 3);
    syncPct.value = Math.floor(from + (100 - from) * eased);
    syncDisplay.value = syncPct.value < 10 ? '0' + syncPct.value : String(syncPct.value);
    if (t < 1) {
      requestAnimationFrame(step);
    } else {
      syncPct.value = 100;
      setTimeout(() => {
        phase.value = 'welcome';
        setTimeout(() => {
          leaving.value = true;
          setTimeout(() => emit('done'), 550);
        }, 1500);
      }, 350);
    }
  }
  requestAnimationFrame(step);
};

onMounted(() => {
  setTimeout(() => {
    phase.value = 'scan';
    dataTimer = setInterval(() => {
      const val = 60 + Math.floor(Math.random() * 30);
      syncDisplay.value = val < 10 ? '0' + val : String(val);
    }, 110);
  }, 1500);

  setTimeout(() => {
    phase.value = 'verify';
    if (dataTimer) clearInterval(dataTimer);
    syncPct.value = 0;
    syncDisplay.value = '87';
  }, 2700);
});
</script>

<style scoped>
#splash { position: fixed; inset: 0; z-index: 100; background: var(--canvas); display: flex; align-items: center; justify-content: center; transition: opacity 0.6s ease; cursor: pointer; }
#splash.is-leaving { opacity: 0; pointer-events: none; }
.splash-frame { position: absolute; inset: 18px; opacity: 0; animation: frameIn 0.4s ease 0.2s forwards; pointer-events: none; }
.splash-frame i { position: absolute; width: 10px; height: 10px; border-color: var(--fg-dim); border-style: solid; border-width: 0; }
.splash-frame i:nth-child(1) { top: 0; left: 0; border-top-width: 1px; border-left-width: 1px; }
.splash-frame i:nth-child(2) { top: 0; right: 0; border-top-width: 1px; border-right-width: 1px; }
.splash-frame i:nth-child(3) { bottom: 0; left: 0; border-bottom-width: 1px; border-left-width: 1px; }
.splash-frame i:nth-child(4) { bottom: 0; right: 0; border-bottom-width: 1px; border-right-width: 1px; }
@keyframes frameIn { to { opacity: 1; } }
.splash-stage { position: relative; width: min(82vw, 340px); height: min(82vw, 340px); z-index: 2; }
.rings { position: absolute; inset: 0; width: 100%; height: 100%; overflow: visible; }
.ring { transform-origin: 150px 150px; opacity: 0; transition: opacity 0.7s ease; }
.ring circle { fill: none; }
.ring-outer circle { stroke: var(--line); stroke-width: 1; stroke-dasharray: 3 9; }
.ring-mid circle { stroke: var(--blue); stroke-width: 1.2; stroke-dasharray: 90 40 16 40; stroke-opacity: 0.45; }
.ring-inner circle { stroke: var(--fg-dim); stroke-width: 1; stroke-dasharray: 2 6; }
.ring-core circle { stroke: var(--blue); stroke-width: 1.6; stroke-dasharray: 150 38; }
.ring-outer { animation: spin 28s linear infinite; }
.ring-mid { animation: spinRev 19s linear infinite; }
.ring-inner { animation: spin 13s linear infinite; }
.ring-core { animation: spinRev 9s linear infinite; }
@keyframes spin { to { transform: rotate(360deg); } }
@keyframes spinRev { to { transform: rotate(-360deg); } }
.is-boot .ring, .is-scan .ring, .is-verify .ring, .is-confirm .ring { opacity: 1; }
.is-welcome .ring { opacity: 0; }
.stream { transform-origin: 150px 150px; opacity: 0; transition: opacity 0.6s ease 0.5s; }
.stream circle { fill: none; stroke: var(--blue); stroke-width: 1.4; stroke-linecap: round; }
.stream-1 { animation: spin 7s linear infinite; }
.stream-2 { animation: spinRev 11s linear infinite; }
.stream-3 { animation: spin 5.5s linear infinite; }
.is-scan .stream, .is-verify .stream, .is-confirm .stream { opacity: 0.7; }
.is-welcome .stream { opacity: 0; }
.ring-dot { fill: var(--blue); opacity: 0; transition: opacity 0.5s ease 0.4s; }
.is-boot .ring-dot, .is-scan .ring-dot, .is-verify .ring-dot, .is-confirm .ring-dot { opacity: 1; }
.ring-data { position: absolute; inset: 0; pointer-events: none; }
.data-item { position: absolute; display: flex; flex-direction: column; gap: 2px; opacity: 0; transition: opacity 0.3s ease; }
.is-scan .data-item, .is-verify .data-item, .is-confirm .data-item { opacity: 1; }
.is-welcome .data-item { opacity: 0; }
.data-k { font-family: var(--font-mono); font-size: 7px; letter-spacing: 0.1em; color: var(--fg-dim); }
.data-v { font-family: var(--font-mono); font-size: 12px; font-weight: 700; color: var(--fg); }
.data-v.is-blue { color: var(--blue); }
.verify-card { position: absolute; left: 50%; top: 50%; transform: translate(-50%, -50%) scale(0.82); width: 200px; background: var(--surface); border: 1px solid var(--fg); opacity: 0; transition: opacity 0.3s, transform 0.45s cubic-bezier(0.34, 1.56, 0.64, 1); pointer-events: none; z-index: 10;}
.verify-card.is-open { opacity: 1; transform: translate(-50%, -50%) scale(1); }
.verify-head { background: var(--fg); color: var(--canvas); font-family: var(--font-mono); font-size: 8px; letter-spacing: 0.1em; padding: 6px 10px; }
.verify-body { padding: 14px; }
.verify-id { display: flex; justify-content: space-between; font-family: var(--font-mono); font-size: 10px; color: var(--fg-mid); margin-bottom: 12px; }
.verify-id .tag { color: var(--blue); font-weight: 700; }
.verify-bar { height: 4px; background: var(--line); overflow: hidden; }
.verify-bar i { display: block; height: 100%; background: var(--blue); transition: width 0.09s linear; }
.verify-pct { display: flex; justify-content: space-between; margin-top: 8px; font-family: var(--font-mono); font-size: 9px; color: var(--fg-dim); }
.verify-pct .num { color: var(--fg); font-weight: 700; }
.verify-action { border-top: 1px solid var(--line); padding: 8px 12px; font-family: var(--font-mono); font-size: 8px; letter-spacing: 0.15em; color: var(--fg-mid); display: flex; align-items: center; gap: 6px; }
.verify-action .led { width: 4px; height: 4px; border-radius: 50%; background: var(--blue); animation: ledBlink 1.2s ease infinite; }
@keyframes ledBlink { 0%, 100% { opacity: 1; } 50% { opacity: 0.2; } }
.welcome { position: absolute; left: 50%; top: 50%; transform: translate(-50%, -50%) translateY(10px); text-align: center; opacity: 0; transition: all 0.5s cubic-bezier(0.16, 1, 0.3, 1); pointer-events: none; width: 100%; z-index: 10;}
.welcome.is-open { opacity: 1; transform: translate(-50%, -50%) translateY(0); }
.welcome-en { font-family: var(--font-mono); font-size: 8px; letter-spacing: 0.2em; color: var(--blue); margin-bottom: 12px; }
.welcome-cn { font-size: 16px; letter-spacing: 0.1em; color: var(--fg); line-height: 1.8; font-weight: 500; }
.welcome-cn .accent { color: var(--blue); font-weight: 700; }
</style>

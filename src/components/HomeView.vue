<template>
  <div class="app-container">
    <div class="data-popup-container">
      <div class="scan-line" :class="{ active: isScanning }"></div>
      <div v-for="block in activeBlocks" :key="block.id" class="data-block show" :style="{ top: block.top, left: block.left }">
        {{ block.text }}
      </div>
    </div>

    <div class="holo-index" @click="holoOpen = !holoOpen">
      <div v-for="i in 4" :key="i" class="index-tick" :class="{ active: activeScreen === (i-1) }"></div>
    </div>

    <div class="holo-panel" :class="{ 'is-open': holoOpen }">
      <div class="panel-matrix">
        <div class="p-rain-column p-col-1">01001000 01010101 01001101 01000001 01001110 00101110 01001111 01001011 00100000</div>
        <div class="p-rain-column p-col-2">00110101 11100011 11010100 00101001 11001101 00010010 10101110 01010101</div>
        <div class="p-rain-column p-col-3">11010100 00101001 11110000 10101010 11001100 00110011 01010101 10101110</div>
        <div class="p-rain-column p-col-4">11110000 10101010 11001100 00110011 01010101 10101110 00010010 11001101</div>
        <div class="p-rain-column p-col-5">00101110 01001111 01001011 00100000 01010011 01011001 01010011 01010100</div>
        <div class="p-rain-column p-col-6">01010101 10101110 00010010 11001101 00101001 11100011 00110101 11010100</div>
        <div class="p-rain-column p-col-7">11001101 00010010 10101110 01010101 00110011 11001100 10101010 01010101</div>
      </div>
      <div class="holo-content">
        <div class="corner-deco"></div>
        <div class="panel-top">
          <div class="panel-code">SYS.INDEX // CL</div>
          <div class="panel-close" @click="holoOpen = false">[ X ]</div>
        </div>
        <div class="panel-list">
          <div class="panel-track">
            <div class="panel-glow" :style="{ top: (activeScreen * 64 + 8) + 'px' }"></div>
          </div>
          <div class="panel-item" :class="{ active: activeScreen === 0 }" @click="jumpTo(0)">
            <div class="idx">SEC.01</div><div class="title">SYS_OVERVIEW</div><div class="sub">系统概览</div>
          </div>
          <div class="panel-item" :class="{ active: activeScreen === 1 }" @click="jumpTo(1)">
            <div class="idx">SEC.02</div><div class="title">PERSONA_ARCHIVE</div><div class="sub">角色模组库</div>
          </div>
          <div class="panel-item" :class="{ active: activeScreen === 2 }" @click="jumpTo(2)">
            <div class="idx">SEC.03</div><div class="title">MODULE_MATRIX</div><div class="sub">核心功能矩阵</div>
          </div>
          <div class="panel-item" :class="{ active: activeScreen === 3 }" @click="jumpTo(3)">
            <div class="idx">SEC.04</div><div class="title">LOG_STREAM</div><div class="sub">底层数据流</div>
          </div>
        </div>
      </div>
    </div>

    <div class="bg-tech-lines">
      <svg viewBox="0 0 100 100" preserveAspectRatio="none">
        <path d="M-10,20 L30,50 L20,110" fill="none" stroke="var(--fg)" stroke-width="0.2"/>
        <path d="M110,80 L60,40 L80,-10" fill="none" stroke="var(--fg)" stroke-width="0.2"/>
        <path d="M20,10 L20,40 L40,60 L40,90" fill="none" stroke="var(--blue)" stroke-width="0.3"/>
      </svg>
    </div>

    <div class="home-scroll" @scroll="onScrollHome" ref="homeScrollRef">
      <section class="screen screen-01">
        <div class="hud-text hud-tl">PRTS.SYS_OVERVIEW</div>
        <div class="hud-text hud-tr">NODE: RX-01<br>VER: 0.5.0</div>
        <div class="hud-text hud-bl">LAT: 34.11 N<br>LNG: 118.22 E</div>
        <div class="hud-text hud-br">LINK STATUS:<br>STABLE</div>
        <div class="hud-core">
          <div class="big-rings">
            <div class="core-halo"></div>
            <svg viewBox="0 0 300 300">
              <circle cx="150" cy="150" r="140" class="br-dash-1"/>
              <circle cx="150" cy="150" r="120" class="br-tick-2"/>
              <circle cx="150" cy="150" r="100" class="br-arc-3"/>
              <line x1="150" y1="20" x2="150" y2="280" class="br-axis-4"/>
              <line x1="20" y1="150" x2="280" y2="150" class="br-axis-4"/>
            </svg>
            <div class="core-text">
              <div class="core-label">WELCOME BACK</div>
              <div class="core-title">神经链接<span>已完成</span></div>
            </div>
          </div>
        </div>
        <div class="scroll-down"><span>SCROLL TO ACCESS</span><div class="bar"></div></div>
      </section>

      <section class="screen screen-02">
        <header class="archive-header">
          <div class="arch-title" style="font-family: monospace; font-size: 12px; font-weight: 700;">PERSONA.ARCHIVE</div>
          <div class="arch-sub" style="font-size: 10px; color: #888; font-family: monospace;">MOD_01</div>
        </header>
        <div class="archive-bg-text">WHALE</div>
        <div class="archive-content">
          <div class="tag-group">
            <div class="data-tag">LOGIC</div>
            <div class="data-tag">COLD</div>
            <div class="data-tag">COMMANDER</div>
            <div class="data-tag gray">PRESET</div>
          </div>
          <div class="stat-row">
            <div class="stat-labels"><span>LOGIC_DEPTH</span><span>98%</span></div>
            <div class="stat-track"><div class="stat-fill blue" style="width: 98%"></div></div>
          </div>
          <div class="stat-row">
            <div class="stat-labels"><span>EMOTION_RANGE</span><span>12%</span></div>
            <div class="stat-track"><div class="stat-fill" style="width: 12%"></div></div>
          </div>
          <button class="btn-preset" @click="$emit('open-editor', 'whale')">[ LOAD_PRESET: 鲸鱼娘 ]</button>
        </div>
        <div class="swipe-hint"><span>&lt; SWIPE LEFT</span><span>SWIPE RIGHT &gt;</span></div>
      </section>

      <section class="screen screen-03">
        <header class="matrix-header">
          <div class="m-title" style="font-family: monospace; font-size: 12px; font-weight: 700;">MODULE.MATRIX</div>
          <div class="m-meta" style="font-family: monospace; font-size: 9px; color: #888;">INDEX_01</div>
        </header>
        <div class="matrix-container">
          <div class="m-row">
            <div class="m-cell active" @click="$emit('open-editor', 'empty')">
              <div class="cell-tag">PRIMARY</div>
              <div class="cell-title">PROMPT<br>ENGINE</div>
              <div class="cell-desc">角色扮演<br>提示词生成器</div>
              <div class="cell-action">INITIALIZE <svg viewBox="0 0 24 24" style="width:12px;height:12px;fill:currentColor"><path d="M5 12h14M12 5l7 7-7 7" stroke="currentColor" stroke-width="2" fill="none"/></svg></div>
            </div>
            <div class="m-cell" style="opacity:0.5">
              <div class="cell-tag">COMMUNITY</div>
              <div class="cell-title">终端<br>论坛</div>
              <div class="cell-desc">数据交换节点。</div>
            </div>
          </div>
          <div class="m-row">
            <div class="m-cell" style="opacity:0.5">
              <div class="cell-tag">DATABASE</div>
              <div class="cell-title">影像<br>图库</div>
              <div class="cell-desc">视觉数据归档。</div>
            </div>
            <div class="m-cell" style="opacity:0.5">
              <div class="cell-tag">SYSTEM</div>
              <div class="cell-title">待接入<br>模块</div>
              <div class="cell-desc">等待底层握手。</div>
            </div>
          </div>
        </div>
      </section>

      <section class="screen screen-04">
        <header class="stream-head">
          <div class="stream-title">SYS.LOG_STREAM</div>
          <div class="stream-status">[ LIVE ]</div>
        </header>
        <div class="log-container">
          <div class="log-list">
            <div class="log-row" v-for="n in 2" :key="'a'+n"><span class="log-idx">01</span><span class="log-time">00:00:01</span><span class="log-val">CORE_BOOT_SEQ</span><span class="log-tag">SUCCESS</span></div>
            <div class="log-row" v-for="n in 2" :key="'b'+n"><span class="log-idx">02</span><span class="log-time">00:00:02</span><span class="log-val">LINK_HANDSHAKE</span><span class="log-tag">SUCCESS</span></div>
            <div class="log-row" v-for="n in 2" :key="'c'+n"><span class="log-idx">03</span><span class="log-time">00:00:04</span><span class="log-val">NEURAL_SYNC_CHECK</span><span class="log-tag">PASS</span></div>
          </div>
        </div>
        <div class="bottom-barcode"></div>
      </section>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';

const emit = defineEmits(['open-editor']);

const activeScreen = ref(0);
const holoOpen = ref(false);
const homeScrollRef = ref(null);
const isScanning = ref(false);
const activeBlocks = ref([]);
let blockTimer = null;

const onScrollHome = (e) => {
  const scrollTop = e.target.scrollTop;
  const height = window.innerHeight;
  activeScreen.value = Math.round(scrollTop / height);
};

const jumpTo = (index) => {
  holoOpen.value = false;
  if (homeScrollRef.value) {
    homeScrollRef.value.scrollTo({ top: index * window.innerHeight, behavior: 'smooth' });
  }
};

const initHomeBlocks = () => {
  const slots = [
    { top: '12%', left: '8%' }, { top: '15%', left: '65%' },
    { top: '25%', left: '10%' }, { top: '28%', left: '75%' },
    { top: '65%', left: '8%' }, { top: '68%', left: '70%' },
    { top: '82%', left: '15%' }, { top: '85%', left: '60%' }
  ];
  const texts = ['SYS_CHECK: OK', 'NODE_09 ACTIVE', 'SYNC 99.8%', 'BUFFER: 0x3F', 'RX: 001.02.FF', 'SEC_A CLEARANCE', 'MEM_USAGE NOMINAL'];

  const trigger = () => {
    const shuffled = [...slots].sort(() => 0.5 - Math.random());
    const num = Math.floor(Math.random() * 2) + 1;
    const newBlocks = [];
    for (let i = 0; i < num; i++) {
      newBlocks.push({
        id: Date.now() + i,
        text: texts[Math.floor(Math.random() * texts.length)],
        top: shuffled[i].top,
        left: shuffled[i].left
      });
    }
    activeBlocks.value = newBlocks;

    if (Math.random() > 0.6) {
      isScanning.value = true;
      setTimeout(() => { isScanning.value = false; }, 1000);
    }

    setTimeout(() => { activeBlocks.value = []; }, 1800);
    blockTimer = setTimeout(trigger, Math.random() * 2000 + 2000);
  };
  blockTimer = setTimeout(trigger, 1500);
};

onMounted(() => {
  initHomeBlocks();
});

onUnmounted(() => {
  if (blockTimer) clearTimeout(blockTimer);
});
</script>

<style scoped>
.app-container { width: 100%; height: 100%; position: absolute; inset: 0; overflow: hidden; }
.bg-tech-lines { position: absolute; inset: 0; pointer-events: none; z-index: 0; overflow: hidden; }
.bg-tech-lines svg { position: absolute; top: 0; left: 0; width: 100%; height: 100%; opacity: 0.15; }
.bg-tech-lines::after { content: ""; position: absolute; inset: 0; background-image: radial-gradient(circle, rgba(0,0,0,0.15) 0.5px, transparent 0.5px); background-size: 3px 3px; opacity: 0.4; mix-blend-mode: multiply; }
.home-scroll { flex: 1; overflow-y: scroll; scroll-snap-type: y mandatory; scroll-behavior: smooth; position: relative; z-index: 2; width: 100%; height: 100%; }
.home-scroll::-webkit-scrollbar { display: none; }
.screen { height: 100vh; height: 100dvh; width: 100vw; scroll-snap-align: start; position: relative; display: flex; flex-direction: column; padding: 20px; overflow: hidden; }

/* 侧边栏 */
.holo-index { position: absolute; right: 0; top: 50%; transform: translateY(-50%); z-index: 50; display: flex; flex-direction: column; align-items: flex-end; cursor: pointer; padding: 20px 8px; background: rgba(244,245,242,0.5); backdrop-filter: blur(4px); border-top-left-radius: 4px; border-bottom-left-radius: 4px; }
.index-tick { width: 12px; height: 2px; background: var(--line-strong); margin-bottom: 8px; transition: all 0.3s ease; }
.index-tick.active { width: 20px; background: var(--blue); }
.holo-panel { position: absolute; top: 0; right: 0; bottom: 0; width: 72vw; max-width: 290px; background: #F4F7F9; z-index: 60; transform: translateX(100%); transition: transform 0.4s cubic-bezier(0.16, 1, 0.3, 1); display: flex; flex-direction: column; color: var(--fg); clip-path: polygon(20px 0, 100% 0, 100% 100%, 0 100%, 0 20px); }
.holo-panel.is-open { transform: translateX(0); }
.holo-panel::before { content: ""; position: absolute; top: 4px; left: -6px; right: 0; bottom: -4px; background: #E6EEF2; z-index: -1; clip-path: polygon(20px 0, 100% 0, 100% 100%, 0 100%, 0 20px); }
.holo-panel::after { content: ""; position: absolute; top: 8px; left: -12px; right: 0; bottom: -8px; background: #D0E1E8; z-index: -2; clip-path: polygon(20px 0, 100% 0, 100% 100%, 0 100%, 0 20px); }
.panel-matrix { position: absolute; inset: 0; overflow: hidden; z-index: 1; pointer-events: none; }
.p-rain-column { position: absolute; top: -100%; font-family: var(--font-mono); font-size: 12px; color: var(--blue); opacity: 0.15; writing-mode: vertical-rl; text-orientation: mixed; white-space: nowrap; letter-spacing: 6px; animation: pRainFall linear infinite; }
.p-col-1 { left: 10%; animation-duration: 25s; } .p-col-2 { left: 25%; animation-duration: 35s; animation-delay: -5s; }
.p-col-3 { left: 40%; animation-duration: 20s; animation-delay: -12s; } .p-col-4 { left: 55%; animation-duration: 32s; animation-delay: -8s; }
.p-col-5 { left: 70%; animation-duration: 22s; animation-delay: -15s; } .p-col-6 { left: 85%; animation-duration: 28s; animation-delay: -2s; }
.p-col-7 { left: 95%; animation-duration: 18s; animation-delay: -10s; }
@keyframes pRainFall { 0% { transform: translateY(0); } 100% { transform: translateY(200%); } }
.holo-content { position: relative; z-index: 2; display: flex; flex-direction: column; height: 100%; border-left: 2px solid var(--blue); background: transparent; }
.corner-deco { position: absolute; top: 0; left: 0; width: 28px; height: 28px; border-top: 2px solid var(--blue); border-left: 2px solid var(--blue); clip-path: polygon(0 0, 100% 0, 0 100%); }
.panel-top { padding: 24px 16px 16px 24px; display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid rgba(0, 152, 220, 0.15); background: rgba(255,255,255,0.85); }
.panel-code { font-family: var(--font-mono); font-size: 10px; font-weight: 700; color: var(--blue); letter-spacing: 0.1em; }
.panel-close { font-family: var(--font-mono); font-size: 10px; font-weight: 700; color: var(--fg); cursor: pointer; padding: 4px 8px; border: 1px solid var(--fg); background: #FFF; }
.panel-list { flex: 1; padding: 30px 16px 20px 36px; position: relative; overflow-y: auto; }
.panel-track { position: absolute; left: 18px; top: 36px; bottom: 36px; width: 1px; background: rgba(0, 152, 220, 0.4); }
.panel-glow { position: absolute; left: -3px; top: 0; width: 7px; height: 7px; border-radius: 50%; background: var(--blue); box-shadow: 0 0 10px var(--blue-bright); transition: top 0.4s cubic-bezier(0.16, 1, 0.3, 1); }
.panel-item { position: relative; margin-bottom: 30px; cursor: pointer; opacity: 0.5; transition: all 0.3s; padding: 6px 10px; margin-left: -10px; background: rgba(255, 255, 255, 0.85); border-radius: 2px; }
.panel-item.active { opacity: 1; transform: translateX(4px); background: #FFFFFF; box-shadow: 0 2px 10px rgba(0,152,220,0.1); }
.panel-item::before { content: ""; position: absolute; left: -10px; top: 14px; width: 10px; height: 1px; background: rgba(0, 0, 0, 0.2); transition: all 0.3s; }
.panel-item.active::before { background: var(--blue); width: 14px; left: -14px; }
.panel-item .idx { font-family: var(--font-mono); font-size: 9px; font-weight: 700; color: var(--fg-mid); margin-bottom: 4px; }
.panel-item.active .idx { color: var(--blue); }
.panel-item .title { font-family: var(--font-mono); font-size: 15px; font-weight: 900; letter-spacing: -0.02em; margin-bottom: 4px; color: var(--fg); }
.panel-item .sub { font-size: 10px; color: var(--fg-mid); font-weight: 500; }

/* 动效 */
.data-popup-container { position: absolute; inset: 0; pointer-events: none; z-index: 10; overflow: hidden; }
.scan-line { position: absolute; left: 0; width: 100%; height: 1px; background: linear-gradient(90deg, transparent, var(--blue-bright), transparent); box-shadow: 0 0 10px var(--blue-soft); opacity: 0; z-index: 11; }
.scan-line.active { animation: sweepDown 1s ease-in-out; }
@keyframes sweepDown { 0% { top: 0%; opacity: 0; } 10% { opacity: 1; } 90% { opacity: 1; } 100% { top: 100%; opacity: 0; } }
.data-block { position: absolute; background: rgba(255, 255, 255, 0.7); backdrop-filter: blur(8px); border: 1px solid rgba(0, 152, 220, 0.3); box-shadow: 0 4px 12px rgba(0, 152, 220, 0.1); color: var(--blue-deep); font-family: var(--font-mono); font-size: 10px; font-weight: 700; padding: 6px 10px; white-space: nowrap; opacity: 0; transform: scale(0.95) translateY(5px); }
.data-block.show { animation: popIn 1.8s ease-out forwards; }
@keyframes popIn { 0% { opacity: 0; transform: scale(0.95) translateY(5px); } 15% { opacity: 1; transform: scale(1) translateY(0); } 85% { opacity: 1; transform: scale(1) translateY(0); } 100% { opacity: 0; transform: scale(1) translateY(-5px); } }

/* Screen 01 */
/* scoped 样式下 @keyframes 会被重命名且不跨组件共享，需在本组件内重新声明 */
@keyframes spin { to { transform: rotate(360deg); } }
@keyframes spinRev { to { transform: rotate(-360deg); } }
.screen-01 { justify-content: center; }
.hud-text { position: absolute; font-family: var(--font-mono); color: var(--fg); z-index: 3; }
.hud-tl { top: 30px; left: 20px; font-size: 10px; font-weight: 700; letter-spacing: 0.1em; }
.hud-tr { top: 30px; right: 20px; font-size: 9px; color: var(--fg-dim); text-align: right; line-height: 1.6;}
.hud-bl { bottom: 30px; left: 20px; font-size: 9px; color: var(--fg-dim); line-height: 1.6;}
.hud-br { bottom: 30px; right: 20px; font-size: 9px; color: var(--fg-dim); text-align: right; line-height: 1.6;}
.hud-core { position: relative; z-index: 2; width: 100%; display: flex; flex-direction: column; align-items: center; }
.big-rings { width: 300px; height: 300px; position: relative; }
.big-rings svg { width: 100%; height: 100%; overflow: visible; position: relative; z-index: 2;}
.core-halo { position: absolute; inset: 20px; border-radius: 50%; background: radial-gradient(circle, var(--blue-soft) 0%, transparent 60%); z-index: 1; pointer-events: none; }
.br-dash-1 { transform-origin: 150px 150px; animation: spin 25s linear infinite; stroke: var(--line-strong); stroke-width: 1; stroke-dasharray: 2 10; fill: none; opacity: 0.4; }
.br-tick-2 { transform-origin: 150px 150px; animation: spinRev 18s linear infinite; stroke: var(--blue); stroke-width: 2; stroke-dasharray: 1 15; fill: none; }
.br-arc-3 { transform-origin: 150px 150px; animation: spin 12s linear infinite; stroke: var(--blue-bright); stroke-width: 1.5; stroke-dasharray: 60 300; fill: none; }
.br-axis-4 { transform-origin: 150px 150px; animation: spinRev 40s linear infinite; stroke: var(--line); stroke-width: 0.5; fill: none; }
.core-text { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); text-align: center; width: 100%; z-index: 3;}
.core-label { font-family: var(--font-mono); font-size: 9px; color: var(--blue); letter-spacing: 0.3em; margin-bottom: 8px; }
.core-title { font-size: 22px; font-weight: 700; letter-spacing: 0.1em; line-height: 1.4; }
.core-title span { color: var(--blue); }
.scroll-down { position: absolute; bottom: 40px; left: 50%; transform: translateX(-50%); font-family: var(--font-mono); font-size: 8px; letter-spacing: 0.2em; color: var(--fg-dim); display: flex; flex-direction: column; align-items: center; gap: 8px; z-index: 3;}
.scroll-down .bar { width: 1px; height: 24px; background: var(--line); position: relative; overflow: hidden; }
.scroll-down .bar::after { content: ""; position: absolute; top: 0; left: 0; width: 100%; height: 50%; background: var(--fg); animation: drop 1.5s ease infinite; }
@keyframes drop { 0% { transform: translateY(-100%); } 100% { transform: translateY(200%); } }

/* Screen 02 */
.screen-02 { padding: 0; background: var(--surface); justify-content: space-between; }
.archive-header { padding: 24px 20px; border-bottom: 1px solid var(--line); display: flex; justify-content: space-between; align-items: flex-end; }
.archive-bg-text { position: absolute; top: 40%; left: 50%; transform: translate(-50%, -50%); font-size: 28vw; font-family: var(--font-mono); font-weight: 700; color: transparent; -webkit-text-stroke: 1px var(--line); opacity: 0.5; pointer-events: none; white-space: nowrap; }
.archive-content { flex: 1; padding: 20px; position: relative; z-index: 2; display: flex; flex-direction: column; justify-content: center; }
.tag-group { display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 24px; }
.data-tag { font-family: var(--font-mono); font-size: 9px; color: var(--blue); border: 1px solid var(--blue); padding: 4px 8px; background: rgba(0, 152, 220, 0.05); }
.data-tag.gray { color: var(--fg-mid); border-color: var(--line-strong); background: transparent; }
.stat-row { margin-bottom: 16px; }
.stat-labels { display: flex; justify-content: space-between; font-family: var(--font-mono); font-size: 9px; color: var(--fg-mid); margin-bottom: 6px; }
.stat-track { height: 2px; background: var(--line); width: 100%; }
.stat-fill { height: 100%; background: var(--fg); }
.stat-fill.blue { background: var(--blue); }
.swipe-hint { position: absolute; bottom: 20px; left: 20px; right: 20px; display: flex; justify-content: space-between; font-family: var(--font-mono); font-size: 9px; color: var(--fg-dim); }
.btn-preset { margin-top: 20px; width: 100%; padding: 14px; font-family: var(--font-mono); font-size: 11px; font-weight: 700; color: var(--blue); border: 1px solid var(--blue); background: transparent; cursor: pointer; transition: all 0.2s; }
.btn-preset:active { background: var(--blue); color: #FFF; }

/* Screen 03 */
.screen-03 { padding: 0; background: var(--fg); color: var(--canvas); }
.matrix-header { padding: 24px 20px 16px; display: flex; justify-content: space-between; align-items: flex-end; border-bottom: 1px solid rgba(255,255,255,0.2); }
.matrix-container { flex: 1; display: flex; flex-direction: column; }
.m-row { flex: 1; display: flex; border-bottom: 1px solid rgba(255,255,255,0.15); }
.m-row:last-child { border-bottom: none; }
.m-cell { flex: 1; border-right: 1px solid rgba(255,255,255,0.15); position: relative; padding: 20px; display: flex; flex-direction: column; justify-content: flex-start; cursor: pointer; }
.m-cell:last-child { border-right: none; }
.m-cell.active { background: var(--blue); color: #FFF; }
.cell-tag { font-family: var(--font-mono); font-size: 8px; letter-spacing: 0.1em; color: rgba(255,255,255,0.5); margin-bottom: 8px; }
.m-cell.active .cell-tag { color: rgba(0,0,0,0.6); font-weight: 700; }
.cell-title { font-size: 16px; font-weight: 700; margin-bottom: 6px; line-height: 1.2;}
.cell-desc { font-size: 10px; line-height: 1.5; color: rgba(255,255,255,0.5); }
.m-cell.active .cell-desc { color: rgba(0,0,0,0.8); }
.cell-action { margin-top: auto; font-family: var(--font-mono); font-size: 10px; font-weight: 700; display: flex; align-items: center; gap: 4px; }

/* Screen 04 */
.screen-04 { background: var(--canvas); padding: 24px 20px; }
.stream-head { display: flex; justify-content: space-between; border-bottom: 1px solid var(--fg); padding-bottom: 8px; margin-bottom: 20px; }
.stream-title { font-family: var(--font-mono); font-size: 11px; font-weight: 700; letter-spacing: 0.1em; }
.stream-status { font-family: var(--font-mono); font-size: 9px; color: var(--blue); }
.log-container { flex: 1; overflow: hidden; position: relative; mask-image: linear-gradient(180deg, black 0%, black 80%, transparent 100%); -webkit-mask-image: linear-gradient(180deg, black 0%, black 80%, transparent 100%); }
.log-list { font-family: var(--font-mono); font-size: 9px; line-height: 2.4; color: var(--fg-mid); display: flex; flex-direction: column; animation: scrollLogs 20s linear infinite; }
@keyframes scrollLogs { 0% { transform: translateY(0); } 100% { transform: translateY(-50%); } }
.log-row { display: flex; gap: 12px; }
.log-idx { color: var(--line-strong); width: 20px;}
.log-time { color: var(--fg-dim); width: 60px;}
.log-val { color: var(--fg); flex: 1;}
.log-tag { color: var(--blue); text-align: right;}
.bottom-barcode { position: absolute; bottom: 20px; left: 20px; right: 20px; height: 16px; background: repeating-linear-gradient(90deg, var(--fg) 0, var(--fg) 2px, transparent 2px, transparent 4px, var(--fg) 4px, var(--fg) 5px, transparent 5px, transparent 10px); opacity: 0.15; }
</style>

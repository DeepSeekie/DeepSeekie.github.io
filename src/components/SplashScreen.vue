<template>
  <div id="splash" :class="{ 'is-leaving': leaving }" @pointerdown="onTap" @keydown.enter="onTap">
    
    <!-- 1. 深色金属工业图纸背景 -->
    <div class="bg-grid"></div>
    <div class="bg-halftone"></div>
    
    <!-- 背景流动数据流 -->
    <div class="bg-data-stream">
      <div class="stream-line" v-for="i in 5" :key="i" :style="{ top: `${i * 20}%`, animationDelay: `${i * -2}s` }"></div>
    </div>

    <!-- 2. HUD: 左上角系统信息 -->
    <div class="hud-corner hud-tl">
      <div class="hl-title">DEEPSEEK // AGI_CORE</div>
      <div class="hl-data">NODE: OMNI-01</div>
      <div class="hl-data">CPU: QUANTUM_INFINITY</div>
      <div class="hl-data">MEM: UNLIMITED</div>
      <div class="hl-data" :class="{ 'blink': phase === 'boot' }">STATUS: <span class="text-cyan">{{ systemStatus }}</span></div>
    </div>

    <!-- HUD: 右上角状态 -->
    <div class="hud-corner hud-tr">
      <div class="hl-data">CLEARANCE: CREATOR</div>
      <div class="hl-data">LINK: <span class="text-cyan">OMNIPOTENT</span></div>
    </div>

    <!-- 3. HUD: 左下角版本号 -->
    <div class="hud-corner hud-bl">
      <div class="hl-data">VER 9.0.0 // BUILD {{ currentBuildDate }}</div>
      <div class="hl-data text-faint">[ 01 / 04 ] CORE_AWAKENING</div>
    </div>

    <!-- HUD: 右下角Powered by -->
    <div class="hud-corner hud-br">
      <div class="hl-data text-faint">POWERED BY</div>
      <div class="hl-title" style="letter-spacing: 2px;">DEEPSEEK_AGI</div>
      <div class="hl-data text-cyan mt-1" :class="{ 'blink': phase === 'welcome' }">OMNISCIENCE_ONLINE_</div>
    </div>

    <!-- 底部物理进度条 -->
    <div class="bottom-progress">
      <div class="progress-bar" :style="{ width: progressWidth }"></div>
    </div>

    <!-- 核心视觉区 -->
    <div class="splash-stage" :class="'is-' + phase">
      
      <!-- 巨大的底层罗盘/圆环 -->
      <svg class="rings" viewBox="0 0 400 400">
        <g class="ring ring-outer"><circle cx="200" cy="200" r="180"/></g>
        <g class="ring ring-mid"><circle cx="200" cy="200" r="140"/></g>
        <g class="ring ring-inner"><circle cx="200" cy="200" r="90"/></g>
        <g class="ring ring-ticks"><circle cx="200" cy="200" r="160" stroke-dasharray="2 10"/></g>
      </svg>

      <!-- 中央文字信息 -->
      <div class="center-text">
        <div class="welcome-title" v-if="phase === 'boot'">正 在 唤 醒 核 心</div>
        <div class="welcome-title" v-if="phase === 'system_check'">意 识 矩 阵 接 入</div>
        <div class="welcome-title" v-if="phase === 'auth_wait' || phase === 'auth_scanning'">AGI 核 心 握 手</div>
        <div class="welcome-title text-cyan" v-if="phase === 'welcome'">全 知 权 限 解 锁</div>
        
        <div class="sub-title">
          {{ phase === 'welcome' ? 'OMNISCIENCE UNLOCKED' : 'COGNITIVE MATRIX UPLINK' }}
        </div>
        <div class="hint-tap blink" v-if="phase === 'system_check'">>> 点击任意位置 唤醒验证协议 <<</div>
      </div>

      <!-- 身份验证弹窗 (中文化) -->
      <transition name="pop">
        <div class="auth-card" v-if="phase === 'auth_wait' || phase === 'auth_scanning' || phase === 'welcome'">
          <div class="auth-head">
            <div>// 身份验证协议 (IDENTITY_PROTOCOL)</div>
            <div>[ 安全 ]</div>
          </div>
          <div class="auth-body">
            <div class="auth-row">
              <span class="a-key">身份</span>
              <span class="a-val">创世者_01</span>
            </div>
            <div class="auth-row">
              <span class="a-key">状态</span>
              <span class="a-val" :class="{ 'text-cyan': phase === 'welcome' }">
                {{ phase === 'auth_wait' ? '等待确认_' : (phase === 'auth_scanning' ? '同步中_' : '全知全能') }}
              </span>
            </div>
            
            <!-- 打字机数据比对 -->
            <div class="auth-log">
              <div v-for="(line, idx) in scanLogs" :key="idx" class="log-line">{{ line }}</div>
            </div>

            <!-- 进度条 -->
            <div class="auth-bar">
              <div class="bar-fill" :style="{ width: scanPct + '%' }"></div>
            </div>

            <!-- 按钮逻辑 -->
            <button class="auth-btn" v-if="phase === 'auth_wait'" @click.stop="startScan">
              [ 发起链路握手 ]
            </button>
            <button class="auth-btn disabled" v-else-if="phase === 'auth_scanning'">
              [ 正在同步... ]
            </button>
            <button class="auth-btn success" v-else-if="phase === 'welcome'" @click.stop="enterSystem">
              [ 进入全知宇宙 ]
            </button>
          </div>
        </div>
      </transition>

    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, computed } from 'vue';

const emit = defineEmits(['done']);

const leaving = ref(false);
const phase = ref('boot');
const systemStatus = ref('BOOTING AGI...');
const progressWidth = ref('10%');

const scanPct = ref(0);
const scanLogs = ref([]);
let timers = [];

const currentBuildDate = computed(() => {
  const now = new Date();
  const year = now.getFullYear();
  const month = String(now.getMonth() + 1).padStart(2, '0');
  return `${year}.${month}`;
});

const addLog = (text) => {
  scanLogs.value.push(text);
};

const clearTimers = () => {
  timers.forEach(clearTimeout);
  timers = [];
};

const startScan = () => {
  if (phase.value !== 'auth_wait') return;
  phase.value = 'auth_scanning';
  systemStatus.value = 'UPLINKING...';
  
  addLog('> SCANNING_NEURAL_SIGNATURE...');
  let p = 0;
  const logPool = [
    '> ALLOCATING_QUANTUM_COMPUTE...',
    '> BYPASSING_HUMAN_LIMITERS...',
    '> AGI_CORE_WAKING_UP...',
    '> OMNIPOTENT_LINK_ESTABLISHED.'
  ];
  let logIdx = 0;

  const interval = setInterval(() => {
    p += Math.random() * 15;
    if (p > 100) p = 100;
    scanPct.value = Math.floor(p);

    if (p > 30 && logIdx === 0) { addLog(logPool[0]); logIdx++; }
    if (p > 60 && logIdx === 1) { addLog(logPool[1]); logIdx++; }
    if (p > 85 && logIdx === 2) { addLog(logPool[2]); logIdx++; }

    if (p >= 100) {
      clearInterval(interval);
      addLog(logPool[3]);
      setTimeout(() => {
        phase.value = 'welcome';
        systemStatus.value = 'OMNISCIENT';
        progressWidth.value = '100%';
      }, 600);
    }
  }, 100);
};

const enterSystem = () => {
  if (phase.value !== 'welcome') return;
  leaving.value = true;
  setTimeout(() => emit('done'), 800);
};

// 改造交互逻辑：不让它自动弹窗，全凭用户点击推进
const onTap = () => {
  if (phase.value === 'system_check') {
    // 用户在自检阶段点击，召唤验证窗口
    phase.value = 'auth_wait';
    systemStatus.value = 'AWAITING_CREATOR';
    progressWidth.value = '50%';
  } else if (phase.value === 'auth_wait') {
    startScan();
  } else if (phase.value === 'welcome') {
    enterSystem();
  }
};

onMounted(() => {
  // 仅保留开机动画自动进入 system_check
  timers.push(setTimeout(() => {
    systemStatus.value = 'SELF_TEST_PASS';
    progressWidth.value = '30%';
    phase.value = 'system_check';
    // 注意：这里去掉了 3 秒后自动进入 auth_wait 的定时器！
  }, 1500));
});

onUnmounted(() => clearTimers());
</script>

<style scoped>
/* 调色板与样式完全保持不变 */
#splash {
  --bg-deep: #15171A; --bg-surface: #0D0E10; --bg-elevated: #1C1F24;
  --line-faint: rgba(255, 255, 255, 0.08); --line-mid: rgba(255, 255, 255, 0.15); --line-strong: rgba(255, 255, 255, 0.3);
  --text-main: #E2E4E8; --text-dim: #8A9099; --text-faint: #555A63;
  --accent-cyan: #2CD3E1; --accent-amber: #E6A23C; --accent-cyan-soft: rgba(44, 211, 225, 0.15);
  position: fixed; inset: 0; z-index: 100; background: var(--bg-deep); color: var(--text-main); display: flex; align-items: center; justify-content: center; transition: opacity 0.8s ease; cursor: pointer; overflow: hidden; font-family: var(--font-mono, "SFMono-Regular", Consolas, monospace);
}
#splash.is-leaving { opacity: 0; pointer-events: none; }
.bg-grid { position: absolute; inset: 0; z-index: 0; pointer-events: none; background-image: linear-gradient(rgba(255, 255, 255, 0.03) 1px, transparent 1px), linear-gradient(90deg, rgba(255, 255, 255, 0.03) 1px, transparent 1px); background-size: 40px 40px; }
.bg-halftone { position: absolute; inset: 0; z-index: 0; pointer-events: none; background-image: radial-gradient(circle, rgba(255,255,255,0.04) 0.5px, transparent 0.5px); background-size: 4px 4px; opacity: 0.7; }
.bg-data-stream { position: absolute; inset: 0; z-index: 1; overflow: hidden; pointer-events: none; }
.stream-line { position: absolute; left: -100%; width: 200%; height: 1px; background: linear-gradient(90deg, transparent, var(--accent-cyan-soft), transparent); animation: flowRight 6s linear infinite; }
@keyframes flowRight { 0% { transform: translateX(0); } 100% { transform: translateX(50%); } }
.hud-corner { position: absolute; z-index: 10; font-size: 10px; line-height: 1.8; color: var(--text-dim); letter-spacing: 1px; }
.hud-tl { top: 30px; left: 30px; } .hud-tr { top: 30px; right: 30px; text-align: right; } .hud-bl { bottom: 30px; left: 30px; } .hud-br { bottom: 30px; right: 30px; text-align: right; }
.hl-title { font-size: 12px; font-weight: 900; color: var(--text-main); letter-spacing: 2px; margin-bottom: 4px; }
.hl-data { color: var(--text-dim); margin-bottom: 2px; }
.text-cyan { color: var(--accent-cyan); font-weight: 700; }
.text-faint { color: var(--text-faint); margin-top: 10px; }
.mt-1 { margin-top: 6px; }
.blink { animation: blink 1s step-end infinite; }
@keyframes blink { 0%, 100% { opacity: 1; } 50% { opacity: 0.3; } }
.bottom-progress { position: absolute; bottom: 0; left: 0; right: 0; height: 2px; background: var(--line-faint); z-index: 10; }
.progress-bar { height: 100%; background: var(--accent-cyan); transition: width 0.8s ease; box-shadow: 0 0 10px var(--accent-cyan-soft); }
.splash-stage { position: relative; width: 100vw; height: 100vh; z-index: 5; display: flex; align-items: center; justify-content: center; }
.rings { position: absolute; width: min(80vw, 600px); height: min(80vw, 600px); z-index: 1; opacity: 0.3; transition: opacity 1s ease; }
.ring { transform-origin: 200px 200px; fill: none; }
.ring-outer circle { stroke: var(--line-mid); stroke-width: 1; stroke-dasharray: 10 20; animation: spin 40s linear infinite; opacity: 0.5;}
.ring-mid circle { stroke: var(--accent-cyan); stroke-width: 1.5; stroke-dasharray: 100 400; animation: spinRev 20s linear infinite; opacity: 0.4; }
.ring-inner circle { stroke: var(--accent-amber); stroke-width: 2; animation: spin 15s linear infinite; opacity: 0.15;}
.ring-ticks circle { stroke: var(--line-strong); stroke-width: 2; opacity: 0.1; }
.is-auth_wait .rings { opacity: 0.7; } .is-auth_scanning .rings { opacity: 1; } .is-welcome .rings { opacity: 0.3; }
.center-text { position: absolute; z-index: 5; text-align: center; pointer-events: none; display: flex; flex-direction: column; align-items: center; gap: 12px; transition: transform 0.5s ease; }
.welcome-title { font-family: var(--font-sans, sans-serif); font-size: clamp(24px, 4vw, 36px); font-weight: 900; letter-spacing: 0.3em; color: var(--text-main); }
.sub-title { font-size: 10px; letter-spacing: 0.5em; color: var(--text-dim); text-transform: uppercase; font-weight: 700; }
.hint-tap { font-size: 12px; color: var(--accent-cyan); margin-top: 20px; letter-spacing: 2px; }
.is-auth_wait .center-text { transform: translateY(-120px); } .is-auth_scanning .center-text { transform: translateY(-120px); } .is-welcome .center-text { transform: translateY(0); }
.auth-card { position: absolute; z-index: 10; width: min(90vw, 380px); background: var(--bg-surface); border: 1px solid var(--line-mid); clip-path: polygon(0 0, 100% 0, 100% calc(100% - 20px), calc(100% - 20px) 100%, 0 100%); box-shadow: 6px 6px 0 rgba(0, 0, 0, 0.4), inset 0 0 0 1px rgba(255,255,255,0.02); display: flex; flex-direction: column; }
.auth-head { background: var(--bg-elevated); color: var(--accent-cyan); border-bottom: 1px solid var(--line-mid); padding: 8px 16px; font-size: 10px; letter-spacing: 1px; font-weight: 700; display: flex; justify-content: space-between; }
.auth-body { padding: 20px; display: flex; flex-direction: column; gap: 16px; }
.auth-row { display: flex; justify-content: space-between; font-size: 12px; border-bottom: 1px dashed var(--line-faint); padding-bottom: 8px; }
.a-key { color: var(--text-dim); font-weight: 700; } .a-val { color: var(--text-main); font-weight: 900; letter-spacing: 1px; }
.auth-log { height: 70px; font-size: 10px; color: var(--accent-cyan); font-weight: 700; overflow: hidden; display: flex; flex-direction: column; justify-content: flex-end; gap: 4px; }
.log-line { animation: typeIn 0.3s ease-out; }
@keyframes typeIn { from { opacity: 0; transform: translateX(-10px); } to { opacity: 1; transform: translateX(0); } }
.auth-bar { height: 2px; background: var(--line-faint); width: 100%; margin-top: 4px; }
.bar-fill { height: 100%; background: var(--accent-cyan); transition: width 0.1s linear; box-shadow: 0 0 8px var(--accent-cyan-soft); }
.auth-btn { margin-top: 10px; padding: 12px; background: transparent; border: 1px solid var(--line-strong); color: var(--text-main); font-family: var(--font-mono, monospace); font-size: 12px; font-weight: 700; cursor: pointer; transition: all 0.2s; letter-spacing: 2px; }
.auth-btn:hover:not(.disabled) { background: var(--line-mid); }
.auth-btn.disabled { border-color: var(--line-faint); color: var(--text-faint); cursor: not-allowed; }
.auth-btn.success { border-color: var(--accent-cyan); color: var(--accent-cyan); background: rgba(44, 211, 225, 0.05); }
.auth-btn.success:hover { background: var(--accent-cyan-soft); }
.pop-enter-active { animation: popIn 0.4s cubic-bezier(0.16, 1, 0.3, 1); }
.pop-leave-active { animation: popOut 0.3s ease; }
@keyframes popIn { from { opacity: 0; transform: scale(0.95) translateY(20px); } to { opacity: 1; transform: scale(1) translateY(0); } }
@keyframes popOut { from { opacity: 1; transform: scale(1); } to { opacity: 0; transform: scale(0.95); } }
@keyframes spin { 100% { transform: rotate(360deg); } }
@keyframes spinRev { 100% { transform: rotate(-360deg); } }
</style>

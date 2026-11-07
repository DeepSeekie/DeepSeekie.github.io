<template>
  <div id="splash" :class="{ 'is-leaving': leaving }" @pointerdown="onTap" @keydown.enter="onTap">
    
    <!-- 1. 工业图纸背景与网格 -->
    <div class="bg-grid"></div>
    <div class="bg-halftone"></div>
    
    <!-- 背景流动数据流 (淡青色细线) -->
    <div class="bg-data-stream">
      <div class="stream-line" v-for="i in 5" :key="i" :style="{ top: `${i * 20}%`, animationDelay: `${i * -2}s` }"></div>
    </div>

    <!-- 2. HUD: 左上角系统信息 -->
    <div class="hud-corner hud-tl">
      <div class="hl-title">PRTS // SYS_OS</div>
      <div class="hl-data">NODE: RX-01</div>
      <div class="hl-data">CPU: RHODES_QUANTUM_V4</div>
      <div class="hl-data">MEM: 128.0 TB</div>
      <div class="hl-data" :class="{ 'blink': phase === 'boot' }">STATUS: <span class="text-cyan">{{ systemStatus }}</span></div>
    </div>

    <!-- HUD: 右上角状态 -->
    <div class="hud-corner hud-tr">
      <div class="hl-data">CLEARANCE: LEVEL_A</div>
      <div class="hl-data">LINK: <span class="text-cyan">STABLE</span></div>
    </div>

    <!-- 3. HUD: 左下角版本号 (已去除音频条) -->
    <div class="hud-corner hud-bl">
      <div class="hl-data">VER 0.1.0 // BUILD 2024.05</div>
      <div class="hl-data text-faint">[ 01 / 04 ] BOOT_SEQUENCE</div>
    </div>

    <!-- HUD: 右下角Powered by -->
    <div class="hud-corner hud-br">
      <div class="hl-data text-faint">POWERED BY</div>
      <div class="hl-title" style="letter-spacing: 2px;">RHODES ISLAND</div>
      <div class="hl-data text-cyan mt-1" :class="{ 'blink': phase === 'welcome' }">ACCESS_GRANTED_</div>
    </div>

    <!-- 底部物理进度条 -->
    <div class="bottom-progress">
      <div class="progress-bar" :style="{ width: progressWidth }"></div>
    </div>

    <!-- ========================================== -->
    <!-- 核心视觉区 -->
    <!-- ========================================== -->
    <div class="splash-stage" :class="'is-' + phase">
      
      <!-- 巨大的底层罗盘/圆环 -->
      <svg class="rings" viewBox="0 0 400 400">
        <g class="ring ring-outer"><circle cx="200" cy="200" r="180"/></g>
        <g class="ring ring-mid"><circle cx="200" cy="200" r="140"/></g>
        <g class="ring ring-inner"><circle cx="200" cy="200" r="90"/></g>
        <!-- 刻度 -->
        <g class="ring ring-ticks"><circle cx="200" cy="200" r="160" stroke-dasharray="2 10"/></g>
      </svg>

      <!-- 中央文字信息 -->
      <div class="center-text">
        <div class="welcome-title" v-if="phase === 'boot' || phase === 'system_check'">神 经 链 接 重 构</div>
        <div class="welcome-title" v-if="phase === 'auth_wait' || phase === 'auth_scanning'">等 待 权 限 验 证</div>
        <div class="welcome-title text-cyan" v-if="phase === 'welcome'">重 链 接 成 功</div>
        
        <div class="sub-title">
          {{ phase === 'welcome' ? 'NEURAL LINK RESTORED' : 'NEURAL LINK INITIALIZATION' }}
        </div>
      </div>

      <!-- ========================================== -->
      <!-- 身份验证弹窗 (亮色工业风) -->
      <!-- ========================================== -->
      <transition name="pop">
        <div class="auth-card" v-if="phase === 'auth_wait' || phase === 'auth_scanning' || phase === 'welcome'">
          <div class="auth-head">
            <div>// BIOMETRIC_CONFIRMATION</div>
            <div>[ SECURE ]</div>
          </div>
          <div class="auth-body">
            <div class="auth-row">
              <span class="a-key">ID</span>
              <span class="a-val">DOCTOR_001</span>
            </div>
            <div class="auth-row">
              <span class="a-key">STATUS</span>
              <span class="a-val" :class="{ 'text-cyan': phase === 'welcome' }">
                {{ phase === 'auth_wait' ? 'AWAITING_TAP_' : (phase === 'auth_scanning' ? 'SCANNING_' : 'VERIFIED') }}
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

            <!-- 按钮逻辑：等待点击 -> 扫描中 -> 等待点击进入 -->
            <button class="auth-btn" v-if="phase === 'auth_wait'" @click.stop="startScan">
              [ GRANT_ACCESS ]
            </button>
            <button class="auth-btn disabled" v-else-if="phase === 'auth_scanning'">
              [ SCANNING... ]
            </button>
            <button class="auth-btn success" v-else-if="phase === 'welcome'" @click.stop="enterSystem">
              [ ENTER_TERMINAL ]
            </button>
          </div>
        </div>
      </transition>

    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';

const emit = defineEmits(['done']);

const leaving = ref(false);
const phase = ref('boot');
const systemStatus = ref('INITIALIZING...');
const progressWidth = ref('10%');

// 验证相关状态
const scanPct = ref(0);
const scanLogs = ref([]);
let timers = [];

const addLog = (text) => {
  scanLogs.value.push(text);
};

const clearTimers = () => {
  timers.forEach(clearTimeout);
  timers = [];
};

// 点击开始扫描
const startScan = () => {
  if (phase.value !== 'auth_wait') return;
  phase.value = 'auth_scanning';
  systemStatus.value = 'VALIDATING...';
  
  addLog('> SCANNING_BIOMETRICS...');
  let p = 0;
  const logPool = [
    '> MATCHING_NEURAL_PATTERN...',
    '> CROSS_REFERENCING_DB...',
    '> CLEARANCE_LEVEL_A_CONFIRMED.',
    '> LINK_ESTABLISHED.'
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
        // 扫描完成，进入 welcome 状态，但绝对不自动跳转！
        phase.value = 'welcome';
        systemStatus.value = 'AUTHORIZED';
        progressWidth.value = '100%';
      }, 600);
    }
  }, 100);
};

// 点击进入主系统
const enterSystem = () => {
  if (phase.value !== 'welcome') return;
  leaving.value = true;
  setTimeout(() => emit('done'), 800); // 触发淡出并通知父组件
};

// 全局点击判定
const onTap = () => {
  if (phase.value === 'auth_wait') {
    startScan();
  } else if (phase.value === 'welcome') {
    enterSystem();
  }
};

onMounted(() => {
  // 启动系统自检
  timers.push(setTimeout(() => {
    systemStatus.value = 'SELF_TEST_PASS';
    progressWidth.value = '30%';
    phase.value = 'system_check';
  }, 1500));

  // 进入等待状态，此后将永久停留，直到用户点击
  timers.push(setTimeout(() => {
    systemStatus.value = 'AWAITING_INPUT';
    progressWidth.value = '50%';
    phase.value = 'auth_wait';
  }, 3000));
});

onUnmounted(() => clearTimers());
</script>

<style scoped>
/* ============================================================
   基础层：亮色纸感
   ============================================================ */
#splash {
  position: fixed; inset: 0; z-index: 100;
  background: var(--canvas);
  color: var(--fg);
  display: flex; align-items: center; justify-content: center;
  transition: opacity 0.8s ease;
  cursor: pointer; overflow: hidden;
  font-family: var(--font-mono);
}
#splash.is-leaving { opacity: 0; pointer-events: none; }

/* 1. 工业图纸网格背景 */
.bg-grid {
  position: absolute; inset: 0; z-index: 0; pointer-events: none;
  background-image: 
    linear-gradient(rgba(0, 0, 0, 0.04) 1px, transparent 1px),
    linear-gradient(90deg, rgba(0, 0, 0, 0.04) 1px, transparent 1px);
  background-size: 40px 40px;
}

/* 半调网点质感 */
.bg-halftone {
  position: absolute; inset: 0; z-index: 0; pointer-events: none;
  background-image: radial-gradient(circle, rgba(0,0,0,0.08) 0.5px, transparent 0.5px);
  background-size: 4px 4px; opacity: 0.6;
}

/* 背景横向数据流 */
.bg-data-stream { position: absolute; inset: 0; z-index: 1; overflow: hidden; pointer-events: none; }
.stream-line {
  position: absolute; left: -100%; width: 200%; height: 1px;
  background: linear-gradient(90deg, transparent, rgba(0, 152, 220, 0.3), transparent);
  animation: flowRight 6s linear infinite;
}
@keyframes flowRight { 0% { transform: translateX(0); } 100% { transform: translateX(50%); } }

/* ============================================================
   HUD 四角信息
   ============================================================ */
.hud-corner { position: absolute; z-index: 10; font-size: 10px; line-height: 1.8; color: var(--fg-mid); letter-spacing: 1px; }
.hud-tl { top: 30px; left: 30px; }
.hud-tr { top: 30px; right: 30px; text-align: right; }
.hud-bl { bottom: 30px; left: 30px; }
.hud-br { bottom: 30px; right: 30px; text-align: right; }

.hl-title { font-size: 12px; font-weight: 900; color: var(--fg); letter-spacing: 2px; margin-bottom: 4px; }
.hl-data { color: var(--fg-dim); margin-bottom: 2px; }
.text-cyan { color: var(--blue); font-weight: 700; }
.text-faint { color: #aaa; margin-top: 10px; }
.mt-1 { margin-top: 6px; }
.blink { animation: blink 1s step-end infinite; }
@keyframes blink { 0%, 100% { opacity: 1; } 50% { opacity: 0.3; } }

/* 底部总进度条 */
.bottom-progress {
  position: absolute; bottom: 0; left: 0; right: 0; height: 2px; background: rgba(0,0,0,0.1); z-index: 10;
}
.progress-bar { height: 100%; background: var(--blue); transition: width 0.8s ease; box-shadow: 0 0 10px rgba(0,152,220,0.5); }

/* ============================================================
   核心舞台 (视觉中心)
   ============================================================ */
.splash-stage {
  position: relative; width: 100vw; height: 100vh; z-index: 5;
  display: flex; align-items: center; justify-content: center;
}

/* 巨大的罗盘圆环 */
.rings {
  position: absolute; width: min(80vw, 600px); height: min(80vw, 600px); z-index: 1;
  opacity: 0.4; transition: opacity 1s ease;
}
.ring { transform-origin: 200px 200px; fill: none; }
.ring-outer circle { stroke: var(--line-strong); stroke-width: 1; stroke-dasharray: 10 20; animation: spin 40s linear infinite; opacity: 0.3;}
.ring-mid circle { stroke: var(--blue); stroke-width: 1.5; stroke-dasharray: 100 400; animation: spinRev 20s linear infinite; opacity: 0.6; }
.ring-inner circle { stroke: var(--fg); stroke-width: 2; animation: spin 15s linear infinite; opacity: 0.2;}
.ring-ticks circle { stroke: var(--line-strong); stroke-width: 2; opacity: 0.1; }

.is-auth_wait .rings { opacity: 0.8; }
.is-auth_scanning .rings { opacity: 1; }
.is-welcome .rings { opacity: 0.4; }

/* 中央大标题 */
.center-text {
  position: absolute; z-index: 5; text-align: center; pointer-events: none;
  display: flex; flex-direction: column; align-items: center; gap: 12px;
  transition: transform 0.5s ease;
}
.welcome-title {
  font-family: var(--font-sans); font-size: clamp(24px, 4vw, 36px);
  font-weight: 900; letter-spacing: 0.3em; color: var(--fg);
}
.sub-title {
  font-size: 10px; letter-spacing: 0.5em; color: var(--fg-dim); text-transform: uppercase; font-weight: 700;
}

.is-auth_wait .center-text { transform: translateY(-120px); }
.is-auth_scanning .center-text { transform: translateY(-120px); }
.is-welcome .center-text { transform: translateY(0); }

/* ============================================================
   身份验证弹窗 (亮色工业卡片)
   ============================================================ */
.auth-card {
  position: absolute; z-index: 10;
  width: min(90vw, 380px); background: var(--surface);
  border: 1px solid var(--fg);
  clip-path: polygon(0 0, 100% 0, 100% calc(100% - 20px), calc(100% - 20px) 100%, 0 100%);
  box-shadow: 6px 6px 0 rgba(0, 0, 0, 0.05);
  display: flex; flex-direction: column;
}
.auth-head {
  background: var(--fg); color: var(--surface);
  padding: 8px 16px; font-size: 10px; letter-spacing: 1px; font-weight: 700;
  display: flex; justify-content: space-between;
}
.auth-body { padding: 20px; display: flex; flex-direction: column; gap: 16px; }

.auth-row { display: flex; justify-content: space-between; font-size: 12px; border-bottom: 1px dashed var(--line); padding-bottom: 8px; }
.a-key { color: var(--fg-dim); font-weight: 700; }
.a-val { color: var(--fg); font-weight: 900; letter-spacing: 1px; }

/* 终端日志输出 */
.auth-log {
  height: 70px; font-size: 10px; color: var(--blue); font-weight: 700; overflow: hidden;
  display: flex; flex-direction: column; justify-content: flex-end; gap: 4px;
}
.log-line { animation: typeIn 0.3s ease-out; }
@keyframes typeIn { from { opacity: 0; transform: translateX(-10px); } to { opacity: 1; transform: translateX(0); } }

/* 验证进度条 */
.auth-bar { height: 2px; background: var(--line); width: 100%; margin-top: 4px; }
.bar-fill { height: 100%; background: var(--blue); transition: width 0.1s linear; }

/* 按钮 */
.auth-btn {
  margin-top: 10px; padding: 12px; background: transparent;
  border: 1px solid var(--fg); color: var(--fg); font-family: var(--font-mono); font-size: 12px; font-weight: 700;
  cursor: pointer; transition: all 0.2s; letter-spacing: 2px;
}
.auth-btn:hover:not(.disabled) { background: var(--fg); color: var(--surface); }
.auth-btn.disabled { border-color: var(--line); color: var(--fg-dim); cursor: not-allowed; }
.auth-btn.success { border-color: var(--blue); color: var(--blue); background: rgba(0, 152, 220, 0.05); }
.auth-btn.success:hover { background: var(--blue); color: var(--surface); }

/* 弹窗过渡动画 */
.pop-enter-active { animation: popIn 0.4s cubic-bezier(0.16, 1, 0.3, 1); }
.pop-leave-active { animation: popOut 0.3s ease; }
@keyframes popIn { from { opacity: 0; transform: scale(0.95) translateY(20px); } to { opacity: 1; transform: scale(1) translateY(0); } }
@keyframes popOut { from { opacity: 1; transform: scale(1); } to { opacity: 0; transform: scale(0.95); } }

@keyframes spin { 100% { transform: rotate(360deg); } }
@keyframes spinRev { 100% { transform: rotate(-360deg); } }
</style>

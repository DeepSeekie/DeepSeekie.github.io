<template>
  <div id="splash" :class="{ 'is-leaving': leaving }" @pointerdown="onTap" @keydown.enter="onTap">
    
    <!-- 1. 工业金属/网格背景 -->
    <div class="bg-metal"></div>
    <div class="bg-grid"></div>
    
    <!-- 背景流动数据流 -->
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

    <!-- 3. HUD: 左下角版本号与音频条 -->
    <div class="hud-corner hud-bl">
      <div class="audio-bars">
        <span></span><span></span><span></span><span></span>
      </div>
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
        <!-- 雷达刻度 -->
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
      <!-- 身份验证弹窗 (仅在 auth_wait 及之后出现) -->
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

            <!-- 按钮 -->
            <button class="auth-btn" v-if="phase === 'auth_wait'" @click.stop="startScan">
              [ GRANT_ACCESS ]
            </button>
            <button class="auth-btn disabled" v-else-if="phase === 'auth_scanning'">
              [ SCANNING... ]
            </button>
            <button class="auth-btn success" v-else-if="phase === 'welcome'">
              [ ACCESS_GRANTED ]
            </button>
          </div>
        </div>
      </transition>

    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

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
        phase.value = 'welcome';
        systemStatus.value = 'AUTHORIZED';
        progressWidth.value = '100%';
        setTimeout(() => {
          leaving.value = true;
          setTimeout(() => emit('done'), 800); // 淡出后通知父组件
        }, 2000); // 欢迎界面停留时间
      }, 600);
    }
  }, 100);
};

const onTap = () => {
  if (phase.value === 'auth_wait') {
    startScan();
  }
};

onMounted(() => {
  // 1. 启动自检阶段
  timers.push(setTimeout(() => {
    systemStatus.value = 'SELF_TEST_PASS';
    progressWidth.value = '30%';
    phase.value = 'system_check';
  }, 1500));

  // 2. 进入待验证阶段
  timers.push(setTimeout(() => {
    systemStatus.value = 'AWAITING_INPUT';
    progressWidth.value = '50%';
    phase.value = 'auth_wait';
  }, 3000));
});

// 组件卸载清理定时器
import { onUnmounted } from 'vue';
onUnmounted(() => clearTimers());
</script>

<style scoped>
/* ============================================================
   基础层
   ============================================================ */
#splash {
  position: fixed; inset: 0; z-index: 100;
  background: #0A0A0C; /* 极深的暗色 */
  color: #E0E0E0;
  display: flex; align-items: center; justify-content: center;
  transition: opacity 0.8s ease;
  cursor: pointer; overflow: hidden;
  font-family: var(--font-mono);
}
#splash.is-leaving { opacity: 0; pointer-events: none; }

/* 1. 工业金属背景 (通过多重渐变模拟拉丝与墙壁) */
.bg-metal {
  position: absolute; inset: 0; z-index: 0;
  background: 
    linear-gradient(90deg, rgba(255,255,255,0.02) 1px, transparent 1px),
    linear-gradient(rgba(255,255,255,0.02) 1px, transparent 1px);
  background-size: 100px 100px;
  background-color: #0d0d10;
  box-shadow: inset 0 0 200px rgba(0,0,0,0.9);
}
.bg-metal::after {
  content: ""; position: absolute; inset: 0;
  /* 模拟金属反光条纹 */
  background: repeating-linear-gradient(
    0deg,
    transparent,
    transparent 2px,
    rgba(255,255,255,0.01) 2px,
    rgba(255,255,255,0.01) 4px
  );
}

/* 极细网格 */
.bg-grid {
  position: absolute; inset: 0; z-index: 1; pointer-events: none;
  background-image: 
    linear-gradient(rgba(0, 195, 255, 0.05) 1px, transparent 1px),
    linear-gradient(90deg, rgba(0, 195, 255, 0.05) 1px, transparent 1px);
  background-size: 40px 40px;
  mask-image: radial-gradient(circle at center, black 20%, transparent 70%);
}

/* 背景横向数据流 */
.bg-data-stream { position: absolute; inset: 0; z-index: 1; overflow: hidden; pointer-events: none; }
.stream-line {
  position: absolute; left: -100%; width: 200%; height: 1px;
  background: linear-gradient(90deg, transparent, rgba(0,195,255,0.2), transparent);
  animation: flowRight 4s linear infinite;
}
@keyframes flowRight { 0% { transform: translateX(0); } 100% { transform: translateX(50%); } }

/* ============================================================
   HUD 四角信息
   ============================================================ */
.hud-corner { position: absolute; z-index: 10; font-size: 10px; line-height: 1.8; color: var(--fg-dim); letter-spacing: 1px; }
.hud-tl { top: 30px; left: 30px; }
.hud-tr { top: 30px; right: 30px; text-align: right; }
.hud-bl { bottom: 30px; left: 30px; }
.hud-br { bottom: 30px; right: 30px; text-align: right; }

.hl-title { font-size: 12px; font-weight: 700; color: #FFF; letter-spacing: 2px; margin-bottom: 4px; }
.hl-data { color: #666; margin-bottom: 2px; }
.text-cyan { color: #00C3FF; }
.text-faint { color: #444; margin-top: 10px; }
.mt-1 { margin-top: 6px; }
.blink { animation: blink 1s step-end infinite; }
@keyframes blink { 0%, 100% { opacity: 1; } 50% { opacity: 0.3; } }

/* 音频跳动条 */
.audio-bars { display: flex; gap: 3px; align-items: flex-end; height: 16px; margin-bottom: 8px; }
.audio-bars span { display: block; width: 3px; background: #C8102E; /* 红色点缀 */ }
.audio-bars span:nth-child(1) { height: 60%; animation: bounce 1s infinite alternate; }
.audio-bars span:nth-child(2) { height: 100%; animation: bounce 1.2s infinite alternate; }
.audio-bars span:nth-child(3) { height: 40%; animation: bounce 0.8s infinite alternate; }
.audio-bars span:nth-child(4) { height: 80%; animation: bounce 1.1s infinite alternate; }
@keyframes bounce { 0% { transform: scaleY(0.3); } 100% { transform: scaleY(1); } }

/* 底部总进度条 */
.bottom-progress {
  position: absolute; bottom: 0; left: 0; right: 0; height: 2px; background: rgba(255,255,255,0.1); z-index: 10;
}
.progress-bar { height: 100%; background: #00C3FF; transition: width 1s ease; box-shadow: 0 0 10px #00C3FF; }

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
  opacity: 0.3; transition: opacity 1s ease;
}
.ring { transform-origin: 200px 200px; fill: none; }
.ring-outer circle { stroke: #333; stroke-width: 1; stroke-dasharray: 10 20; animation: spin 40s linear infinite; }
.ring-mid circle { stroke: #00C3FF; stroke-width: 1; stroke-dasharray: 100 400; animation: spinRev 20s linear infinite; opacity: 0.5; }
.ring-inner circle { stroke: #444; stroke-width: 2; animation: spin 15s linear infinite; }
.ring-ticks circle { stroke: #555; stroke-width: 2; }

/* 状态机控制圆环高亮 */
.is-auth_wait .rings { opacity: 0.8; }
.is-auth_scanning .rings { opacity: 1; }
.is-welcome .rings { opacity: 0.5; }

/* 中央大标题 */
.center-text {
  position: absolute; z-index: 5; text-align: center; pointer-events: none;
  display: flex; flex-direction: column; align-items: center; gap: 12px;
  transition: transform 0.5s ease;
}
.welcome-title {
  font-family: var(--font-sans); font-size: clamp(24px, 4vw, 36px);
  font-weight: 900; letter-spacing: 0.3em; color: #FFF;
  text-shadow: 0 0 20px rgba(0,0,0,0.8);
}
.sub-title {
  font-size: 10px; letter-spacing: 0.5em; color: #666; text-transform: uppercase;
}

/* 在验证阶段把中间的字变小并往上顶 */
.is-auth_wait .center-text { transform: translateY(-120px); }
.is-auth_scanning .center-text { transform: translateY(-120px); }
.is-welcome .center-text { transform: translateY(0); }

/* ============================================================
   身份验证弹窗 (斜切角、工业感)
   ============================================================ */
.auth-card {
  position: absolute; z-index: 10;
  width: min(90vw, 380px); background: rgba(15, 18, 20, 0.9);
  backdrop-filter: blur(10px); border: 1px solid rgba(0, 195, 255, 0.3);
  /* 模拟斜切角 */
  clip-path: polygon(0 0, 100% 0, 100% calc(100% - 20px), calc(100% - 20px) 100%, 0 100%);
  box-shadow: 0 20px 50px rgba(0,0,0,0.8), inset 0 0 0 1px rgba(0,195,255,0.1);
  display: flex; flex-direction: column;
}
.auth-head {
  background: rgba(0, 195, 255, 0.1); color: #00C3FF;
  padding: 8px 16px; font-size: 10px; letter-spacing: 1px;
  display: flex; justify-content: space-between; border-bottom: 1px solid rgba(0, 195, 255, 0.2);
}
.auth-body { padding: 20px; display: flex; flex-direction: column; gap: 16px; }

.auth-row { display: flex; justify-content: space-between; font-size: 12px; border-bottom: 1px dashed #333; padding-bottom: 8px; }
.a-key { color: #666; }
.a-val { color: #FFF; font-weight: 700; letter-spacing: 1px; }

/* 终端日志输出 */
.auth-log {
  height: 70px; font-size: 10px; color: #00C3FF; overflow: hidden;
  display: flex; flex-direction: column; justify-content: flex-end; gap: 4px;
}
.log-line { animation: typeIn 0.3s ease-out; }
@keyframes typeIn { from { opacity: 0; transform: translateX(-10px); } to { opacity: 1; transform: translateX(0); } }

/* 验证进度条 */
.auth-bar { height: 4px; background: #222; width: 100%; margin-top: 4px; }
.bar-fill { height: 100%; background: #00C3FF; transition: width 0.1s linear; box-shadow: 0 0 10px #00C3FF; }

/* 按钮 */
.auth-btn {
  margin-top: 10px; padding: 12px; background: transparent;
  border: 1px solid #FFF; color: #FFF; font-family: var(--font-mono); font-size: 12px;
  cursor: pointer; transition: all 0.2s; letter-spacing: 2px;
}
.auth-btn:hover:not(.disabled) { background: #FFF; color: #000; }
.auth-btn.disabled { border-color: #333; color: #555; cursor: not-allowed; }
.auth-btn.success { border-color: #00C3FF; color: #00C3FF; background: rgba(0,195,255,0.1); }

/* 弹窗过渡动画 */
.pop-enter-active { animation: popIn 0.4s cubic-bezier(0.16, 1, 0.3, 1); }
.pop-leave-active { animation: popOut 0.3s ease; }
@keyframes popIn { from { opacity: 0; transform: scale(0.95) translateY(20px); } to { opacity: 1; transform: scale(1) translateY(0); } }
@keyframes popOut { from { opacity: 1; transform: scale(1); } to { opacity: 0; transform: scale(0.95); } }

@keyframes spin { 100% { transform: rotate(360deg); } }
@keyframes spinRev { 100% { transform: rotate(-360deg); } }
</style>

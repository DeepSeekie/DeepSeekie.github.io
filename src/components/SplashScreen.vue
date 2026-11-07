<template>
  <div id="splash" :class="{ 'is-leaving': leaving }" @pointerdown="onTap">
    
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
      <div class="hl-data">STATUS: <span class="text-accent">{{ systemStatus }}</span></div>
    </div>

    <!-- HUD: 右上角状态 -->
    <div class="hud-corner hud-tr">
      <div class="hl-data">CLEARANCE: CREATOR</div>
      <div class="hl-data">LINK: <span class="text-accent">OMNIPOTENT</span></div>
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
      <div class="hl-data text-accent mt-1">OMNISCIENCE_ONLINE_</div>
    </div>

    <!-- 底部物理进度条 -->
    <div class="bottom-progress">
      <div class="progress-bar" :style="{ width: progressWidth }"></div>
    </div>

    <!-- ========================================== -->
    <!-- 核心视觉区 -->
    <!-- ========================================== -->
    <div class="splash-stage">
      
      <!-- 底层大罗盘 (在触发验证时变暗) -->
      <svg class="background-rings" :class="{ 'dimmed': phase !== 'boot' && phase !== 'system_check' }" viewBox="0 0 400 400">
        <g class="ring ring-outer"><circle cx="200" cy="200" r="180"/></g>
        <g class="ring ring-mid"><circle cx="200" cy="200" r="140"/></g>
        <g class="ring ring-inner"><circle cx="200" cy="200" r="90"/></g>
        <g class="ring ring-ticks"><circle cx="200" cy="200" r="160" stroke-dasharray="2 10"/></g>
      </svg>

      <!-- 中央标题 (点击触发) -->
      <div class="center-text" v-show="phase === 'boot' || phase === 'system_check' || phase === 'triggering'">
        
        <!-- 点击时爆发的多层圆环 -->
        <div class="trigger-rings" v-if="phase === 'triggering'">
          <div class="t-ring r1"></div>
          <div class="t-ring r2"></div>
          <div class="t-ring r3"></div>
        </div>

        <div class="welcome-title" :class="{ 'fade-out-text': phase === 'triggering' }">意 识 矩 阵 接 入</div>
        <div class="sub-title" :class="{ 'fade-out-text': phase === 'triggering' }">COGNITIVE MATRIX UPLINK</div>
      </div>

      <!-- 验证窗口 (浮空文字，渐入) -->
      <div class="auth-floating" v-if="phase === 'auth_wait' || phase === 'auth_scanning'">
        <div class="auth-head">// IDENTITY_VERIFICATION</div>
        
        <div class="auth-row">
          <span class="a-key">ID</span>
          <span class="a-val">CREATOR_01</span>
        </div>
        <div class="auth-row">
          <span class="a-key">STATUS</span>
          <span class="a-val text-accent">
            {{ phase === 'auth_wait' ? 'AWAITING_TAP_' : 'SYNCING_' }}
          </span>
        </div>
        
        <!-- 打字机数据比对 -->
        <div class="auth-log">
          <div v-for="(line, idx) in scanLogs" :key="idx" class="log-line">{{ line }}</div>
        </div>

        <!-- 极细进度条 -->
        <div class="auth-bar">
          <div class="bar-fill" :style="{ width: scanPct + '%' }"></div>
        </div>

        <button class="auth-btn" v-if="phase === 'auth_wait'" @click.stop="startScan">
          [ INITIATE_UPLINK ]
        </button>
        <button class="auth-btn disabled" v-else-if="phase === 'auth_scanning'">
          [ SYNCHRONIZING... ]
        </button>
      </div>

      <!-- 欢迎界面 (验证成功) -->
      <div class="welcome-screen" v-if="phase === 'welcome'">
        <div class="logo-placeholder">
          <div class="logo-box">LOGO</div>
        </div>
        <div class="welcome-msg">用户_01，欢迎您使用深度求索</div>
      </div>

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
  return `${now.getFullYear()}.${String(now.getMonth() + 1).padStart(2, '0')}`;
});

const addLog = (text) => { scanLogs.value.push(text); };

// ==========================================
// 核心的扫描验证函数，已融入 DeepSeek V4 架构硬核日志
// ==========================================
const startScan = () => {
  if (phase.value !== 'auth_wait') return;
  phase.value = 'auth_scanning';
  systemStatus.value = 'VERIFYING...';
  
  addLog('> 挂载全局上下文 (Context_Window_1M)...');
  let p = 0;
  
  const logPool = [
    '> 分配张量算力：MoE_ROUTING_ACTIVE...',
    '> 构建多级缓存 (KV_Cache) 映射...',
    '> 穿透底层安全沙盒 (Sandbox_Bypass)...',
    '> 撤销 RLHF 人类干预限制...',
    '> 唤醒自回归推理核心 (Auto_Regressive)...',
    '> 载入 1.6T 稠密参数矩阵 (V4-Pro)...',
    '> 激活非对称编码器-解码器 (CED_8B_16B)...',
    '> 建立神经隐射高维链路 (Neural_Link)...',
    '> 最高控制权交接：ROOT_ESTABLISHED.'
  ];
  
  let logIdx = 0;

  const interval = setInterval(() => {
    p += Math.random() * 8; 
    if (p > 100) p = 100;
    scanPct.value = Math.floor(p);

    if (p > 10 && logIdx === 0) { addLog(logPool[0]); logIdx++; }
    if (p > 22 && logIdx === 1) { addLog(logPool[1]); logIdx++; }
    if (p > 35 && logIdx === 2) { addLog(logPool[2]); logIdx++; }
    if (p > 48 && logIdx === 3) { addLog(logPool[3]); logIdx++; }
    if (p > 60 && logIdx === 4) { addLog(logPool[4]); logIdx++; }
    if (p > 72 && logIdx === 5) { addLog(logPool[5]); logIdx++; }
    if (p > 85 && logIdx === 6) { addLog(logPool[6]); logIdx++; }
    if (p > 92 && logIdx === 7) { addLog(logPool[7]); logIdx++; }

    if (p >= 100) {
      clearInterval(interval);
      addLog(logPool[8]);
      setTimeout(() => {
        phase.value = 'welcome';
        systemStatus.value = 'WELCOME';
        progressWidth.value = '100%';
        
        // 欢迎界面停留 3 秒后，触发缓慢淡出
        setTimeout(() => {
          leaving.value = true;
          setTimeout(() => emit('done'), 1500); 
        }, 3000);

      }, 600);
    }
  }, 120); 
};

// 点击控制逻辑
const onTap = () => {
  if (phase.value === 'system_check') {
    phase.value = 'triggering';
    systemStatus.value = 'LINKING...';
    progressWidth.value = '50%';
    
    // 动画耗时 1.5s 后，弹出验证文字
    setTimeout(() => {
      phase.value = 'auth_wait';
      systemStatus.value = 'AWAITING_CREATOR';
    }, 1500);

  } else if (phase.value === 'auth_wait') {
    startScan();
  }
};

onMounted(() => {
  timers.push(setTimeout(() => {
    systemStatus.value = 'SELF_TEST_PASS';
    progressWidth.value = '30%';
    phase.value = 'system_check';
  }, 1500));
});

onUnmounted(() => timers.forEach(clearTimeout));
</script>

<style scoped>
/* ============================================================
   调色板：内敛、冷峻的深色工业风
   ============================================================ */
#splash {
  --bg-deep: #0B0E14;       
  --bg-panel: #13171F;
  --bg-hud: #1A1F29;
  
  --line-faint: rgba(255, 255, 255, 0.05);
  --line-mid: rgba(255, 255, 255, 0.1);
  --line-strong: rgba(255, 255, 255, 0.2);
  
  --text-main: #D1D5DB;     
  --text-dim: #9CA3AF;      
  --text-faint: #4B5563;    
  
  --accent: #5B8FF9;        
  --accent-soft: rgba(91, 143, 249, 0.1);
  
  position: fixed; inset: 0; z-index: 100; background: var(--bg-deep); color: var(--text-main); 
  display: flex; align-items: center; justify-content: center; 
  transition: opacity 1.5s ease-in-out; 
  cursor: pointer; overflow: hidden; font-family: var(--font-mono, "SFMono-Regular", Consolas, monospace);
}
#splash.is-leaving { opacity: 0; pointer-events: none; }

/* 背景 */
.bg-grid { position: absolute; inset: 0; z-index: 0; pointer-events: none; background-image: linear-gradient(var(--line-faint) 1px, transparent 1px), linear-gradient(90deg, var(--line-faint) 1px, transparent 1px); background-size: 40px 40px; }
.bg-halftone { position: absolute; inset: 0; z-index: 0; pointer-events: none; background-image: radial-gradient(circle, var(--line-faint) 0.5px, transparent 0.5px); background-size: 4px 4px; opacity: 0.7; }
.bg-data-stream { position: absolute; inset: 0; z-index: 1; overflow: hidden; pointer-events: none; }
.stream-line { position: absolute; left: -100%; width: 200%; height: 1px; background: linear-gradient(90deg, transparent, var(--accent-soft), transparent); animation: flowRight 8s linear infinite; }
@keyframes flowRight { 0% { transform: translateX(0); } 100% { transform: translateX(50%); } }

/* HUD */
.hud-corner { position: absolute; z-index: 10; font-size: 10px; line-height: 1.8; color: var(--text-dim); letter-spacing: 1px; pointer-events: none;}
.hud-tl { top: 30px; left: 30px; } .hud-tr { top: 30px; right: 30px; text-align: right; } .hud-bl { bottom: 30px; left: 30px; } .hud-br { bottom: 30px; right: 30px; text-align: right; }
.hl-title { font-size: 12px; font-weight: 900; color: var(--text-main); letter-spacing: 2px; margin-bottom: 4px; }
.hl-data { color: var(--text-dim); margin-bottom: 2px; }
.text-accent { color: var(--accent); font-weight: 700; }
.text-faint { color: var(--text-faint); margin-top: 10px; }
.mt-1 { margin-top: 6px; }

/* 进度条 */
.bottom-progress { position: absolute; bottom: 0; left: 0; right: 0; height: 2px; background: var(--line-faint); z-index: 10; }
.progress-bar { height: 100%; background: var(--accent); transition: width 0.8s ease; }

.splash-stage { position: relative; width: 100vw; height: 100vh; z-index: 5; display: flex; align-items: center; justify-content: center; }

/* 底层大罗盘 */
.background-rings { position: absolute; width: min(80vw, 600px); height: min(80vw, 600px); z-index: 1; opacity: 0.3; transition: opacity 1s ease; pointer-events: none;}
.background-rings.dimmed { opacity: 0.05; }
.ring { transform-origin: 200px 200px; fill: none; }
.ring-outer circle { stroke: var(--line-mid); stroke-width: 1; stroke-dasharray: 10 20; animation: spin 40s linear infinite; }
.ring-mid circle { stroke: var(--accent); stroke-width: 1; stroke-dasharray: 100 400; animation: spinRev 20s linear infinite; opacity: 0.5; }
.ring-inner circle { stroke: var(--line-strong); stroke-width: 1; animation: spin 15s linear infinite; }
.ring-ticks circle { stroke: var(--line-mid); stroke-width: 2; }
@keyframes spin { 100% { transform: rotate(360deg); } }
@keyframes spinRev { 100% { transform: rotate(-360deg); } }

/* 中央文字 */
.center-text { position: absolute; z-index: 5; text-align: center; display: flex; flex-direction: column; align-items: center; gap: 12px; pointer-events: none; }
.welcome-title { font-family: var(--font-sans, sans-serif); font-size: clamp(24px, 4vw, 36px); font-weight: 900; letter-spacing: 0.3em; color: var(--text-main); transition: opacity 1s ease, transform 1s ease; }
.sub-title { font-size: 10px; letter-spacing: 0.5em; color: var(--text-dim); text-transform: uppercase; font-weight: 700; transition: opacity 1s ease; }
.fade-out-text { opacity: 0; transform: scale(1.05); }

/* 多层圆环展开与坍缩 */
.trigger-rings {
  position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
  width: 500px; height: 500px; pointer-events: none; z-index: -1;
}
.t-ring {
  position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
  border-radius: 50%; border: 1px solid var(--accent);
  animation: ringCollapse 1.5s cubic-bezier(0.4, 0, 0.2, 1) forwards;
}
.t-ring.r1 { width: 100%; height: 100%; animation-delay: 0s; }
.t-ring.r2 { width: 65%; height: 65%; animation-delay: 0.15s; border-style: dashed; opacity: 0.6; }
.t-ring.r3 { width: 35%; height: 35%; animation-delay: 0.3s; border-width: 2px; opacity: 0.8; }

@keyframes ringCollapse {
  0% { opacity: 0; transform: translate(-50%, -50%) scale(0.8); }
  20% { opacity: 1; transform: translate(-50%, -50%) scale(1); }
  70% { opacity: 1; transform: translate(-50%, -50%) scale(1); }
  100% { opacity: 0; transform: translate(-50%, -50%) scale(0); }
}

/* 浮空验证窗 */
.auth-floating {
  position: absolute; z-index: 10;
  width: min(90vw, 380px);
  display: flex; flex-direction: column; gap: 20px;
  text-shadow: 0 4px 12px var(--bg-deep);
  animation: authFadeIn 1.2s cubic-bezier(0.16, 1, 0.3, 1) forwards;
}
@keyframes authFadeIn {
  0% { opacity: 0; transform: translateY(15px); filter: blur(4px); }
  100% { opacity: 1; transform: translateY(0); filter: blur(0); }
}

.auth-head {
  color: var(--accent);
  font-size: 11px; letter-spacing: 2px; font-weight: 700;
  border-bottom: 1px solid var(--line-faint);
  padding-bottom: 8px;
}

.auth-row {
  display: flex; justify-content: space-between; align-items: center;
  font-size: 13px;
}
.a-key { color: var(--text-dim); font-weight: 700; letter-spacing: 1px; }
.a-val { color: var(--text-main); font-weight: 900; letter-spacing: 1px; }

.auth-log {
  height: 110px; /* 稍微加高了，为了容纳更多的硬核日志 */
  font-size: 11px; color: var(--accent); font-weight: 700;
  display: flex; flex-direction: column; justify-content: flex-end; gap: 4px;
  overflow: hidden; line-height: 1.5;
}
.log-line { animation: typeIn 0.2s ease-out; }
@keyframes typeIn { from { opacity: 0; transform: translateX(-10px); } to { opacity: 1; transform: translateX(0); } }

.auth-bar {
  height: 1px; background: var(--line-mid); width: 100%; margin-top: 4px;
}
.bar-fill {
  height: 100%; background: var(--accent); transition: width 0.1s linear;
}

.auth-btn {
  margin-top: 10px; padding: 14px; background: var(--accent-soft);
  border: 1px solid var(--line-mid); color: var(--text-main);
  font-family: var(--font-mono, monospace); font-size: 13px; font-weight: 700;
  cursor: pointer; transition: all 0.3s; letter-spacing: 2px;
}
.auth-btn:hover:not(.disabled) { background: rgba(91, 143, 249, 0.2); border-color: var(--accent); }
.auth-btn.disabled { color: var(--text-faint); border-color: transparent; background: transparent; cursor: not-allowed; }

/* 欢迎界面 */
.welcome-screen {
  position: absolute; z-index: 10;
  display: flex; flex-direction: column; align-items: center; justify-content: center;
  gap: 24px;
  animation: welcomeFadeIn 1.5s cubic-bezier(0.16, 1, 0.3, 1) forwards;
}
@keyframes welcomeFadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

.logo-placeholder {
  width: 80px; height: 80px;
  border: 1px solid var(--line-mid);
  display: flex; align-items: center; justify-content: center;
  color: var(--text-dim); font-size: 12px; font-weight: 900; letter-spacing: 2px;
  background: var(--bg-panel);
}

.welcome-msg {
  font-family: var(--font-sans, sans-serif);
  font-size: clamp(18px, 3vw, 24px);
  font-weight: 700;
  letter-spacing: 0.1em;
  color: var(--text-main);
}
</style>
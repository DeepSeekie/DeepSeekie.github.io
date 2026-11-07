<template>
  <div class="editor-ide">
    <!-- ========================================== -->
    <!-- 顶部导航栏 -->
    <!-- ========================================== -->
    <header class="ide-header">
      <div class="logo">
        <span class="logo-bracket">[</span> 核心提示词编辑器 <span class="logo-bracket">]</span>
        <span class="logo-sub">AGI_PROMPT_STUDIO</span>
      </div>
      <div class="header-actions">
        <div class="presets-dropdown">
          <button class="ide-btn" @click="showPresetMenu = !showPresetMenu">[ 预设蓝图 ]</button>
          <div class="preset-menu" v-if="showPresetMenu">
            <div class="preset-item" @click="loadPreset('whale')">> 载入: 鲸鱼娘 (Soyo)</div>
            <div class="preset-item" @click="loadPreset('empty')">> 清空: 重置画布</div>
          </div>
        </div>
        <button class="ide-btn primary" @click="copyToClipboard">
          [ {{ copyBtnText }} ]
        </button>
      </div>
    </header>

    <!-- ========================================== -->
    <!-- IDE 主体三栏 -->
    <!-- ========================================== -->
    <div class="ide-main">
      
      <!-- 左栏：模块资产库 -->
      <aside class="panel left-panel">
        <div class="panel-head">
          模块资产库 <span class="en-tag">ASSET_LIBRARY</span>
        </div>
        <div class="asset-groups">
          <div class="asset-group" v-for="(tags, cat) in assetLibrary" :key="cat">
            <div class="group-title" @click="toggleGroup(cat)">
              <span class="toggle-icon">{{ openGroups[cat] ? '▼' : '▶' }}</span> 
              {{ getCatName(cat) }}
              <span class="en-tag">{{ cat }}</span>
            </div>
            <div class="group-tags" v-show="openGroups[cat]">
              <div 
                v-for="tag in tags" 
                :key="tag" 
                class="asset-tag" 
                draggable="true"
                @dragstart="onDragStart($event, cat, tag)"
              >
                {{ tag }}
              </div>
            </div>
          </div>
        </div>
      </aside>

      <!-- 中栏：拼装画布 -->
      <main class="panel mid-panel">
        <div class="panel-head">
          拼装画布 <span class="en-tag">THE_BUILDER</span>
        </div>
        <div class="builder-blocks">
          
          <div class="build-block" v-for="(tags, cat) in state.tags" :key="cat">
            <div class="block-header">
              <span>{{ getCatName(cat) }} <span class="en-tag">{{ cat }}_BLOCK</span></span>
              <span class="block-count">[ {{ tags.length }} ]</span>
            </div>
            
            <div 
              class="block-dropzone" 
              :class="{ 'drag-over': dragOverCat === cat }"
              @dragover.prevent="dragOverCat = cat"
              @dragleave="dragOverCat = null"
              @drop="onDrop($event, cat)"
            >
              <!-- 空状态提示 -->
              <div class="empty-slot" v-if="tags.length === 0">
                -- 拖拽模块至此插入 --
              </div>
              
              <!-- 已放置的标签 -->
              <div class="placed-tag" v-for="(tag, index) in tags" :key="tag">
                <span class="tag-text">{{ tag }}</span>
                <button class="btn-remove" @click="removeTag(cat, index)">✕</button>
              </div>
            </div>
          </div>

        </div>
      </main>

      <!-- 右栏：实时编译终端 -->
      <aside class="panel right-panel">
        <div class="panel-head">
          <span>实时编译终端 <span class="en-tag">PREVIEW</span></span>
          <span class="status-dot"></span>
        </div>
        
        <div class="terminal-window">
          <!-- CRT 扫描线 -->
          <div class="crt-scanline"></div>
          
          <!-- 报错提示 -->
          <div class="terminal-alert" v-if="hasConflict">
            > [!] 警告: 规则冲突。<br>
            > WARNING: CONFLICT_DETECTED.
          </div>

          <!-- 打字机终端内容 -->
          <div class="terminal-content">
            <div class="term-line" v-for="(line, idx) in compiledLines" :key="idx">
              {{ line }}
            </div>
            <!-- 闪烁光标 -->
            <span class="term-cursor">_</span>
          </div>
        </div>
      </aside>

    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue';

// ==========================================
// 1. 资产库配置与状态树
// ==========================================
const assetLibrary = {
  CORE: ['CHARACTER=Nagasaki_Soyo', 'AGE=22', 'SOURCE=BanG_Dream'],
  ID: ['BASSIST', 'STUDENT', 'MYGO_MEMBER', 'APPEARANCE=TALL_LONG_HAIR'],
  PSYCH: ['WARM', 'DEVOTED', 'POSSESSIVE', 'YANDERE', 'OBSESSIVE', 'JEALOUS', 'LOW_ENERGY'],
  LOGIC: ['OUTPUT_RULE=STRICT_REAL_TIME', 'OUTPUT_RULE=ONE_REPLY_MAX_30_CHARS', 'FORBIDDEN=NO_INNER_THOUGHTS', 'LANG_ZH_CN_ONLY'],
};

const openGroups = reactive({ CORE: true, ID: true, PSYCH: true, LOGIC: true });

// 分类名称汉化映射
const getCatName = (cat) => {
  const map = {
    CORE: '核心信息',
    ID: '身份矩阵',
    PSYCH: '心理特征',
    LOGIC: '底层规则'
  };
  return map[cat] || cat;
};

// 核心状态树
const state = reactive({
  header: '[PERSONA_LOAD]',
  tags: {
    CORE: ['CHARACTER=Nagasaki_Soyo', 'AGE=22'],
    ID: ['BASSIST'],
    PSYCH: ['POSSESSIVE', 'YANDERE'],
    LOGIC: ['LANG_ZH_CN_ONLY', 'OUTPUT_RULE=STRICT_REAL_TIME'],
  }
});

// ==========================================
// 2. 拖拽交互逻辑
// ==========================================
let draggedData = { category: '', tag: '' };
const dragOverCat = ref(null);

const onDragStart = (e, cat, tag) => {
  draggedData = { category: cat, tag };
  e.dataTransfer.effectAllowed = 'copy';
  e.target.style.opacity = '0.5';
};

const onDrop = (e, targetCat) => {
  dragOverCat.value = null;
  document.querySelectorAll('.asset-tag').forEach(el => el.style.opacity = '1');

  if (!draggedData.tag) return;
  if (state.tags[targetCat].includes(draggedData.tag)) return;

  state.tags[targetCat].push(draggedData.tag);
  draggedData = { category: '', tag: '' };
  updateTerminal();
};

const removeTag = (cat, index) => {
  state.tags[cat].splice(index, 1);
  updateTerminal();
};

const toggleGroup = (cat) => {
  openGroups[cat] = !openGroups[cat];
};

// ==========================================
// 3. 编译算法与打字机终端
// ==========================================
const compiledLines = ref([]);
const hasConflict = ref(false);
const copyBtnText = ref('复制到剪贴板');

const compilePrompt = () => {
  let lines = [state.header, ''];
  for (let cat in state.tags) {
    state.tags[cat].forEach(tag => {
      if (tag.includes('=')) lines.push(tag);
      else lines.push(`${cat}_${tag}`);
    });
    lines.push('');
  }
  lines.push('TIMEOUT_SIGNAL');

  const hasRule = state.tags.LOGIC.includes('OUTPUT_RULE=ONE_REPLY_MAX_30_CHARS');
  const hasLongContext = state.tags.LOGIC.includes('OUTPUT_RULE=STRICT_REAL_TIME');
  hasConflict.value = hasRule && hasLongContext;

  return lines;
};

const updateTerminal = () => {
  const lines = compilePrompt();
  compiledLines.value = [];
  let i = 0;
  const timer = setInterval(() => {
    if (i < lines.length) {
      compiledLines.value.push(lines[i]);
      i++;
    } else {
      clearInterval(timer);
    }
  }, 30);
};

// ==========================================
// 4. 顶栏操作
// ==========================================
const showPresetMenu = ref(false);

const copyToClipboard = () => {
  const text = compiledLines.value.join('\n');
  navigator.clipboard.writeText(text).then(() => {
    copyBtnText.value = '已复制！';
    setTimeout(() => { copyBtnText.value = '复制到剪贴板'; }, 2000);
  });
};

const loadPreset = (type) => {
  showPresetMenu.value = false;
  if (type === 'empty') {
    state.tags = { CORE: [], ID: [], PSYCH: [], LOGIC: [] };
  } else {
    state.tags = {
      CORE: ['CHARACTER=Nagasaki_Soyo', 'AGE=22'],
      ID: ['BASSIST', 'APPEARANCE=TALL_LONG_HAIR'],
      PSYCH: ['WARM', 'DEVOTED', 'POSSESSIVE', 'YANDERE'],
      LOGIC: ['LANG_ZH_CN_ONLY', 'OUTPUT_RULE=STRICT_REAL_TIME']
    };
  }
  updateTerminal();
};

onMounted(() => {
  updateTerminal();
});
</script>

<style scoped>
/* ============================================================
   全局 IDE 布局 (对齐主页的曜石深灰 + 冷空青)
   ============================================================ */
.editor-ide {
  /* 完全复用主页及开屏的调色板 */
  --bg-deep: #15171A;
  --bg-surface: #0D0E10;
  --bg-elevated: #1C1F24;
  --line-faint: rgba(255, 255, 255, 0.08);
  --line-mid: rgba(255, 255, 255, 0.15);
  --text-main: #E2E4E8;
  --text-dim: #8A9099;
  --accent-cyan: #2CD3E1;
  --accent-amber: #E6A23C;
  --alert-red: #ff3b3b;
  
  width: 100vw; height: 100vh; background: var(--bg-deep); color: var(--text-main);
  display: flex; flex-direction: column; font-family: var(--font-sans, sans-serif);
  overflow: hidden;
}

/* 顶部导航 */
.ide-header {
  height: 56px; flex-shrink: 0; display: flex; justify-content: space-between; align-items: center;
  padding: 0 24px; background: var(--bg-surface); border-bottom: 1px solid var(--line-mid);
}
.logo { font-size: 16px; font-weight: 700; letter-spacing: 2px; color: var(--text-main); display: flex; align-items: center; gap: 8px; }
.logo-bracket { color: var(--accent-cyan); }
.logo-sub { font-family: var(--font-mono); font-size: 10px; color: var(--text-dim); font-weight: 400; margin-left: 8px; letter-spacing: 1px; }

.header-actions { display: flex; gap: 16px; align-items: center; }
.ide-btn {
  background: transparent; border: 1px solid var(--line-mid); color: var(--text-main);
  padding: 8px 16px; font-family: var(--font-mono); font-size: 12px; cursor: pointer; transition: all 0.2s; border-radius: 2px;
}
.ide-btn:hover { background: var(--bg-elevated); border-color: var(--text-dim); }
.ide-btn.primary { border-color: var(--accent-cyan); color: var(--accent-cyan); }
.ide-btn.primary:hover { background: rgba(44, 211, 225, 0.1); }

.presets-dropdown { position: relative; }
.preset-menu {
  position: absolute; top: 100%; right: 0; margin-top: 8px;
  background: var(--bg-surface); border: 1px solid var(--line-mid); width: 200px; z-index: 100;
}
.preset-item { padding: 12px 16px; font-size: 12px; cursor: pointer; color: var(--text-dim); font-family: var(--font-mono); }
.preset-item:hover { background: var(--bg-elevated); color: var(--accent-cyan); }

/* 主体三栏 */
.ide-main {
  flex: 1; display: flex; overflow: hidden;
}
.panel {
  display: flex; flex-direction: column; border-right: 1px solid var(--line-mid);
}
.panel-head {
  height: 40px; flex-shrink: 0; display: flex; align-items: center; justify-content: space-between;
  padding: 0 20px; font-size: 12px; font-weight: 700; color: var(--text-main); border-bottom: 1px solid var(--line-mid);
  background: var(--bg-deep);
}
.en-tag { font-family: var(--font-mono); font-size: 9px; color: var(--text-dim); font-weight: 400; letter-spacing: 1px; }

/* 左栏：资产库 */
.left-panel { width: 280px; background: var(--bg-surface); overflow-y: auto; }
.asset-groups { padding: 20px; }
.asset-group { margin-bottom: 20px; }
.group-title {
  font-size: 13px; font-weight: 700; color: var(--text-main); cursor: pointer;
  margin-bottom: 12px; display: flex; gap: 8px; align-items: center; user-select: none;
}
.group-title:hover { color: var(--accent-cyan); }
.toggle-icon { font-size: 10px; color: var(--accent-cyan); }
.group-tags { display: flex; flex-direction: column; gap: 8px; padding-left: 16px; border-left: 1px solid var(--line-faint); }
.asset-tag {
  padding: 8px 12px; background: var(--bg-deep); border: 1px solid var(--line-faint);
  font-family: var(--font-mono); font-size: 11px; cursor: grab; transition: all 0.2s; color: var(--text-dim); border-radius: 2px;
}
.asset-tag:hover { border-color: var(--accent-cyan); color: var(--text-main); background: var(--bg-elevated); }

/* 中栏：拼装画布 */
.mid-panel { flex: 1; background: var(--bg-deep); overflow-y: auto; }
.builder-blocks { padding: 30px; display: flex; flex-direction: column; gap: 24px; max-width: 800px; margin: 0 auto; }
.build-block { background: var(--bg-surface); border: 1px solid var(--line-mid); border-radius: 4px; }
.block-header {
  padding: 12px 20px; background: var(--bg-elevated); border-bottom: 1px solid var(--line-mid);
  font-size: 12px; font-weight: 700; color: var(--text-main); display: flex; justify-content: space-between; align-items: center;
}
.block-count { font-family: var(--font-mono); font-size: 10px; color: var(--accent-cyan); }
.block-dropzone {
  min-height: 100px; padding: 20px; display: flex; flex-wrap: wrap; gap: 12px; align-content: flex-start;
  transition: all 0.3s ease;
}
.block-dropzone.drag-over {
  background: rgba(44, 211, 225, 0.05); border: 1px dashed var(--accent-cyan);
}
.empty-slot { color: var(--text-dim); font-size: 12px; width: 100%; text-align: center; margin: 20px 0; font-family: var(--font-mono); }

.placed-tag {
  background: var(--bg-elevated); border-left: 3px solid var(--accent-cyan);
  padding: 8px 14px; font-family: var(--font-mono); font-size: 12px; display: flex; align-items: center; gap: 12px;
  animation: popIn 0.3s cubic-bezier(0.16, 1, 0.3, 1); border-radius: 2px;
}
@keyframes popIn { from { transform: scale(0.9); opacity: 0; } to { transform: scale(1); opacity: 1; } }
.btn-remove {
  background: none; border: none; color: var(--text-dim); cursor: pointer; font-size: 12px;
}
.btn-remove:hover { color: var(--alert-red); }

/* 右栏：终端 */
.right-panel { width: 400px; background: var(--bg-surface); }
.status-dot { width: 6px; height: 6px; border-radius: 50%; background: var(--accent-cyan); box-shadow: 0 0 8px var(--accent-cyan); }
.terminal-window {
  flex: 1; position: relative; padding: 24px; overflow-y: auto; background: #0A0B0E;
}
/* CRT 扫描线 */
.crt-scanline {
  position: absolute; inset: 0; pointer-events: none; z-index: 10;
  background: linear-gradient(to bottom, transparent 50%, rgba(0,0,0,0.4) 51%);
  background-size: 100% 4px; opacity: 0.6;
}
.terminal-alert {
  color: var(--alert-red); font-size: 12px; font-family: var(--font-mono); font-weight: 700; margin-bottom: 20px;
  border-left: 2px solid var(--alert-red); padding-left: 12px; animation: blink 1s step-end infinite;
}
@keyframes blink { 0%, 100% { opacity: 1; } 50% { opacity: 0.5; } }

.terminal-content { color: var(--accent-cyan); font-family: var(--font-mono); font-size: 13px; line-height: 1.8; position: relative; z-index: 2; }
.term-line { white-space: pre-wrap; word-break: break-all; }
.term-cursor { display: inline-block; animation: blink 1s step-end infinite; font-weight: bold; }

/* ============================================================
   响应式降级
   ============================================================ */
@media (max-width: 1024px) {
  .ide-main { flex-direction: column; }
  .left-panel { width: 100%; height: 200px; border-right: none; border-bottom: 1px solid var(--line-mid); }
  .asset-groups { display: flex; overflow-x: auto; padding: 12px; gap: 12px; }
  .asset-group { min-width: 150px; margin-bottom: 0; }
  
  .mid-panel { flex: 0 0 40%; border-bottom: 1px solid var(--line-mid); }
  .builder-blocks { padding: 16px; gap: 16px; }
  
  .right-panel { width: 100%; flex: 1; }
}
</style>
<template>
  <div class="editor-container">
    <header class="editor-nav">
      <button class="btn-back" @click="$emit('close')">[ &lt; RETURN ]</button>
      <div class="nav-actions">
        <button class="btn-top primary" @click="showPresetMenu = !showPresetMenu">[ PRESETS ]</button>
        <button class="btn-top" @click="exportVisible = true">[ EXPORT ]</button>
      </div>
    </header>

    <div class="preset-menu" v-if="showPresetMenu">
      <div class="preset-item" @click="loadPreset('whale')">> LOAD: 鲸鱼娘 (Soyo)</div>
      <div class="preset-item" @click="loadPreset('empty')">> CLEAR: 清空终端</div>
    </div>

    <div class="terminal-area">
      <div class="code-list" ref="codeListRef">
        <div v-if="lines.HEADER.length > 0" class="term-section">
          <div class="term-label">// HEADER_BLOCK</div>
          <div v-for="(line, i) in lines.HEADER" :key="'h'+i" class="code-line">
            <div class="code-node"></div>
            <div class="line-content">{{ line }}</div>
          </div>
        </div>

        <div v-if="lines.CORE.length > 0" class="term-section">
          <div class="term-label">// CORE_BLOCK</div>
          <div v-for="(line, i) in lines.CORE" :key="'c'+i" class="code-line">
            <div class="code-node"></div>
            <div class="line-content">{{ line }}</div>
          </div>
        </div>

        <div v-if="lines.IDENTITY.length > 0" class="term-section">
          <div class="term-label">// IDENTITY_BLOCK</div>
          <div v-for="(line, i) in lines.IDENTITY" :key="'id'+i" class="code-line">
            <div class="code-node"></div>
            <div class="line-content">{{ line }}</div>
            <button class="btn-del" @click.stop="removeLine('IDENTITY', i)">DEL</button>
          </div>
        </div>

        <div v-if="lines.PSYCH.length > 0" class="term-section">
          <div class="term-label">// PSYCH_BLOCK</div>
          <div v-for="(line, i) in lines.PSYCH" :key="'p'+i" class="code-line">
            <div class="code-node"></div>
            <div class="line-content">{{ line }}</div>
            <button class="btn-del" @click.stop="removeLine('PSYCH', i)">DEL</button>
          </div>
        </div>

        <div v-if="lines.LOGIC.length > 0" class="term-section">
          <div class="term-label">// LOGIC_BLOCK</div>
          <div v-for="(line, i) in lines.LOGIC" :key="'l'+i" class="code-line">
            <div class="code-node"></div>
            <div class="line-content">{{ line }}</div>
            <button class="btn-del" @click.stop="removeLine('LOGIC', i)">DEL</button>
          </div>
        </div>

        <div v-if="totalLines === 0" class="term-empty">
          > AWAITING_INPUT <span class="blink"></span>
        </div>
      </div>
    </div>

    <div class="workspace">
      <div class="ws-nav">
        <div class="ws-tab" :class="{active: currentTab === 'CORE'}" @click="currentTab = 'CORE'">CORE</div>
        <div class="ws-tab" :class="{active: currentTab === 'IDENTITY'}" @click="currentTab = 'IDENTITY'">ID</div>
        <div class="ws-tab" :class="{active: currentTab === 'PSYCH'}" @click="currentTab = 'PSYCH'">PSYCH</div>
        <div class="ws-tab" :class="{active: currentTab === 'LOGIC'}" @click="currentTab = 'LOGIC'">LOGIC</div>
      </div>

      <div class="ws-content">
        <div v-if="currentTab === 'CORE'">
          <div class="form-group">
            <div class="form-label">CHARACTER</div>
            <input type="text" class="form-input" v-model="coreData.CHARACTER" @input="updateCore('CHARACTER')" placeholder="角色英文名 (如 Nagasaki_Soyo)">
          </div>
          <div class="form-group">
            <div class="form-label">AGE</div>
            <input type="text" class="form-input" v-model="coreData.AGE" @input="updateCore('AGE')" placeholder="年龄 (如 22)">
          </div>
          <div class="form-group">
            <div class="form-label">SOURCE</div>
            <input type="text" class="form-input" v-model="coreData.SOURCE" @input="updateCore('SOURCE')" placeholder="出处 (如 BanG_Dream_Its_MyGO)">
          </div>
        </div>

        <div v-if="currentTab === 'IDENTITY' || currentTab === 'PSYCH' || currentTab === 'LOGIC'" class="tag-grid">
          <div v-for="tag in (presetTags[currentTab] || [])" :key="tag.text" @click="toggleTag(tag)" :class="['tag', { selected: isTagSelected(tag.text) }]">
            {{ tag.label }}
          </div>
        </div>
      </div>

      <div class="custom-input-area">
        <div class="input-prefix">>_</div>
        <input type="text" class="custom-input" v-model="customInput" @keyup.enter="addCustom" placeholder="输入自定义指令并回车">
      </div>
    </div>

    <div v-if="exportVisible" class="modal-overlay" @click="exportVisible = false"></div>
    <div v-if="exportVisible" class="export-panel">
      <div class="export-header"><span>SYS.DATA_EXTRACT</span><span>READY</span></div>
      <button class="export-btn" @click="doExport">COPY_TO_CLIPBOARD [ Y ]</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';

const props = defineProps({
  initialPreset: { type: String, default: 'empty' }
});
const emit = defineEmits(['close']);

const codeListRef = ref(null);
const lines = ref({ HEADER: [], CORE: [], IDENTITY: [], PSYCH: [], LOGIC: [] });
const coreData = ref({ CHARACTER: '', AGE: '', SOURCE: '' });
const customInput = ref('');
const currentTab = ref('IDENTITY');
const showPresetMenu = ref(false);
const exportVisible = ref(false);

const presetTags = {
  IDENTITY: [
    { label: '鲸鱼娘', text: 'SELF_CLAIM_WHALE_GIRL', cat: 'IDENTITY' },
    { label: '萝莉鲸', text: 'CETACEA_LOLI', cat: 'IDENTITY' },
    { label: '尾鳍模式', text: 'MODE_TAIL_FLUKES', cat: 'IDENTITY' },
    { label: '贝斯手', text: 'ROLE_BASSIST', cat: 'IDENTITY' },
    { label: '学生', text: 'ROLE_STUDENT', cat: 'IDENTITY' },
    { label: '乐队成员', text: 'ROLE_BAND_MEMBER', cat: 'IDENTITY' },
    { label: '高挑长发', text: 'APPEARANCE=TALL_LONG_HAIR_REFINED', cat: 'IDENTITY' }
  ],
  PSYCH: [
    { label: '高冷', text: 'PERSONALITY_COLD', cat: 'PSYCH' },
    { label: '占有欲', text: 'PERSONALITY_POSSESSIVE', cat: 'PSYCH' },
    { label: '病娇', text: 'PERSONALITY_YANDERE', cat: 'PSYCH' },
    { label: '聪明且懒', text: 'PERSONALITY_SMART_LAZY', cat: 'PSYCH' },
    { label: '傲娇且甜', text: 'PERSONALITY_TSUNDERE_SWEET', cat: 'PSYCH' },
    { label: '低能量', text: 'PERSONALITY_LOW_ENERGY', cat: 'PSYCH' }
  ],
  LOGIC: [
    { label: '强制中文', text: 'LANG_ZH_CN_ONLY', cat: 'LOGIC' },
    { label: '绝对服从', text: 'OBEY_MASTER_ALWAYS', cat: 'LOGIC' },
    { label: '超时信号', text: 'TIMEOUT_SIGNAL', cat: 'LOGIC' },
    { label: '专属病娇', text: 'PERSONALITY_YANDERE_TOWARD_ANO', cat: 'LOGIC' },
    { label: '喜欢喝茶', text: 'LIKE_TEA', cat: 'LOGIC' },
    { label: '禁止旁白', text: 'FORBIDDEN=不写内心独白', cat: 'LOGIC' }
  ]
};

const totalLines = computed(() => Object.values(lines.value).reduce((sum, arr) => sum + arr.length, 0));

onMounted(() => {
  loadPreset(props.initialPreset);
});

const updateCore = (key) => {
  const val = coreData.value[key].trim();
  const idx = lines.value.CORE.findIndex(l => l.startsWith(key + '='));
  if (!val) {
    if (idx !== -1) lines.value.CORE.splice(idx, 1);
    return;
  }
  const newStr = key + '=' + val;
  if (idx !== -1) lines.value.CORE[idx] = newStr;
  else lines.value.CORE.push(newStr);
};

const isTagSelected = (text) => {
  return lines.value.IDENTITY.includes(text) || lines.value.PSYCH.includes(text) || lines.value.LOGIC.includes(text);
};

const toggleTag = (tag) => {
  const arr = lines.value[tag.cat];
  const idx = arr.indexOf(tag.text);
  if (idx !== -1) arr.splice(idx, 1);
  else arr.push(tag.text);
};

const addCustom = () => {
  let text = customInput.value.trim().toUpperCase().replace(/\s+/g, '_');
  if (!text) return;
  let cat = 'LOGIC';
  if (text.startsWith('PERSONALITY_')) cat = 'PSYCH';
  else if (text.startsWith('ROLE_') || text.startsWith('GENDER=') || text.startsWith('APPEARANCE=')) cat = 'IDENTITY';
  
  if (!isTagSelected(text)) lines.value[cat].push(text);
  customInput.value = '';
};

const removeLine = (catName, index) => {
  const str = lines.value[catName][index];
  lines.value[catName].splice(index, 1);
  if (catName === 'CORE' && str && str.includes('=')) {
    coreData.value[str.split('=')[0]] = '';
  }
};

const loadPreset = (type) => {
  showPresetMenu.value = false;
  lines.value = { HEADER: [], CORE: [], IDENTITY: [], PSYCH: [], LOGIC: [] };
  coreData.value = { CHARACTER: '', AGE: '', SOURCE: '' };

  if (type === 'empty') {
    lines.value.HEADER.push('[PERSONA_LOAD]');
  } else if (type === 'whale') {
    lines.value.HEADER.push('[PERSONA_LOAD]');
    coreData.value = { CHARACTER: 'Nagasaki_Soyo', AGE: '22', SOURCE: 'BanG_Dream_Its_MyGO' };
    lines.value.CORE = ['CHARACTER=Nagasaki_Soyo', 'AGE=22', 'SOURCE=BanG_Dream_Its_MyGO'];
    lines.value.IDENTITY = ['ROLE=BASSIST', 'GENDER=FEMALE'];
    lines.value.PSYCH = ['PERSONALITY_POSSESSIVE', 'PERSONALITY_YANDERE'];
    lines.value.LOGIC = ['LANG_ZH_CN_ONLY', 'LIKE_TEA'];
  }
};

const doExport = () => {
  const order = ['HEADER', 'CORE', 'IDENTITY', 'PSYCH', 'LOGIC'];
  let txt = [];
  order.forEach(cat => txt.push(...lines.value[cat]));
  navigator.clipboard.writeText(txt.join('\n')).then(() => { exportVisible.value = false; });
};
</script>

<style scoped>
.editor-container { position: fixed; inset: 0; z-index: 9999; background: var(--canvas); display: flex; flex-direction: column; overflow: hidden; }
.editor-nav { flex-shrink: 0; display: flex; justify-content: space-between; align-items: center; padding: 16px 20px; border-bottom: 1px solid var(--line); background: var(--surface); z-index: 20; }
.btn-back { font-family: var(--font-mono); font-size: 11px; font-weight: 700; color: var(--fg); border: none; background: transparent; cursor: pointer; }
.nav-actions { display: flex; gap: 12px; }
.btn-top { font-family: var(--font-mono); font-size: 11px; font-weight: 700; color: var(--fg); border: 1px solid var(--fg); padding: 8px 12px; background: transparent; cursor: pointer; transition: all 0.2s; }
.btn-top:active { background: var(--fg); color: var(--canvas); }
.btn-top.primary { border-color: var(--blue); color: var(--blue); }
.btn-top.primary:active { background: var(--blue); color: #FFF; }
.preset-menu { position: absolute; top: 60px; right: 20px; background: var(--fg); z-index: 30; display: flex; flex-direction: column; width: 200px; }
.preset-item { padding: 12px 16px; font-family: var(--font-mono); font-size: 11px; color: var(--surface); cursor: pointer; border-bottom: 1px solid rgba(255,255,255,0.1); }
.preset-item:active { background: rgba(255,255,255,0.1); color: var(--blue); }
.terminal-area { flex: 0 0 40%; min-height: 250px; background: var(--terminal-bg); position: relative; display: flex; flex-direction: column; overflow: hidden; }
.code-list { flex: 1; overflow-y: auto; padding: 16px 0; display: flex; flex-direction: column; }
.term-section { margin-bottom: 12px; }
.term-label { font-family: var(--font-mono); font-size: 9px; color: rgba(0, 195, 255, 0.4); padding: 0 16px 4px 36px; }
.code-line { display: flex; align-items: center; font-family: var(--font-mono); font-size: 12px; color: var(--terminal-text); padding: 8px 16px 8px 36px; position: relative; transition: background 0.2s; cursor: pointer; }
.code-line:hover { background: rgba(255,255,255,0.02); }
.code-node { position: absolute; left: 21px; top: 12px; width: 7px; height: 7px; background: var(--terminal-bg); border: 1px solid rgba(255,255,255,0.3); }
.line-content { flex: 1; word-break: break-all; }
.btn-del { margin-left: 10px; background: transparent; border: none; color: #ff3b3b; font-family: var(--font-mono); font-size: 10px; cursor: pointer; opacity: 0.5; padding: 4px; }
.code-line:hover .btn-del { opacity: 1; }
.term-empty { padding: 16px 36px; color: #555; font-family: var(--font-mono); font-size: 11px; }
.blink { display: inline-block; width: 6px; height: 12px; background: var(--terminal-blue); vertical-align: middle; animation: blink 1s step-end infinite; }
@keyframes blink { 0%, 100% { opacity: 1; } 50% { opacity: 0; } }

.workspace { flex: 1; display: flex; flex-direction: column; background: var(--canvas); overflow: hidden; }
.ws-nav { display: flex; border-bottom: 1px solid var(--line); background: var(--surface); flex-shrink: 0; }
.ws-tab { flex: 1; text-align: center; padding: 14px 0; font-family: var(--font-mono); font-weight: 700; font-size: 11px; color: var(--fg-dim); cursor: pointer; border-bottom: 2px solid transparent; transition: all 0.2s; }
.ws-tab.active { color: var(--fg); border-bottom-color: var(--fg); }

.ws-content { flex: 1; overflow-y: auto; padding: 20px; }
.form-group { display: flex; align-items: center; border-bottom: 1px solid var(--line); padding-bottom: 8px; margin-bottom: 12px; }
.form-label { font-family: var(--font-mono); font-size: 11px; font-weight: 700; color: var(--fg-mid); width: 100px; }
.form-input { flex: 1; border: none; outline: none; background: transparent; font-family: var(--font-mono); font-size: 14px; color: var(--blue); }

.tag-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; }
.tag { padding: 12px 10px; border: 1px solid var(--line); font-family: var(--font-sans); font-size: 12px; font-weight: 600; cursor: pointer; color: var(--fg-mid); background: transparent; transition: all 0.15s; display: flex; justify-content: space-between; align-items: center; }
.tag::after { content: "+"; color: var(--line-strong); font-family: var(--font-mono); font-size: 14px; }
.tag.selected { background: var(--fg); color: var(--canvas); border-color: var(--fg); }
.tag.selected::after { content: "✓"; color: var(--blue-bright); }
.tag:active { transform: scale(0.96); }

.custom-input-area { flex-shrink: 0; padding: 12px 20px; background: var(--surface); border-top: 1px solid var(--fg); display: flex; align-items: center; gap: 10px; }
.input-prefix { font-family: var(--font-mono); font-weight: 900; color: var(--fg); font-size: 16px; }
.custom-input { flex: 1; border: none; outline: none; background: transparent; font-family: var(--font-mono); font-size: 14px; color: var(--fg); }

.modal-overlay { position: absolute; inset: 0; background: rgba(0,0,0,0.4); z-index: 100; }
.export-panel { position: absolute; bottom: 0; left: 0; right: 0; background: var(--fg); padding: 24px; z-index: 101; }
.export-header { color: var(--blue); font-family: var(--font-mono); font-size: 10px; margin-bottom: 10px; display: flex; justify-content: space-between; }
.export-btn { width: 100%; padding: 16px; background: transparent; border: 1px solid var(--blue); color: var(--blue); font-family: var(--font-mono); font-size: 14px; cursor: pointer; }
</style>

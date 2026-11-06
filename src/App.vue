<template>
  <div class="app-root">
    <SplashScreen v-if="!splashDone" @done="onSplashDone" />
    
    <div class="main-interface" v-if="splashDone">
      <HomeView 
        v-show="currentView === 'home'" 
        @open-editor="handleOpenEditor"
      />
      
      <EditorView 
        v-if="currentView === 'editor'" 
        :initial-preset="presetToLoad"
        @close="currentView = 'home'"
      />
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import SplashScreen from './components/SplashScreen.vue';
import HomeView from './components/HomeView.vue';
import EditorView from './components/EditorView.vue';

const splashDone = ref(false);
const currentView = ref('home');
const presetToLoad = ref('empty');

const onSplashDone = () => {
  splashDone.value = true;
};

const handleOpenEditor = (presetType) => {
  presetToLoad.value = presetType || 'empty';
  currentView.value = 'editor';
};
</script>

<style>
@import './style.css';

.app-root {
  width: 100%;
  height: 100%;
  position: relative;
}
.main-interface {
  width: 100%;
  height: 100%;
  position: absolute;
  inset: 0;
}
</style>

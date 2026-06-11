<template>
  <div class="relative w-full h-[100dvh] overflow-hidden bg-[#140411] text-white flex flex-col font-sans select-none">
    
    <!-- Floating Audio Ambient Control HUD (Top Right) -->
    <div class="absolute top-4 right-4 z-50 pointer-events-auto flex items-center gap-2">
      <!-- Minimal sound wave active bars -->
      <div v-if="!isMuted" class="flex items-end gap-0.5 h-3.5 px-1 animate-pulse">
        <span class="w-0.5 bg-[#ff5e8c] h-2 animate-[barPulse_0.6s_infinite_alternate_ease-in-out]"></span>
        <span class="w-0.5 bg-[#c84095] h-3 animate-[barPulse_0.8s_infinite_alternate_ease-in-out_0.2s]"></span>
        <span class="w-0.5 bg-[#ffd27a] h-1.5 animate-[barPulse_0.5s_infinite_alternate_ease-in-out_0.1s]"></span>
      </div>

      <button
        @click="toggleAmbientAudio"
        class="flex items-center justify-center w-10 h-10 rounded-full border bg-black/65 backdrop-blur-md cursor-pointer transition-all duration-300"
        :class="[
          isMuted 
            ? 'border-white/20 text-gray-400 hover:text-white' 
            : 'border-[#ff5e8c] text-[#ff5e8c] shadow-[0_0_12px_rgba(255,94,140,0.4)] hover:scale-105'
        ]"
        title="Background Music Toggle"
        id="bg_music_toggle"
      >
        <!-- Speaker Icon Mute -->
        <svg v-if="isMuted" class="w-4.5 h-4.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.8" d="M5.586 15H4a1 1 0 01-1-1v-4a1 1 0 011-1h1.586l4.707-4.707C10.923 3.663 12 4.109 12 5v14c0 .891-1.077 1.337-1.707.707L5.586 15z" />
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.8" d="M17 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2" />
        </svg>

        <!-- Speaker Icon Active -->
        <svg v-else class="w-4.5 h-4.5 animate-[bellBeat_1.5s_infinite_ease-in-out]" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.8" d="M15.536 8.464a5 5 0 010 7.072m2.828-9.9a9 9 0 010 12.728M5.586 15H4a1 1 0 01-1-1v-4a1 1 0 011-1h1.586l4.707-4.707C10.923 3.663 12 4.109 12 5v14c0 .891-1.077 1.337-1.707.707L5.586 15z" />
        </svg>
      </button>
    </div>

    <!-- Router Stage Components with cross-fade view effects -->
    <transition name="page-fade" mode="out-in">
      <UnlockGate3D v-if="activeStage === 'gate'" @next="advanceTo('tunnel')" />
      <TimeTunnel3D v-else-if="activeStage === 'tunnel'" @next="advanceTo('magic')" />
      <MagicInteraction v-else-if="activeStage === 'magic'" :selectedTheme="selectedTheme" @select-theme="setTheme" @next="advanceTo('vouchers')" />
      <WishCoupon v-else-if="activeStage === 'vouchers'" :selectedTheme="selectedTheme" @next="advanceTo('celebration')" />
      <CelebrationCake3D v-else-if="activeStage === 'celebration'" :selectedTheme="selectedTheme" @next="advanceTo('gate')" />
    </transition>

    <!-- Tiny Toast warning for muting status if she didn't enable it yet -->
    <transition name="toast-slide">
      <div 
        v-if="showAudioPrompt" 
        class="absolute bottom-24 left-1/2 -translate-x-1/2 bg-black/80 backdrop-blur-md py-2.5 px-4 rounded-xl border border-[#ff5e8c]/30 text-[11px] font-sans font-bold tracking-wider text-[#ffd27a]/95 text-center shadow-[0_0_15px_rgba(255,94,140,0.25)] flex items-center gap-2 pointer-events-auto z-40"
      >
        <span class="animate-ping w-1.5 h-1.5 rounded-full bg-[#ff5e8c] inline-block"></span>
        <span>👉 点击右上角开启专属浪漫背景音乐 🎵</span>
      </div>
    </transition>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';

// Page Views components
import UnlockGate3D from './components/UnlockGate3D.vue';
import TimeTunnel3D from './components/TimeTunnel3D.vue';
import MagicInteraction from './components/MagicInteraction.vue';
import WishCoupon from './components/WishCoupon.vue';
import CelebrationCake3D from './components/CelebrationCake3D.vue';

// Active router stage states
// Stages layout: gate -> tunnel -> magic -> vouchers -> celebration -> (gate)
const activeStage = ref<'gate' | 'tunnel' | 'magic' | 'vouchers' | 'celebration'>('gate');

// 全局选中的礼服主题，影响后续所有关卡配色
const selectedTheme = ref<'pink' | 'blue' | 'gold'>('pink');

function setTheme(theme: 'pink' | 'blue' | 'gold') {
  selectedTheme.value = theme;
}

// Ambient Audio States
const isMuted = ref(true);
const showAudioPrompt = ref(true);

let audioCtx: AudioContext | null = null;
let chordInterval: any = null;

// Character Console Easter Egg and initializers
onMounted(() => {
  logConsoleArt();
  
  // Slide away audio alert after 5.5 seconds
  setTimeout(() => {
    showAudioPrompt.value = false;
  }, 5500);
});

onUnmounted(() => {
  stopSpaceSynth();
});

// Advanced Interactive Stage advance triggers
function advanceTo(stage: 'gate' | 'tunnel' | 'magic' | 'vouchers' | 'celebration') {
  activeStage.value = stage;
  
  // Close audio recommendation toast once she begins interacting
  showAudioPrompt.value = false;
}

// Giant beautiful Console character Easter Egg for is wife
function logConsoleArt() {
  console.log(`
%c   ████████      ████████
 %c ██      ████  ████      ██
%c ██          ████          ██
%c ██            ██            ██
 %c  ██                      ██
 %c    ██                  ██
 %c      ██              ██
 %c        ██          ██
 %c          ██      ██
 %c            ██  ██
 %c              ██

%c✨ COSMIC CODE LOCKED :: 老婆，生日快乐！ :: ✨
-----------------------------------------
[LOVE MATRIX:: COMPILING ENDLESS ADORATION]
🔓 DECRYPT STATE: COMPLETED // 浪漫编译就绪
🌟 KEY: LOVE-PROTOCOL-ACTIVE-FOREVER
`, 
  'color: #ff5e8c; font-weight: bold;', 
  'color: #ff5e8c; font-weight: bold;', 
  'color: #c84095; font-weight: bold;', 
  'color: #c84095; font-weight: bold;', 
  'color: #ffd27a; font-weight: bold;', 
  'color: #ffd27a; font-weight: bold;', 
  'color: #ff5e8c; font-weight: bold;', 
  'color: #ff5e8c; font-weight: bold;', 
  'color: #c84095; font-weight: bold;', 
  'color: #c84095; font-weight: bold;', 
  'color: #ffd27a; font-weight: bold;', 
  'color: #ff5e8c; font-weight: bold; font-size: 14px; text-shadow: 0 0 8px rgba(255,94,140,0.4);'
  );
}

// Autonomous Web Audio synthesized romantic ambient space bell chords loop
function toggleAmbientAudio() {
  isMuted.value = !isMuted.value;
  showAudioPrompt.value = false;

  if (isMuted.value) {
    stopSpaceSynth();
  } else {
    playSpaceSynth();
  }
}

function playSpaceSynth() {
  // Ensure we boot audio context lazily based on user click trust parameters to keep browsers happy
  if (!audioCtx) {
    audioCtx = new (window.AudioContext || (window as any).webkitAudioContext)();
  }

  if (audioCtx.state === 'suspended') {
    audioCtx.resume();
  }

  // Harmonic chord scales (peaceful dreamlike warm frequencies)
  const padChords = [
    [196.00, 246.94, 293.66, 392.00], // G Major (Peaceful)
    [220.00, 261.63, 329.63, 440.00], // A Minor (Reflective)
    [174.61, 220.00, 261.63, 349.23], // F Major (Astral)
    [196.00, 246.94, 293.66, 493.88]  // G Major add9 (Resolved)
  ];

  let currentChordIndex = 0;

  const playIntervalPulse = () => {
    if (!audioCtx || isMuted.value) return;
    const now = audioCtx.currentTime;
    const selectedFreqs = padChords[currentChordIndex];
    currentChordIndex = (currentChordIndex + 1) % padChords.length;

    selectedFreqs.forEach((freq, idx) => {
      const oscillator = audioCtx.createOscillator();
      const gainNode = audioCtx.createGain();

      // Soft waveform selection
      oscillator.type = 'triangle';
      oscillator.frequency.setValueAtTime(freq, now);

      // Micro detuning to generate rich choral space thickness
      oscillator.detune.setValueAtTime((idx - 1.5) * 6.5, now);

      // Linear attack, lingering sustain, beautiful celestial exponential decay parameters
      gainNode.gain.setValueAtTime(0, now);
      gainNode.gain.linearRampToValueAtTime(0.045, now + 1.2); // sweet slow attack fade-in
      gainNode.gain.setValueAtTime(0.045, now + 1.8);
      gainNode.gain.exponentialRampToValueAtTime(0.0001, now + 4.2); // slow warm wind-down

      oscillator.connect(gainNode);
      gainNode.connect(audioCtx.destination);

      oscillator.start(now);
      // Soft stop after lifecycle completes
      oscillator.stop(now + 4.3);
    });
  };

  playIntervalPulse();
  chordInterval = setInterval(playIntervalPulse, 3800);
}

function stopSpaceSynth() {
  if (chordInterval) {
    clearInterval(chordInterval);
    chordInterval = null;
  }
}
</script>

<style>
/* Custom Keyframe animations for main template visuals */
@keyframes barPulse {
  0% {
    height: 3px;
  }
  100% {
    height: 14px;
  }
}

@keyframes bellBeat {
  0%, 100% {
    transform: scale(1);
    opacity: 1;
  }
  50% {
    transform: scale(1.06);
    opacity: 0.85;
  }
}

/* Page transitions styling layout */
.page-fade-enter-active, .page-fade-leave-active {
  transition: opacity 0.5s ease, transform 0.5s ease;
}
.page-fade-enter-from {
  opacity: 0;
  transform: scale(0.97) translateY(10px);
}
.page-fade-leave-to {
  opacity: 0;
  transform: scale(0.97) translateY(-10px);
}

/* Toast slides rules */
.toast-slide-enter-active, .toast-slide-leave-active {
  transition: all 0.4s cubic-bezier(0.16, 1, 0.3, 1);
}
.toast-slide-enter-from, .toast-slide-leave-to {
  opacity: 0;
  transform: translate(-50%, 20px);
}
</style>

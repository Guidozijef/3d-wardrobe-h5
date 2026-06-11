<template>
  <div class="relative w-full h-full overflow-hidden bg-[#140411] flex flex-col items-center justify-between p-6 hologram-grid">
    <!-- WebGL Background Canvas -->
    <div ref="canvasContainer" class="absolute inset-0 w-full h-full z-0 pointer-events-none"></div>

    <!-- Interface Content layer -->
    <div class="relative z-10 w-full max-w-md mx-auto flex flex-col items-center justify-between h-full pointer-events-none">
      
      <!-- Top Title / System Status -->
      <div class="w-full text-center mt-12 py-3 border-y border-[#ff5e8c]/15 bg-black/40 backdrop-blur-md px-4 rounded-xl transform scale-95 md:scale-100 shadow-[0_4px_25px_rgba(255,94,140,0.1)]">
        <div class="font-serif italic text-xs text-[#ffd27a] tracking-[0.2em] mb-1">
          A Gift of Endless Sweetness // 浪漫祈愿盒
        </div>
        <h2 class="text-xl font-bold font-sans tracking-wide text-white drop-shadow-[0_2px_4px_rgba(0,0,0,0.5)]">
          愿望密码：专属心愿密钥
        </h2>
      </div>

      <!-- Center Interaction Circle (Fingerprint Area) -->
      <div class="flex flex-col items-center justify-center my-auto relative pointer-events-auto">
        <!-- Floating Aura Glow -->
        <div 
          class="absolute w-[220px] h-[220px] rounded-full filter blur-2xl transition-all duration-300"
          :class="[
            isCharging 
              ? 'bg-[#ff5e8c]/30 scale-110 animate-pulse' 
              : isUnlocked 
                ? 'bg-[#ffd27a]/25 scale-125' 
                : 'bg-[#c84095]/10 scale-95'
          ]"
        ></div>

        <!-- Pulse Ring Outer -->
        <div 
          class="relative w-[180px] h-[180px] rounded-full border flex items-center justify-center transition-all duration-500 ease-out cursor-pointer"
          :class="[
            isCharging 
              ? 'border-[#ff5e8c] shadow-[0_0_25px_rgba(255,94,140,0.6)] scale-102' 
              : isUnlocked 
                ? 'border-[#ffd27a] shadow-[0_0_30px_rgba(255,210,122,0.5)]' 
                : 'border-[#ff5e8c]/25 shadow-[0_0_15px_rgba(255,94,140,0.12)]'
          ]"
          @touchstart.prevent="startCharge"
          @touchend="stopCharge"
          @mousedown="startCharge"
          @mouseup="stopCharge"
          @mouseleave="stopCharge"
        >
          <!-- Dynamic Progress Arc (SVG) -->
          <svg class="absolute inset-0 w-full h-full -rotate-90">
            <circle
              cx="90"
              cy="90"
              r="84"
              class="stroke-[#c84095]/10 fill-none"
              stroke-width="2"
            />
            <circle
              cx="90"
              cy="90"
              r="84"
              class="stroke-[#ff5e8c] fill-none transition-all duration-100"
              stroke-width="4"
              stroke-linecap="round"
              :stroke-dasharray="2 * Math.PI * 84"
              :stroke-dashoffset="2 * Math.PI * 84 * (1 - chargeProgress / 100)"
              v-if="!isUnlocked"
            />
            <circle
              cx="90"
              cy="90"
              r="84"
              class="stroke-[#ffd27a] fill-none"
              stroke-width="4"
              stroke-linecap="round"
              :stroke-dasharray="2 * Math.PI * 84"
              stroke-dashoffset="0"
              v-else
            />
          </svg>

          <!-- Icon / Scanning Overlay -->
          <div class="flex flex-col items-center justify-center select-none">
            <!-- Charging Animation Laser Bar -->
            <div 
              v-if="isCharging"
              class="absolute left-6 right-6 h-0.5 bg-[#ffd27a] shadow-[0_0_12px_rgba(255,210,122,1)] animate-bounce"
              style="top: 25%"
            ></div>

            <!-- Lock Heart SVG -->
            <div v-if="!isCharging && !isUnlocked" class="flex flex-col items-center justify-center transform hover:scale-105 duration-300">
              <svg 
                class="w-14 h-14 text-[#ff5e8c] filter drop-shadow-[0_0_10px_rgba(255,94,140,0.5)]"
                fill="currentColor" 
                viewBox="0 0 24 24"
              >
                <path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z"/>
              </svg>
              <span class="text-[9px] text-[#ff5e8c]/80 tracking-widest font-mono font-bold mt-1">PRESS & HOLD</span>
            </div>

            <!-- Charging Face -->
            <div v-else-if="isCharging" class="text-[#ff5e8c] text-[10px] font-mono font-bold animate-pulse text-center leading-relaxed">
              LOVE LOADING
              <div class="text-xl text-white font-serif italic mt-1 font-bold">{{ Math.round(chargeProgress) }}%</div>
            </div>

            <!-- Heart / Unlocked Face -->
            <svg 
              v-else-if="isUnlocked" 
              class="w-16 h-16 text-[#ffd27a] animate-bounce filter drop-shadow-[0_0_15px_rgba(255,210,122,0.8)]" 
              fill="currentColor" 
              viewBox="0 0 24 24"
            >
              <path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z"/>
            </svg>
          </div>
        </div>

        <!-- Instruction text helper -->
        <p class="mt-6 font-sans text-xs text-gray-300 text-center tracking-wider max-w-[240px]">
          {{ instructionText }}
        </p>
      </div>

      <!-- Holographic Overlay Screen inside Heart -->
      <div 
        ref="glassOverlay"
        class="absolute inset-x-4 top-1/2 -translate-y-1/2 p-6 rounded-3xl bg-[#140411]/90 border border-[#ff5e8c]/30 backdrop-blur-lg opacity-0 pointer-events-none transition-all duration-1000 scale-90 flex flex-col items-center justify-center text-center shadow-[0_10px_40px_rgba(255,94,140,0.15)]"
      >
        <span class="font-serif italic text-xs text-[#ffd27a] tracking-[0.2em] mb-2 block">Warmest Blessings // 给你最甜的祝福</span>
        <h1 class="text-3xl font-bold text-transparent bg-clip-text bg-gradient-to-r from-white via-[#ff5e8c] to-[#ffd27a] tracking-widest leading-normal">
          老婆，生日快乐！
        </h1>
        <p class="text-xs text-gray-200 mt-4 leading-relaxed font-sans max-w-xs px-2">
          “宇宙中的万千极光，今日皆因你折射暖芒。祝我的小仙女，岁岁长安，璀璨如初。”
        </p>
        <div class="mt-5 w-16 h-[1px] bg-[#ffd27a]/40 shadow-[0_0_10px_#ffd27a]"></div>
        <div class="text-[10px] font-sans text-[#ffd27a] mt-4 tracking-widest animate-pulse font-medium">
          正在开启时光心愿长廊 ...
        </div>
      </div>

      <!-- Ground Info Footer -->
      <div class="w-full text-center mb-8">
        <span class="font-serif italic text-[10px] text-gray-500 tracking-[0.1em]">
          Designed For My Dearest Wife // 💖
        </span>
      </div>

    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';
import * as THREE from 'three';
import { gsap } from 'gsap';

const emit = defineEmits<{
  (e: 'next'): void;
}>();

// Refs
const canvasContainer = ref<HTMLDivElement | null>(null);
const glassOverlay = ref<HTMLDivElement | null>(null);

// Interactive State vars
const isCharging = ref(false);
const isUnlocked = ref(false);
const chargeProgress = ref(0);
const instructionText = ref('点击并长按上方粉色心愿印记 💖');

let chargeInterval: any = null;

// Three.js core items (DO NOT make them reactive/ref to improve memory & FPS!)
let scene: THREE.Scene;
let camera: THREE.PerspectiveCamera;
let renderer: THREE.WebGLRenderer;
let particleGeometry: THREE.BufferGeometry;
let particleMaterial: THREE.PointsMaterial;
let particleSystem: THREE.Points;
let animationFrameId: number;

// Shockwave elements
let shockwaveMesh: THREE.Mesh | null = null;
let shockwaveMesh2: THREE.Mesh | null = null;

const particleCount = 6500;
let positions: Float32Array;

// Target arrays for animations
let targetScatter: number[] = [];
let targetHeart: number[] = [];
let targetCenter: number[] = [];

// Audio Synthesizer states
let audioCtx: AudioContext | null = null;
let chargeOsc: OscillatorNode | null = null;
let chargeGain: GainNode | null = null;

// Initialization
onMounted(() => {
  initThree();
  window.addEventListener('resize', handleResize);
});

onUnmounted(() => {
  // Clear event listeners
  window.removeEventListener('resize', handleResize);
  stopCharge();
  stopChargeAudio();
  
  // Clean up WebGL Context thoroughly to avoid leaks & hot phones
  cancelAnimationFrame(animationFrameId);
  
  if (particleGeometry) particleGeometry.dispose();
  if (particleMaterial) particleMaterial.dispose();
  if (shockwaveMesh) {
    if (shockwaveMesh.geometry) shockwaveMesh.geometry.dispose();
    if (shockwaveMesh.material) (shockwaveMesh.material as THREE.Material).dispose();
  }
  if (shockwaveMesh2) {
    if (shockwaveMesh2.geometry) shockwaveMesh2.geometry.dispose();
    if (shockwaveMesh2.material) (shockwaveMesh2.material as THREE.Material).dispose();
  }
  if (renderer) {
    renderer.dispose();
    renderer.forceContextLoss();
    if (renderer.domElement && canvasContainer.value && canvasContainer.value.contains(renderer.domElement)) {
      canvasContainer.value.removeChild(renderer.domElement);
    }
  }
  if (audioCtx) {
    audioCtx.close();
  }
});

// Programmatic High-glow neon circle texture
function createCircleTexture(): THREE.Texture {
  const canvas = document.createElement('canvas');
  canvas.width = 64;
  canvas.height = 64;
  const ctx = canvas.getContext('2d');
  if (ctx) {
    const gradient = ctx.createRadialGradient(32, 32, 0, 32, 32, 32);
    gradient.addColorStop(0, 'rgba(255, 255, 255, 1)');
    gradient.addColorStop(0.15, 'rgba(255, 80, 150, 0.95)');
    gradient.addColorStop(0.4, 'rgba(180, 10, 120, 0.55)');
    gradient.addColorStop(0.8, 'rgba(30, 0, 60, 0.15)');
    gradient.addColorStop(1, 'rgba(0, 0, 0, 0)');
    ctx.fillStyle = gradient;
    ctx.fillRect(0, 0, 64, 64);
  }
  const texture = new THREE.CanvasTexture(canvas);
  return texture;
}

// Web Audio charging synthesizer hum (linear pitch shift)
function startChargeAudio() {
  try {
    if (!audioCtx) {
      audioCtx = new (window.AudioContext || (window as any).webkitAudioContext)();
    }
    if (audioCtx.state === 'suspended') {
      audioCtx.resume();
    }
    
    chargeOsc = audioCtx.createOscillator();
    chargeGain = audioCtx.createGain();
    
    chargeOsc.type = 'sine';
    // Start at a low romantic G3 note (196 Hz)
    chargeOsc.frequency.setValueAtTime(196, audioCtx.currentTime);
    
    chargeGain.gain.setValueAtTime(0, audioCtx.currentTime);
    chargeGain.gain.linearRampToValueAtTime(0.05, audioCtx.currentTime + 0.1);
    
    chargeOsc.connect(chargeGain);
    chargeGain.connect(audioCtx.destination);
    
    chargeOsc.start();
  } catch (e) {
    console.error(e);
  }
}

function updateChargeAudio(progress: number) {
  if (!chargeOsc || !audioCtx) return;
  // Increase frequency from 196Hz to 440Hz (A4) based on progress
  const targetFreq = 196 + (440 - 196) * (progress / 100);
  chargeOsc.frequency.setTargetAtTime(targetFreq, audioCtx.currentTime, 0.05);
}

function stopChargeAudio() {
  if (chargeGain && audioCtx) {
    try {
      chargeGain.gain.setValueAtTime(chargeGain.gain.value, audioCtx.currentTime);
      chargeGain.gain.exponentialRampToValueAtTime(0.0001, audioCtx.currentTime + 0.15);
      const tempOsc = chargeOsc;
      setTimeout(() => {
        try { tempOsc?.stop(); } catch(e){}
      }, 200);
    } catch(e){}
  }
  chargeOsc = null;
  chargeGain = null;
}

// Gorgeous major chord arpeggio chime on unlock
function playUnlockChime() {
  try {
    if (!audioCtx) {
      audioCtx = new (window.AudioContext || (window as any).webkitAudioContext)();
    }
    const now = audioCtx.currentTime;
    const freqs = [261.63, 329.63, 392.00, 523.25, 659.25]; // C Major Arpeggio
    
    freqs.forEach((f, idx) => {
      const osc = audioCtx.createOscillator();
      const gainNode = audioCtx.createGain();
      osc.type = 'triangle';
      osc.frequency.setValueAtTime(f, now + idx * 0.06);
      
      gainNode.gain.setValueAtTime(0, now + idx * 0.06);
      gainNode.gain.linearRampToValueAtTime(0.05, now + idx * 0.06 + 0.02);
      gainNode.gain.exponentialRampToValueAtTime(0.0001, now + idx * 0.06 + 1.2);
      
      osc.connect(gainNode);
      gainNode.connect(audioCtx.destination);
      osc.start(now + idx * 0.06);
      osc.stop(now + idx * 0.06 + 1.3);
    });
  } catch(e){}
}

function initThree() {
  if (!canvasContainer.value) return;

  const width = canvasContainer.value.clientWidth || window.innerWidth;
  const height = canvasContainer.value.clientHeight || window.innerHeight;

  // Scene setup
  scene = new THREE.Scene();
  scene.fog = new THREE.FogExp2('#140411', 0.0035);

  // Camera Setup
  camera = new THREE.PerspectiveCamera(60, width / height, 1, 1000);
  const aspect = width / height;
  camera.position.z = 240 * (aspect < 0.75 ? 0.75 / aspect : 1.0);

  // Renderer Setup
  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true, powerPreference: "high-performance" });
  renderer.setSize(width, height);
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
  canvasContainer.value.appendChild(renderer.domElement);

  // Generate particles
  particleGeometry = new THREE.BufferGeometry();
  positions = new Float32Array(particleCount * 3);

  // precalculate coordinates
  for (let i = 0; i < particleCount; i++) {
    // Phase 1 - Scatter Coordinates (Deep space stars)
    const zRadius = 350;
    const x = (Math.random() - 0.5) * zRadius * 1.8;
    const y = (Math.random() - 0.5) * zRadius * 1.5;
    const z = (Math.random() - 0.5) * zRadius;

    positions[i * 3] = x;
    positions[i * 3 + 1] = y;
    positions[i * 3 + 2] = z;

    targetScatter.push(x, y, z);

    // Coordinate target 2 - Fingerprint center (汇聚到指纹一圈)
    const thetaCenter = Math.random() * Math.PI * 2;
    const radiusCenter = 5 + Math.random() * 25;
    targetCenter.push(
      Math.cos(thetaCenter) * radiusCenter,
      Math.sin(thetaCenter) * radiusCenter,
      (Math.random() - 0.5) * 15
    );

    // Coordinate target 3 - Volumetric 3D Heart surface
    const theta = Math.random() * Math.PI * 2;
    const phi = (Math.random() - 0.5) * Math.PI; // volumetric modifier

    // Equation for 3D Heart
    const scale = 4.2;
    const hx = 16 * Math.pow(Math.sin(theta), 3) * Math.cos(phi) * scale;
    const hy = (13 * Math.cos(theta) - 5 * Math.cos(2 * theta) - 2 * Math.cos(3 * theta) - Math.cos(4 * theta)) * scale;
    const hz = 12 * Math.sin(phi) * scale;

    // Shift coordinates slightly up for optical centering on mobile screen
    targetHeart.push(hx, hy + 20, hz);
  }

  particleGeometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));

  // Particle Material With Glowing Texture and Blending overrides!
  particleMaterial = new THREE.PointsMaterial({
    size: 5.5,
    map: createCircleTexture(),
    transparent: true,
    depthWrite: false,
    blending: THREE.AdditiveBlending,
    opacity: 0.85
  });

  particleSystem = new THREE.Points(particleGeometry, particleMaterial);
  scene.add(particleSystem);

  // Dual-layered Shockwave Ring setup
  const ringGeom1 = new THREE.RingGeometry(0.8, 1.0, 64);
  const ringMat1 = new THREE.MeshBasicMaterial({
    color: 0xff5e8c,
    side: THREE.DoubleSide,
    transparent: true,
    opacity: 0,
    blending: THREE.AdditiveBlending
  });
  shockwaveMesh = new THREE.Mesh(ringGeom1, ringMat1);
  shockwaveMesh.position.set(0, 0, 10);
  scene.add(shockwaveMesh);

  const ringGeom2 = new THREE.RingGeometry(0.8, 1.0, 64);
  const ringMat2 = new THREE.MeshBasicMaterial({
    color: 0xffd27a,
    side: THREE.DoubleSide,
    transparent: true,
    opacity: 0,
    blending: THREE.AdditiveBlending
  });
  shockwaveMesh2 = new THREE.Mesh(ringGeom2, ringMat2);
  shockwaveMesh2.position.set(0, 0, 9);
  scene.add(shockwaveMesh2);

  // Start Animation Loop
  animate();
}

function handleResize() {
  if (!canvasContainer.value || !renderer || !camera) return;
  const width = canvasContainer.value.clientWidth;
  const height = canvasContainer.value.clientHeight;
  
  const aspect = width / height;
  camera.aspect = aspect;
  camera.position.z = 240 * (aspect < 0.75 ? 0.75 / aspect : 1.0);
  camera.updateProjectionMatrix();
  renderer.setSize(width, height);
}

// Particle simulation tick
function animate() {
  animationFrameId = requestAnimationFrame(animate);

  // Slow constant rotation
  if (particleSystem) {
    if (isUnlocked.value) {
      // Rotate 3D Heart elegantly for spectacular cosmic look
      particleSystem.rotation.y += 0.009;
      particleSystem.rotation.z = Math.sin(Date.now() * 0.0005) * 0.08;
    } else if (isCharging.value) {
      // Fast swirling vortex of energy inwards
      particleSystem.rotation.y += 0.04;
    } else {
      // Normal background drift
      particleSystem.rotation.y += 0.001;
      particleSystem.rotation.x += 0.0004;
    }
  }

  renderer.render(scene, camera);
}

// Interactivity handlers
function startCharge() {
  if (isUnlocked.value) return;

  isCharging.value = true;
  instructionText.value = '正在为您编织浪漫心愿，爱意蓄能中... 💖';
  startChargeAudio();

  // GSAP animation: drag particles toward finger footprint center coordinate (Phase 1)
  const posAttr = particleGeometry.attributes.position;
  const items = { t: 0 };

  // Smoothly Tween positions to center and apply vortex rotation
  gsap.killTweensOf(items);
  gsap.to(items, {
    t: 1,
    duration: 1.8,
    ease: 'power2.out',
    onUpdate: () => {
      if (!isCharging.value) return; // Guard against premature release
      
      const currentArr = posAttr.array as Float32Array;
      const angleMultiplier = items.t * 2.5; // max rotation twist in radians
      for (let i = 0; i < particleCount; i++) {
        const i3 = i * 3;
        // Interpolate between initial scatter and fingerprint orbit
        const bx = targetScatter[i3] * (1 - items.t) + targetCenter[i3] * items.t;
        const by = targetScatter[i3 + 1] * (1 - items.t) + targetCenter[i3 + 1] * items.t;
        const bz = targetScatter[i3 + 2] * (1 - items.t) + targetCenter[i3 + 2] * items.t;

        // Apply dynamic vortex twist around Z-axis
        const radius = Math.sqrt(bx * bx + by * by) || 1;
        const twistAngle = (45 / (radius + 15)) * angleMultiplier;

        currentArr[i3] = bx * Math.cos(twistAngle) - by * Math.sin(twistAngle);
        currentArr[i3 + 1] = bx * Math.sin(twistAngle) + by * Math.cos(twistAngle);
        currentArr[i3 + 2] = bz;
      }
      posAttr.needsUpdate = true;
    }
  });

  // Increment percentage progress
  if (chargeInterval) clearInterval(chargeInterval);
  chargeInterval = setInterval(() => {
    if (chargeProgress.value < 100) {
      chargeProgress.value += 3.5;
      updateChargeAudio(chargeProgress.value);
    } else {
      clearInterval(chargeInterval);
      triggerUnlock();
    }
  }, 60);
}

function stopCharge() {
  if (isUnlocked.value) return;
  isCharging.value = false;
  clearInterval(chargeInterval);
  stopChargeAudio();

  if (chargeProgress.value < 100) {
    // Reset progress smoothly
    gsap.to(chargeProgress, { value: 0, duration: 0.5 });
    instructionText.value = '心愿充填失败。请持续按住爱心徽章哦 💐';

    // Disperse particles back to scatter format
    const posAttr = particleGeometry.attributes.position;
    const items = { t: 0 };
    const currentPositions = Array.from(posAttr.array as Float32Array);

    gsap.to(items, {
      t: 1,
      duration: 1.5,
      ease: 'power3.out',
      onUpdate: () => {
        const currentArr = posAttr.array as Float32Array;
        for (let i = 0; i < particleCount; i++) {
          const i3 = i * 3;
          currentArr[i3] = currentPositions[i3] * (1 - items.t) + targetScatter[i3] * items.t;
          currentArr[i3 + 1] = currentPositions[i3 + 1] * (1 - items.t) + targetScatter[i3 + 1] * items.t;
          currentArr[i3 + 2] = currentPositions[i3 + 2] * (1 - items.t) + targetScatter[i3 + 2] * items.t;
        }
        posAttr.needsUpdate = true;
      }
    });
  }
}

function triggerUnlock() {
  isUnlocked.value = true;
  isCharging.value = false;
  instructionText.value = '祈福完成！正在为您加载专属时空长廊 ✨';
  stopChargeAudio();
  playUnlockChime();

  // Trigger Shockwave rings expansion
  if (shockwaveMesh) {
    shockwaveMesh.scale.set(1, 1, 1);
    (shockwaveMesh.material as THREE.Material).opacity = 0.95;
    gsap.to(shockwaveMesh.scale, {
      x: 180,
      y: 180,
      z: 1,
      duration: 1.4,
      ease: 'power2.out'
    });
    gsap.to(shockwaveMesh.material, {
      opacity: 0,
      duration: 1.4,
      ease: 'power2.out'
    });
  }

  if (shockwaveMesh2) {
    shockwaveMesh2.scale.set(1, 1, 1);
    (shockwaveMesh2.material as THREE.Material).opacity = 0.95;
    gsap.to(shockwaveMesh2.scale, {
      x: 140,
      y: 140,
      z: 1,
      duration: 1.6,
      ease: 'power2.out',
      delay: 0.1
    });
    gsap.to(shockwaveMesh2.material, {
      opacity: 0,
      duration: 1.6,
      ease: 'power2.out',
      delay: 0.1
    });
  }

  // Step 1: Big Exploding scatter outwards! (全屏大爆炸)
  const posAttr = particleGeometry.attributes.position;
  const currentPositions = Array.from(posAttr.array as Float32Array);
  const explosionTargets: number[] = [];

  for (let i = 0; i < particleCount; i++) {
    const i3 = i * 3;
    // Project vectors radially outward from center with huge force
    const rx = currentPositions[i3];
    const ry = currentPositions[i3 + 1];
    const rz = currentPositions[i3 + 2];
    
    // Normalize and scale outwards
    const length = Math.sqrt(rx * rx + ry * ry + rz * rz) || 1;
    const force = 300 + Math.random() * 250;
    
    explosionTargets.push(
      (rx / length) * force + (Math.random() - 0.5) * 80,
      (ry / length) * force + (Math.random() - 0.5) * 80,
      (rz / length) * force + (Math.random() - 0.5) * 80
    );
  }

  const itemsExplode = { t: 0 };
  gsap.to(itemsExplode, {
    t: 1,
    duration: 0.8,
    ease: 'power4.out',
    onUpdate: () => {
      const currentArr = posAttr.array as Float32Array;
      for (let i = 0; i < particleCount; i++) {
        const i3 = i * 3;
        currentArr[i3] = currentPositions[i3] * (1 - itemsExplode.t) + explosionTargets[i3] * itemsExplode.t;
        currentArr[i3 + 1] = currentPositions[i3 + 1] * (1 - itemsExplode.t) + explosionTargets[i3 + 1] * itemsExplode.t;
        currentArr[i3 + 2] = currentPositions[i3 + 2] * (1 - itemsExplode.t) + explosionTargets[i3 + 2] * itemsExplode.t;
      }
      posAttr.needsUpdate = true;
    },
    onComplete: () => {
      // Step 2: Particles assemble into the rotating 3D Heart shape!
      const postExplosionPositions = Array.from(posAttr.array as Float32Array);
      const itemsHeart = { t: 0 };

      // Change particle colors to vibrant warm love gradient (animate custom material)
      gsap.to(particleMaterial, {
        size: 7.5,
        duration: 2.0,
        ease: 'power2.out'
      });

      gsap.to(itemsHeart, {
        t: 1,
        duration: 2.5,
        ease: 'elastic.out(1, 0.75)',
        onUpdate: () => {
          const currentArr = posAttr.array as Float32Array;
          for (let i = 0; i < particleCount; i++) {
            const i3 = i * 3;
            currentArr[i3] = postExplosionPositions[i3] * (1 - itemsHeart.t) + targetHeart[i3] * itemsHeart.t;
            currentArr[i3 + 1] = postExplosionPositions[i3 + 1] * (1 - itemsHeart.t) + targetHeart[i3 + 1] * itemsHeart.t;
            currentArr[i3 + 2] = postExplosionPositions[i3 + 2] * (1 - itemsHeart.t) + targetHeart[i3 + 2] * itemsHeart.t;
          }
          posAttr.needsUpdate = true;
        },
        onComplete: () => {
          // Fade in Holographic Message glass Card
          if (glassOverlay.value) {
            glassOverlay.value.style.pointerEvents = 'auto';
            gsap.to(glassOverlay.value, {
              opacity: 1,
              scale: 1,
              duration: 1.2,
              ease: 'power2.out'
            });
          }

          // Step 3: Wait 3.5 seconds, then disperse and auto-advance to TimeTunnel
          setTimeout(() => {
            exitGate();
          }, 3800);
        }
      });
    }
  });
}

function exitGate() {
  // Fade out glass card
  if (glassOverlay.value) {
    gsap.to(glassOverlay.value, {
      opacity: 0,
      scale: 0.8,
      duration: 0.8,
      ease: 'power2.in'
    });
  }

  // Shoot camera forward past particles for gorgeous 3D hyper-drive flight feel
  gsap.to(camera.position, {
    z: -300,
    duration: 1.5,
    ease: 'power3.in',
    onUpdate: () => {
      // Increase opacity decay or zoom blur
      if (particleSystem) {
        particleSystem.rotation.y += 0.08;
      }
    },
    onComplete: () => {
      emit('next');
    }
  });
}
</script>

<style scoped>
/* Scoped overrides to protect the glass overlay from outside styles */
.scale-102 {
  transform: scale(1.02);
}
</style>

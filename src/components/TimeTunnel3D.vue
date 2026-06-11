<template>
  <div 
    class="relative w-full h-full overflow-hidden bg-[#140411] flex flex-col items-center justify-between p-4 hologram-grid"
    @touchstart="onTouchStart"
    @touchmove="onTouchMove"
    @mousedown="onMouseDown"
    @mousemove="onMouseMove"
    @mouseup="onMouseUp"
    @mouseleave="onMouseUp"
  >
    <!-- WebGL Background Canvas -->
    <div ref="canvasContainer" class="absolute inset-0 w-full h-full z-0 pointer-events-none"></div>

    <!-- UI Overlay Layer -->
    <div class="relative z-10 w-full max-w-md mx-auto flex flex-col items-center justify-between h-full pointer-events-none">
      
      <!-- Top Guidance HUD -->
      <div class="w-full flex justify-between items-center mt-6 px-4 bg-black/40 backdrop-blur-md py-3 rounded-xl border border-[#ff5e8c]/20">
        <div class="flex flex-col">
          <span class="font-serif italic text-xs text-[#ffd27a]">Chronicle of Love // 时光记忆长廊</span>
          <span class="text-[11px] text-gray-300">滑动或拖曳 · 伴星光追溯甜蜜瞬间</span>
        </div>
        <!-- Skip button -->
        <button 
          @click="emit('next')" 
          class="pointer-events-auto px-3 py-1.5 text-[11px] rounded border border-[#ff5e8c]/40 text-[#ff5e8c] hover:bg-[#ff5e8c]/20 cursor-pointer transition-all uppercase tracking-widest"
          id="skip_tunnel_btn"
        >
          跳过
        </button>
      </div>

      <!-- Center Space: Milestone Cards Display (Rotated / Floating based on scroll depth index) -->
      <div class="relative w-full flex-grow flex items-center justify-center my-4 overflow-visible">
        <div 
          v-for="(card, index) in timelineCards" 
          :key="card.year"
          class="absolute w-full max-w-[285px] sm:max-w-[310px] p-3 sm:p-4 rounded-2xl bg-black/85 border border-[#ff5e8c]/30 backdrop-blur-lg shadow-[0_0_35px_rgba(255,94,140,0.18)] transition-all duration-700 ease-out flex flex-col items-center pointer-events-auto select-none"
          :class="[getCardClass(index)]"
        >
          <!-- Elegant Badge -->
          <span class="px-2.5 py-0.5 rounded-full text-[9px] font-sans font-bold tracking-widest bg-[#ff5e8c]/15 text-[#ff5e8c] border border-[#ff5e8c]/30 mb-2 sm:mb-3">
            💌 {{ card.year }} // {{ card.phase }}
          </span>

          <!-- Photo Window -->
          <div class="relative w-full aspect-[16/10] sm:aspect-[4/3] rounded-lg overflow-hidden border border-white/10 mb-2 sm:mb-3 bg-neutral-900 group">
            <div class="absolute inset-0 bg-gradient-to-t from-black/50 via-transparent to-transparent pointer-events-none z-10"></div>
            
            <img 
              :src="card.photo" 
              alt="Memory Node Visual" 
              class="w-full h-full object-cover transition-transform duration-1000 group-hover:scale-105 filter saturate-[1.1]" 
              referrerPolicy="no-referrer"
            />
            
            <!-- Corner decorations -->
            <div class="absolute top-2.5 left-2.5 w-2 h-2 border-t border-l border-[#ff5e8c]"></div>
            <div class="absolute top-2.5 right-2.5 w-2 h-2 border-t border-r border-[#ff5e8c]"></div>
            <div class="absolute bottom-2.5 left-2.5 w-2 h-2 border-b border-l border-[#ff5e8c]"></div>
            <div class="absolute bottom-2.5 right-2.5 w-2 h-2 border-b border-r border-[#ff5e8c]"></div>
          </div>
          
          <!-- Memory Title -->
          <h3 class="text-sm sm:text-base font-bold text-white mb-1.5 sm:mb-2 tracking-wide text-center">
            {{ card.title }}
          </h3>

          <!-- Divider -->
          <div class="w-12 h-0.5 bg-[#ffd27a]/60 rounded-full mb-1.5 sm:mb-2"></div>

          <!-- Memory Content -->
          <p class="text-[10px] sm:text-[11px] text-gray-300 leading-relaxed font-sans text-left px-1">
            {{ card.content }}
          </p>

          <!-- Details index helper -->
          <div class="mt-2.5 flex justify-between w-full font-sans text-[8px] text-[#ffd27a]/80 tracking-widest uppercase border-t border-white/5 pt-2">
            <span>STATION 0{{ index + 1 }}</span>
            <span>❤ ALWAYS TOGETHER</span>
          </div>
        </div>

        <!-- Touch Tutorial Indicator -->
        <div 
          v-if="depthPercent < 15"
          class="absolute bottom-8 flex flex-col items-center justify-center animate-bounce pointer-events-none"
        >
          <svg class="w-7 h-7 text-[#ff5e8c] animate-pulse" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M19 14l-7 7m0 0l-7-7m7 7V3" />
          </svg>
          <span class="font-sans text-[10px] tracking-wider text-[#ff5e8c] uppercase mt-1 bg-black/60 px-3 py-1 rounded-full">
            向上轻滑屏幕，追溯甜蜜往事 ❤️
          </span>
        </div>
      </div>

      <!-- Bottom Interactive Controls & progress HUD -->
      <div class="w-full flex flex-col items-center mb-8 px-4 pointer-events-auto">
        <!-- Progress Bar representing camera position depth -->
        <div class="w-full bg-white/5 border border-white/10 rounded-full h-2.5 p-0.5 relative mb-4">
          <div 
            class="h-full bg-gradient-to-r from-[#ff5e8c] via-[#c84095] to-[#ffd27a] rounded-full transition-all duration-100 shadow-[0_0_8px_rgba(255,140,180,0.6)]"
            :style="{ width: `${depthPercent}%` }"
          ></div>
          <div class="absolute -top-5 right-0 font-sans text-[9px] text-gray-300">
            ❤ 相伴的每一步，都是风景
          </div>
        </div>

        <!-- Warp Depth / Actions -->
        <div class="w-full flex justify-between items-center font-sans text-[10px] text-gray-400 mb-2">
          <span>相伴进度 // {{ Math.round(depthPercent) }}%</span>
          <span>时光穿梭 // {{ Math.round(400 - cameraZOffset) }} 光年</span>
        </div>

        <!-- Full Transition button once reached end of tunnel -->
        <button
          v-if="depthPercent >= 98"
          @click="emit('next')"
          class="w-full py-3.5 rounded-xl border border-[#ff5e8c] bg-[#ff5e8c]/10 hover:bg-[#ff5e8c]/20 text-[#ff5e8c] font-bold tracking-widest text-xs uppercase animate-pulse cursor-pointer transform hover:scale-102 transition-all shadow-[0_0_20px_rgba(255,94,140,0.4)]"
          id="tunnel_complete_btn"
        >
          探索终点：开启指尖祈愿 💫
        </button>
      </div>

    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, computed } from 'vue';
import * as THREE from 'three';
import { gsap } from 'gsap';

const emit = defineEmits<{
  (e: 'next'): void;
}>();

// Refs
const canvasContainer = ref<HTMLDivElement | null>(null);

// Starry tunnel and expanded romantic milestones configuration
const timelineCards = [
  {
    year: '2019.05',
    phase: '初识星空',
    title: '时光初遇 · 灵魂引力交错',
    photo: 'https://images.unsplash.com/photo-1506318137071-a8e063b4bec0?auto=format&fit=crop&w=400&q=80',
    content: '在庞大的人群字节流中，缘分悄然编译。那一天，你的眸光如璀璨超新星般撞进我的视窗，终结了我生命长廊的无序底噪，拉开了心动的主程序。',
    depth: 350 // Z goes from 350 to 290
  },
  {
    year: '2020.10',
    phase: '浪漫执手',
    title: '温馨执手 · 彼此轨道同步',
    photo: 'https://images.unsplash.com/photo-1518199266791-5375a83190b7?auto=format&fit=crop&w=400&q=80',
    content: '夕阳街头，晚霞漫步。当你温热的指尖轻轻扣进我的手掌，两个原本独立的生命频率，在那个金色的瞬间完成了完美的双向握手与同步。',
    depth: 290 // Z goes from 290 to 230
  },
  {
    year: '2022.02',
    phase: '携手旅途',
    title: '逃离地心 · 奔赴蔚蓝星海',
    photo: 'https://images.unsplash.com/photo-1529156069898-49953e39b3ac?auto=format&fit=crop&w=400&q=80',
    content: '海潮起伏，微风轻拂。每一个绚烂的落日沙滩都有你最治愈的笑脸，那是我们在漫长时光旅途中编译的最美缓冲帧，将浪漫全量加载。',
    depth: 230 // Z goes from 230 to 170
  },
  {
    year: '2023.11',
    phase: '温馨筑巢',
    title: '橘光筑巢 · 烟火琐碎也甜',
    photo: 'https://images.unsplash.com/photo-1513694203232-719a280e022f?auto=format&fit=crop&w=400&q=80',
    content: '橘色壁灯，咖啡香气，还有猫咪软软的鼾声。属于我们的温馨港湾开始热闹，每个细碎的柴米油盐，都被细心保存进我们的共同记忆库。',
    depth: 170 // Z goes from 170 to 110
  },
  {
    year: '2025.04',
    phase: '誓言誓约',
    title: '宿命加冕 · 交换未来密钥',
    photo: 'https://images.unsplash.com/photo-1469371670807-013ccf25f16a?auto=format&fit=crop&w=400&q=80',
    content: '庄重殿堂下，交换的不只是圆环，更是相守一生的核心 consensus 密钥。执子之手、与子偕老是最神圣的安全认证协议，终生不卸载。',
    depth: 110 // Z goes from 110 to 50
  },
  {
    year: '2026.05',
    phase: '璀璨今朝',
    title: '璀璨周年 · 热爱永无边界',
    photo: 'https://images.unsplash.com/photo-1531747118685-ca8fa6e08806?auto=format&fit=crop&w=400&q=80',
    content: '老婆，祝你生日快乐！你是恒定在我漫长宇宙里的绝对重力重心。今天未来的千万光年里，我对你的偏爱，永远处于全速运行的进程状态。',
    depth: 50 // Z < 50
  }
];

// Interactive depth states
// Z ranges from 400 (start) down to 0 (end)
const cameraZOffset = ref(400); 
const depthPercent = computed(() => {
  return Math.min(Math.max(((400 - cameraZOffset.value) / 380) * 100, 0), 100);
});

// Gesture Tracking state vars
let isDragging = false;
let startY = 0;
let currentY = 0;
let baseZ = 400;

// Three.js items (not in ref to avoid runtime lags)
let scene: THREE.Scene;
let camera: THREE.PerspectiveCamera;
let renderer: THREE.WebGLRenderer;
let animationFrameId: number;
let starsGeometry: THREE.BufferGeometry;
let starsMaterial: THREE.PointsMaterial;
let starsPoints: THREE.Points;
let neonGates: THREE.Group; // Group containing the circular rings

// Nebula and Trails
let nebulaGroup: THREE.Group;
let trailGeometry: THREE.PlaneGeometry;
let trailMaterial: THREE.MeshBasicMaterial;

interface TrailInstance {
  mesh: THREE.Mesh;
  vx: number;
  vy: number;
  vz: number;
  life: number;
  size: number;
}
const activeTrails: TrailInstance[] = [];

onMounted(() => {
  initThree();
  window.addEventListener('resize', handleResize);
});

onUnmounted(() => {
  window.removeEventListener('resize', handleResize);
  cancelAnimationFrame(animationFrameId);

  // Deep release
  if (starsGeometry) starsGeometry.dispose();
  if (starsMaterial) starsMaterial.dispose();
  if (neonGates) {
    neonGates.traverse((child) => {
      if (child instanceof THREE.Mesh) {
        if (child.geometry) child.geometry.dispose();
        if (child.material) {
          if (Array.isArray(child.material)) {
            child.material.forEach((m) => m.dispose());
          } else {
            child.material.dispose();
          }
        }
      }
    });
  }

  if (nebulaGroup) {
    nebulaGroup.traverse((child) => {
      if (child instanceof THREE.Mesh) {
        if (child.geometry) child.geometry.dispose();
        if (child.material) {
          if (Array.isArray(child.material)) {
            child.material.forEach((m) => m.dispose());
          } else {
            child.material.dispose();
          }
        }
      }
    });
  }

  if (trailGeometry) trailGeometry.dispose();
  if (trailMaterial) trailMaterial.dispose();
  activeTrails.forEach((t) => {
    scene.remove(t.mesh);
    if (t.mesh.material) (t.mesh.material as THREE.Material).dispose();
  });
  activeTrails.length = 0;
  
  if (renderer) {
    renderer.dispose();
    renderer.forceContextLoss();
    if (renderer.domElement && canvasContainer.value && canvasContainer.value.contains(renderer.domElement)) {
      canvasContainer.value.removeChild(renderer.domElement);
    }
  }
});

// High-glow particle texture
function createTunnelStarTexture(): THREE.Texture {
  const canvas = document.createElement('canvas');
  canvas.width = 32;
  canvas.height = 32;
  const ctx = canvas.getContext('2d');
  if (ctx) {
    const gradient = ctx.createRadialGradient(16, 16, 0, 16, 16, 16);
    gradient.addColorStop(0, 'rgba(255, 255, 255, 1)');
    gradient.addColorStop(0.2, 'rgba(255, 94, 140, 1)');
    gradient.addColorStop(0.5, 'rgba(255, 210, 122, 0.45)');
    gradient.addColorStop(1, 'rgba(0, 0, 0, 0)');
    ctx.fillStyle = gradient;
    ctx.fillRect(0, 0, 32, 32);
  }
  const texture = new THREE.CanvasTexture(canvas);
  return texture;
}

// Spawns stardust trail particles at mouse client position projected into 3D space
function spawnTrailAt(clientX: number, clientY: number) {
  if (!renderer || !camera) return;
  const rect = renderer.domElement.getBoundingClientRect();
  const mouseX = ((clientX - rect.left) / rect.width) * 2 - 1;
  const mouseY = -((clientY - rect.top) / rect.height) * 2 + 1;
  
  // Project to plane 45 units in front of the camera
  const distance = 45;
  const tempV = new THREE.Vector3(mouseX, mouseY, 0.5);
  tempV.unproject(camera);
  
  const dir = tempV.sub(camera.position).normalize();
  const spawnPos = camera.position.clone().add(dir.multiplyScalar(distance));
  
  const colors = [0xff5e8c, 0xffd27a, 0xc84095];
  for (let i = 0; i < 2; i++) {
    const mat = trailMaterial.clone();
    mat.color.setHex(colors[Math.floor(Math.random() * colors.length)]);
    
    const mesh = new THREE.Mesh(trailGeometry, mat);
    mesh.position.copy(spawnPos);
    mesh.position.x += (Math.random() - 0.5) * 4;
    mesh.position.y += (Math.random() - 0.5) * 4;
    mesh.position.z += (Math.random() - 0.5) * 2;
    mesh.quaternion.copy(camera.quaternion);
    
    scene.add(mesh);
    
    const sizeMod = 1.0 + Math.random() * 1.5;
    mesh.scale.set(sizeMod, sizeMod, sizeMod);
    
    activeTrails.push({
      mesh,
      vx: (Math.random() - 0.5) * 0.4,
      vy: (Math.random() - 0.5) * 0.4,
      vz: (Math.random() - 0.5) * 0.2 - 0.4, // float backwards slightly
      life: 1.0,
      size: sizeMod
    });
  }
}

function initThree() {
  if (!canvasContainer.value) return;

  const width = canvasContainer.value.clientWidth || window.innerWidth;
  const height = canvasContainer.value.clientHeight || window.innerHeight;

  // Scene
  scene = new THREE.Scene();
  scene.fog = new THREE.FogExp2('#140411', 0.0035);

  // Camera settings
  camera = new THREE.PerspectiveCamera(60, width / height, 1, 600);
  camera.position.set(0, 0, cameraZOffset.value);

  // WebGL Renderer settings
  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true, powerPreference: "high-performance" });
  renderer.setSize(width, height);
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
  canvasContainer.value.appendChild(renderer.domElement);

  // 1. Generate cylinder tube particles
  const starCount = 4200;
  starsGeometry = new THREE.BufferGeometry();
  const starPositions = new Float32Array(starCount * 3);

  for (let i = 0; i < starCount; i++) {
    const radius = 45 + Math.random() * 35; 
    const angle = Math.random() * Math.PI * 2;
    const z = (Math.random() * 500) - 50; 

    starPositions[i * 3] = Math.cos(angle) * radius;
    starPositions[i * 3 + 1] = Math.sin(angle) * radius;
    starPositions[i * 3 + 2] = z;
  }

  starsGeometry.setAttribute('position', new THREE.BufferAttribute(starPositions, 3));

  starsMaterial = new THREE.PointsMaterial({
    size: 2.3,
    map: createTunnelStarTexture(),
    transparent: true,
    depthWrite: false,
    blending: THREE.AdditiveBlending,
    opacity: 0.9
  });

  starsPoints = new THREE.Points(starsGeometry, starsMaterial);
  scene.add(starsPoints);

  // 2. Generate Neon glowing rings matching card Z depths
  neonGates = new THREE.Group();
  const ringGeom = new THREE.TorusGeometry(32, 0.3, 8, 32);
  const gateDepths = [350, 290, 230, 170, 110, 50, 20];
  const ringColors = ['#ff5e8c', '#ffd27a', '#c84095', '#ffe08a', '#ff5e8c', '#ffd27a', '#c84095'];

  gateDepths.forEach((zDepth, idx) => {
    const mat = new THREE.MeshBasicMaterial({
      color: new THREE.Color(ringColors[idx]),
      transparent: true,
      opacity: 0.6,
      wireframe: true
    });
    const ringMesh = new THREE.Mesh(ringGeom, mat);
    ringMesh.position.set(0, 0, zDepth);
    neonGates.add(ringMesh);

    // Inner glowing ring lines
    const glowRingGeom = new THREE.TorusGeometry(32.4, 0.05, 4, 32);
    const glowRing = new THREE.Mesh(glowRingGeom, new THREE.MeshBasicMaterial({
      color: new THREE.Color(ringColors[idx]),
      transparent: true,
      opacity: 0.3
    }));
    glowRing.position.set(0, 0, zDepth);
    neonGates.add(glowRing);
  });
  scene.add(neonGates);

  // 3. Generate soft Space Nebula backgrounds
  const createNebulaTexture = (coreColor: string, haloColor: string): THREE.Texture => {
    const canvas = document.createElement('canvas');
    canvas.width = 128;
    canvas.height = 128;
    const ctx = canvas.getContext('2d');
    if (ctx) {
      const gradient = ctx.createRadialGradient(64, 64, 0, 64, 64, 64);
      gradient.addColorStop(0, coreColor);
      gradient.addColorStop(0.6, haloColor);
      gradient.addColorStop(1, 'rgba(0,0,0,0)');
      ctx.fillStyle = gradient;
      ctx.fillRect(0, 0, 128, 128);
    }
    return new THREE.CanvasTexture(canvas);
  };

  nebulaGroup = new THREE.Group();
  const nebulaGeom = new THREE.PlaneGeometry(150, 150);
  const textures = [
    createNebulaTexture('rgba(255, 94, 140, 0.22)', 'rgba(200, 64, 149, 0.08)'),
    createNebulaTexture('rgba(255, 210, 122, 0.18)', 'rgba(255, 94, 140, 0.05)'),
    createNebulaTexture('rgba(200, 64, 149, 0.2)', 'rgba(71, 163, 255, 0.06)')
  ];
  
  const nebulaZCoords = [360, 290, 220, 150, 80];
  nebulaZCoords.forEach((zVal, idx) => {
    const mat = new THREE.MeshBasicMaterial({
      map: textures[idx % textures.length],
      transparent: true,
      depthWrite: false,
      blending: THREE.AdditiveBlending,
      opacity: 0.75
    });
    const mesh = new THREE.Mesh(nebulaGeom, mat);
    mesh.position.set(
      (Math.random() - 0.5) * 55,
      (Math.random() - 0.5) * 55,
      zVal
    );
    mesh.rotation.z = Math.random() * Math.PI * 2;
    nebulaGroup.add(mesh);
  });
  scene.add(nebulaGroup);

  // 4. Trail structures template
  trailGeometry = new THREE.PlaneGeometry(1.6, 1.6);
  trailMaterial = new THREE.MeshBasicMaterial({
    map: createTunnelStarTexture(),
    transparent: true,
    opacity: 0.95,
    blending: THREE.AdditiveBlending,
    depthWrite: false
  });

  // Start loop tick
  animate();
}

function handleResize() {
  if (!canvasContainer.value || !renderer || !camera) return;
  const width = canvasContainer.value.clientWidth;
  const height = canvasContainer.value.clientHeight;
  camera.aspect = width / height;
  camera.updateProjectionMatrix();
  renderer.setSize(width, height);
}

function animate() {
  animationFrameId = requestAnimationFrame(animate);

  // Slow drift rotation of current stars points
  if (starsPoints) {
    starsPoints.rotation.z += 0.0022; 
  }

  // Ring Gates rotate slightly
  if (neonGates) {
    neonGates.children.forEach((gate, idx) => {
      if (idx % 2 === 0) {
        gate.rotation.z += 0.007;
      } else {
        gate.rotation.z -= 0.005;
      }
      gate.position.y += Math.sin(Date.now() * 0.0012 + idx) * 0.012;
    });
  }

  // Rotate Nebula clouds
  if (nebulaGroup) {
    nebulaGroup.children.forEach((mesh, idx) => {
      mesh.rotation.z += 0.0008 * (idx % 2 === 0 ? 1 : -1);
    });
  }

  // Update active stardust trails
  for (let i = activeTrails.length - 1; i >= 0; i--) {
    const t = activeTrails[i];
    t.mesh.position.x += t.vx;
    t.mesh.position.y += t.vy;
    t.mesh.position.z += t.vz;
    t.life -= 0.025; // fade out quickly
    
    if (t.life <= 0) {
      scene.remove(t.mesh);
      if (t.mesh.material) (t.mesh.material as THREE.Material).dispose();
      activeTrails.splice(i, 1);
    } else {
      const scaleS = t.life * t.size;
      t.mesh.scale.set(scaleS, scaleS, scaleS);
      if (t.mesh.material && !Array.isArray(t.mesh.material)) {
        t.mesh.material.opacity = t.life * 0.95;
      }
    }
  }

  // Smoothly interpolate (lerp) camera position inside tunnel towards target offset
  if (camera) {
    camera.position.z += (cameraZOffset.value - camera.position.z) * 0.12;
    camera.position.x = Math.sin(Date.now() * 0.02) * (isDragging ? 0.45 : 0.06);
    camera.position.y = Math.cos(Date.now() * 0.015) * (isDragging ? 0.35 : 0.06);
  }

  renderer.render(scene, camera);
}

// Cards styling & visibility states based on camera depth index
function getCardClass(index: number) {
  const card = timelineCards[index];
  const distance = Math.abs(cameraZOffset.value - card.depth);

  if (distance < 35) {
    // Current Card: High glow, perfectly centered
    return 'opacity-100 scale-100 translate-y-0 rotate-0 z-30 pointer-events-auto border-[#ffd27a]/60 shadow-[0_0_35px_rgba(255,210,122,0.45)]';
  } else if (cameraZOffset.value > card.depth) {
    // Upcoming Card: Placed behind, fading out, scaled down
    return 'opacity-0 scale-75 translate-y-[120px] rotate-[-6deg] z-10 pointer-events-none';
  } else {
    // Passed Card: Zoomed past, rotated upwards, vanished
    return 'opacity-0 scale-125 translate-y-[-140px] rotate-[6deg] z-0 pointer-events-none';
  }
}

// Touch Event Parsing
function onTouchStart(e: TouchEvent) {
  isDragging = true;
  startY = e.touches[0].clientY;
  baseZ = cameraZOffset.value;
}

function onTouchMove(e: TouchEvent) {
  if (!isDragging) return;
  currentY = e.touches[0].clientY;
  const deltaY = currentY - startY;
  const targetZ = baseZ + deltaY * 0.78;
  cameraZOffset.value = Math.min(Math.max(targetZ, 20), 400);

  spawnTrailAt(e.touches[0].clientX, e.touches[0].clientY);
}

// Mouse event support for desktop preview users
function onMouseDown(e: MouseEvent) {
  isDragging = true;
  startY = e.clientY;
  baseZ = cameraZOffset.value;
}

function onMouseMove(e: MouseEvent) {
  if (!isDragging) return;
  currentY = e.clientY;
  const deltaY = currentY - startY;
  const targetZ = baseZ + deltaY * 1.05;
  cameraZOffset.value = Math.min(Math.max(targetZ, 20), 400);

  spawnTrailAt(e.clientX, e.clientY);
}

function onMouseUp() {
  isDragging = false;
}
</script>

<style scoped>
/* Floating/Perspective transformations styling */
.translate-y-0 {
  transform: translateY(0);
}
</style>

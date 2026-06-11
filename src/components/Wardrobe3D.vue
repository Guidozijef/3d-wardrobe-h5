<template>
  <div class="relative w-full h-full overflow-hidden bg-[#140411] flex flex-col items-center justify-between p-4 hologram-grid">
    <!-- WebGL 背景渲染画布 -->
    <div ref="canvasContainer" class="absolute inset-0 w-full h-full z-0"></div>

    <!-- UI 交互层 -->
    <div class="relative z-10 w-full max-w-md mx-auto flex flex-col items-center justify-between h-full pointer-events-none">
      
      <!-- 顶部指示面板 -->
      <div class="w-full text-center mt-6 py-3 border-y border-[#ff5e8c]/20 bg-black/60 backdrop-blur-md px-4 rounded-xl shadow-[0_4px_25px_rgba(255,94,140,0.08)]">
        <div class="font-serif italic text-xs text-[#ffd27a] tracking-[0.2em] mb-1">
          Dream Wardrobe // 梦幻魔镜衣橱
        </div>
        <h2 class="text-base font-bold text-white tracking-wide">
          为您的生日派对挑选一套心愿礼服 ✨
        </h2>
      </div>

      <!-- 中间状态显示 (礼服名称) -->
      <div class="my-auto text-center transform scale-95 md:scale-100 flex flex-col items-center gap-1">
        <h1 class="text-2xl font-black tracking-widest text-transparent bg-clip-text bg-gradient-to-r" :class="[activeThemeData.gradientClass]">
          {{ activeThemeData.name }}
        </h1>
        <p class="text-[11px] text-gray-300 italic tracking-wider max-w-[260px] px-3 py-1 bg-black/45 rounded-full border border-white/5">
          " {{ activeThemeData.description }} "
        </p>
      </div>

      <!-- 底部控制区域 -->
      <div class="w-full flex flex-col items-center gap-3 sm:gap-4 mb-4 sm:mb-8 pointer-events-auto">
        <!-- 礼服卡片选择网格 -->
        <div class="grid grid-cols-3 gap-2 w-full">
          <button
            v-for="theme in themes"
            :key="theme.id"
            @click="selectTheme(theme.id)"
            class="py-2.5 rounded-xl border flex flex-col items-center justify-center gap-1.5 transition-all duration-300 cursor-pointer select-none bg-black/60"
            :class="[
              selectedTheme === theme.id
                ? 'border-[#ff5e8c] text-white shadow-[0_0_15px_rgba(255,94,140,0.3)] scale-105 bg-gradient-to-b from-[#ff5e8c]/10 to-transparent'
                : 'border-white/10 text-gray-400 hover:border-white/20'
            ]"
          >
            <span class="text-xl">{{ theme.icon }}</span>
            <span class="text-[10px] font-bold tracking-wider">{{ theme.label }}</span>
          </button>
        </div>

        <!-- 确认选择按钮 -->
        <button
          @click="confirmSelection"
          class="w-full py-3.5 rounded-xl border border-[#ff5e8c] bg-gradient-to-r from-[#ff5e8c] to-[#c84095] text-white font-bold tracking-widest text-xs uppercase cursor-pointer transform hover:scale-102 transition-all shadow-[0_4px_20px_rgba(255,94,140,0.4)] text-center flex items-center justify-center gap-1.5"
          id="confirm_dress_btn"
        >
          <span>穿上礼服，开启魔法连线 ➔</span>
        </button>
      </div>

    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, computed } from 'vue';
import * as THREE from 'three';
import { gsap } from 'gsap';

// 声明外部事件
const emit = defineEmits<{
  (e: 'next'): void;
  (e: 'select-theme', themeId: 'pink' | 'blue' | 'gold'): void;
}>();

// 礼服主题静态配置
const themes = [
  {
    id: 'pink',
    label: '樱粉物语',
    icon: '🌸',
    name: '樱粉落瓣 · 甜心礼裙',
    description: '漫天飞舞的樱粉花瓣，编织成最甜美的粉色梦境。',
    color: 0xff69b4,
    gradientClass: 'from-[#ff8da1] via-[#ff5e8c] to-[#c84095]'
  },
  {
    id: 'blue',
    label: '星海誓言',
    icon: '🌌',
    name: '星海斑斓 · 极光华服',
    description: '凝聚了北欧极光与璀璨星河，将整片夜空穿在身上。',
    color: 0x47a3ff,
    gradientClass: 'from-[#60e0ff] via-[#47a3ff] to-[#c84095]'
  },
  {
    id: 'gold',
    label: '香槟晨曦',
    icon: '✨',
    name: '晨曦流金 · 奢华华裙',
    description: '清晨的第一缕金色阳光，幻化为高贵而璀璨的礼裙。',
    color: 0xffd27a,
    gradientClass: 'from-[#ffebaf] via-[#ffd27a] to-[#ff9d3b]'
  }
] as const;

// 响应式状态变量
const selectedTheme = ref<'pink' | 'blue' | 'gold'>('pink');
const activeThemeData = computed(() => themes.find((t) => t.id === selectedTheme.value)!);

// Three.js 核心对象（不使用 Vue 代理，保障性能）
const canvasContainer = ref<HTMLDivElement | null>(null);
let scene: THREE.Scene;
let camera: THREE.PerspectiveCamera;
let renderer: THREE.WebGLRenderer;
let animationFrameId: number;

// 3D 礼服组
let dressGroup: THREE.Group;
let skirtMesh: THREE.Mesh;
let originalSkirtPositions: number[] = [];

// 3D 魔镜与装饰
let mirrorMesh: THREE.Mesh;

// 粒子系统
let particleGeometry: THREE.BufferGeometry;
let particleMaterial: THREE.PointsMaterial;
let particleSystem: THREE.Points;
const maxParticles = 600;
let particlePositions: Float32Array;

// Web Audio API 实例
let audioCtx: AudioContext | null = null;

onMounted(() => {
  initThree();
  window.addEventListener('resize', handleResize);
});

onUnmounted(() => {
  window.removeEventListener('resize', handleResize);
  cancelAnimationFrame(animationFrameId);

  // 深度释放 3D 资源
  if (particleGeometry) particleGeometry.dispose();
  if (particleMaterial) particleMaterial.dispose();
  if (dressGroup) {
    dressGroup.traverse((child) => {
      if (child instanceof THREE.Mesh) {
        if (child.geometry) child.geometry.dispose();
        if (child.material) {
          if (Array.isArray(child.material)) child.material.forEach((m) => m.dispose());
          else child.material.dispose();
        }
      }
    });
  }
  if (mirrorMesh) {
    if (mirrorMesh.geometry) mirrorMesh.geometry.dispose();
    if (mirrorMesh.material) (mirrorMesh.material as THREE.Material).dispose();
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

/**
 * 产生动态圆形发光贴图
 */
function createSparkleTexture(): THREE.Texture {
  const canvas = document.createElement('canvas');
  canvas.width = 32;
  canvas.height = 32;
  const ctx = canvas.getContext('2d');
  if (ctx) {
    const gradient = ctx.createRadialGradient(16, 16, 0, 16, 16, 16);
    gradient.addColorStop(0, 'rgba(255, 255, 255, 1)');
    gradient.addColorStop(0.3, 'rgba(255, 94, 140, 0.8)');
    gradient.addColorStop(0.7, 'rgba(255, 255, 255, 0.1)');
    gradient.addColorStop(1, 'rgba(0, 0, 0, 0)');
    ctx.fillStyle = gradient;
    ctx.fillRect(0, 0, 32, 32);
  }
  return new THREE.CanvasTexture(canvas);
}

/**
 * 初始化 Three.js 场景
 */
function initThree() {
  if (!canvasContainer.value) return;

  const width = canvasContainer.value.clientWidth || window.innerWidth;
  const height = canvasContainer.value.clientHeight || window.innerHeight;

  // 1. 场景与雾气
  scene = new THREE.Scene();
  scene.fog = new THREE.FogExp2('#140411', 0.004);

  // 2. 相机配置
  camera = new THREE.PerspectiveCamera(50, width / height, 1, 1000);
  const aspect = width / height;
  camera.position.set(0, 8, 55 * (aspect < 0.75 ? 0.75 / aspect : 1.0));
  camera.lookAt(0, 6, 0);

  // 3. 渲染上下文
  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true, powerPreference: "high-performance" });
  renderer.setSize(width, height);
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
  canvasContainer.value.appendChild(renderer.domElement);

  // 4. 灯光系统
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.4);
  scene.add(ambientLight);

  const dirLight = new THREE.DirectionalLight(0xffffff, 0.8);
  dirLight.position.set(10, 40, 20);
  scene.add(dirLight);

  const pinkSpotLight = new THREE.SpotLight(0xff5e8c, 3, 100, Math.PI / 4, 0.5, 1);
  pinkSpotLight.position.set(-20, 25, 10);
  scene.add(pinkSpotLight);

  // 5. 3D 魔镜背景 (椭圆环)
  const mirrorGeom = new THREE.TorusGeometry(18, 0.4, 16, 100);
  const mirrorMat = new THREE.MeshStandardMaterial({
    color: 0xffd27a,
    roughness: 0.1,
    metalness: 0.8,
    transparent: true,
    opacity: 0.7,
    wireframe: false
  });
  mirrorMesh = new THREE.Mesh(mirrorGeom, mirrorMat);
  mirrorMesh.position.set(0, 7, -5);
  // 压扁成椭圆魔镜
  mirrorMesh.scale.set(1.0, 1.25, 0.2);
  scene.add(mirrorMesh);

  // 6. 3D 礼服参数建模组
  dressGroup = new THREE.Group();
  dressGroup.position.set(0, 0, 0);

  // (A) 裙子吊带/项圈
  const neckGeom = new THREE.TorusGeometry(1.5, 0.25, 12, 24);
  const neckMat = new THREE.MeshStandardMaterial({
    color: activeThemeData.value.color,
    roughness: 0.2,
    metalness: 0.6
  });
  const neck = new THREE.Mesh(neckGeom, neckMat);
  neck.rotation.x = Math.PI / 2;
  neck.position.set(0, 15.5, 0);
  dressGroup.add(neck);

  // (B) 礼裙胸衣 (Bodice)
  const bodiceGeom = new THREE.CylinderGeometry(1.6, 2.4, 4.5, 32, 1);
  const bodiceMat = new THREE.MeshStandardMaterial({
    color: activeThemeData.value.color,
    roughness: 0.4,
    metalness: 0.1
  });
  const bodice = new THREE.Mesh(bodiceGeom, bodiceMat);
  bodice.position.set(0, 13.5, 0);
  dressGroup.add(bodice);

  // (C) 褶皱裙摆 (Skirt) - 圆锥体，开口底部
  const skirtGeom = new THREE.ConeGeometry(7.2, 11, 32, 12, true);
  const skirtMat = new THREE.MeshStandardMaterial({
    color: activeThemeData.value.color,
    roughness: 0.5,
    metalness: 0.1,
    side: THREE.DoubleSide,
    transparent: true,
    opacity: 0.85
  });
  skirtMesh = new THREE.Mesh(skirtGeom, skirtMat);
  skirtMesh.position.set(0, 6.0, 0);
  dressGroup.add(skirtMesh);

  // 记录裙摆原始顶点坐标，以用于正弦波漂浮计算
  const posAttr = skirtGeom.attributes.position;
  originalSkirtPositions = Array.from(posAttr.array as Float32Array);

  scene.add(dressGroup);

  // 7. 梦幻粒子系统
  particleGeometry = new THREE.BufferGeometry();
  particlePositions = new Float32Array(maxParticles * 3);

  // 初始化粒子在周围
  for (let i = 0; i < maxParticles; i++) {
    resetParticle(i);
  }

  particleGeometry.setAttribute('position', new THREE.BufferAttribute(particlePositions, 3));
  particleMaterial = new THREE.PointsMaterial({
    size: 1.5,
    map: createSparkleTexture(),
    transparent: true,
    opacity: 0.85,
    blending: THREE.AdditiveBlending,
    depthWrite: false
  });

  particleSystem = new THREE.Points(particleGeometry, particleMaterial);
  scene.add(particleSystem);

  // 启动渲染循环
  animate();
}

/**
 * 重置单个粒子位置与运动轨迹，基于当前所选主题
 */
function resetParticle(index: number) {
  const i3 = index * 3;
  if (selectedTheme.value === 'pink') {
    // 樱粉落花：上方落至下方，带水平漂移
    particlePositions[i3] = (Math.random() - 0.5) * 35;
    particlePositions[i3 + 1] = 16 + Math.random() * 8;
    particlePositions[i3 + 2] = (Math.random() - 0.5) * 20;
  } else if (selectedTheme.value === 'blue') {
    // 星海星环：在腰间 Z=0 附近做圆环运动
    const angle = Math.random() * Math.PI * 2;
    const r = 9 + Math.random() * 5;
    particlePositions[i3] = Math.cos(angle) * r;
    particlePositions[i3 + 1] = 7.5 + (Math.random() - 0.5) * 6;
    particlePositions[i3 + 2] = Math.sin(angle) * r;
  } else {
    // 香槟晨曦：下方往上升起
    particlePositions[i3] = (Math.random() - 0.5) * 25;
    particlePositions[i3 + 1] = -1 - Math.random() * 4;
    particlePositions[i3 + 2] = (Math.random() - 0.5) * 15;
  }
}

/**
 * 屏幕自适应
 */
function handleResize() {
  if (!canvasContainer.value || !renderer || !camera) return;
  const width = canvasContainer.value.clientWidth;
  const height = canvasContainer.value.clientHeight;
  const aspect = width / height;

  camera.aspect = aspect;
  camera.position.set(0, 8, 55 * (aspect < 0.75 ? 0.75 / aspect : 1.0));
  camera.updateProjectionMatrix();
  renderer.setSize(width, height);
}

/**
 * 3D 帧更新渲染循环
 */
function animate() {
  animationFrameId = requestAnimationFrame(animate);
  const time = Date.now();

  // 1. 魔镜和礼服群做轻微的飘浮震荡
  if (mirrorMesh) {
    mirrorMesh.rotation.y = Math.sin(time * 0.0008) * 0.05;
  }
  if (dressGroup) {
    dressGroup.rotation.y = Math.sin(time * 0.0005) * 0.25;
    dressGroup.position.y = Math.sin(time * 0.0015) * 0.35 + 0.5;
  }

  // 2. 核心算法：通过数学位移计算，模拟裙摆风吹微摆动的柔和物理效果
  if (skirtMesh) {
    const posAttr = skirtMesh.geometry.attributes.position;
    const currentArr = posAttr.array as Float32Array;
    const speed = time * 0.0025;

    for (let i = 0; i < posAttr.count; i++) {
      const i3 = i * 3;
      const originalY = originalSkirtPositions[i3 + 1];

      // 仅形变裙子下摆 (Y坐标 < 0 部分)
      if (originalY < 0.5) {
        const originalX = originalSkirtPositions[i3];
        const originalZ = originalSkirtPositions[i3 + 2];
        const angle = Math.atan2(originalX, originalZ);

        // 基于高度与辐角进行正弦波动位移
        const wave = Math.sin(speed + (originalY * 0.4) + (angle * 2.5)) * 0.45 * (Math.abs(originalY - 5.5) / 11.0);
        const nx = Math.cos(angle);
        const nz = Math.sin(angle);

        currentArr[i3] = originalX + nx * wave;
        currentArr[i3 + 2] = originalZ + nz * wave;
      }
    }
    posAttr.needsUpdate = true;
  }

  // 3. 粒子物理学动态循环模拟
  if (particleSystem) {
    const posAttr = particleGeometry.getAttribute('position') as THREE.BufferAttribute;
    const pArr = posAttr.array as Float32Array;

    for (let i = 0; i < maxParticles; i++) {
      const i3 = i * 3;
      if (selectedTheme.value === 'pink') {
        // 樱粉：缓缓飘落，加上微弱横向晃动
        pArr[i3 + 1] -= 0.09; // 下落
        pArr[i3] += Math.sin(time * 0.001 + i) * 0.02; // 左右微晃
        
        // 落地判定，重新在上方生成
        if (pArr[i3 + 1] < -2) {
          resetParticle(i);
        }
      } else if (selectedTheme.value === 'blue') {
        // 星海：做圆形轨道旋转
        const dx = pArr[i3];
        const dz = pArr[i3 + 2];
        // 极坐标角度计算与微增
        let angle = Math.atan2(dz, dx);
        angle += 0.012; // 旋转角速度
        
        const r = Math.sqrt(dx * dx + dz * dz);
        pArr[i3] = Math.cos(angle) * r;
        pArr[i3 + 2] = Math.sin(angle) * r;
        pArr[i3 + 1] += Math.sin(time * 0.001 + i) * 0.015; // 高度微漂浮
      } else {
        // 晨曦：像香槟气泡垂直向上缓缓升起
        pArr[i3 + 1] += 0.09;
        pArr[i3] += Math.cos(time * 0.0015 + i) * 0.02;
        
        // 飞出边界判定，重新在底部生成
        if (pArr[i3 + 1] > 18) {
          resetParticle(i);
        }
      }
    }
    posAttr.needsUpdate = true;
  }

  renderer.render(scene, camera);
}

/**
 * 播放换装点击反馈音效
 */
function playSwitchSound(vibeId: string) {
  try {
    if (!audioCtx) {
      audioCtx = new (window.AudioContext || (window as any).webkitAudioContext)();
    }
    if (audioCtx.state === 'suspended') {
      audioCtx.resume();
    }

    const osc = audioCtx.createOscillator();
    const gainNode = audioCtx.createGain();

    osc.type = 'triangle';
    let freq = 329.63; // E4
    if (vibeId === 'blue') freq = 392.00; // G4
    else if (vibeId === 'gold') freq = 523.25; // C5

    osc.frequency.setValueAtTime(freq, audioCtx.currentTime);
    gainNode.gain.setValueAtTime(0, audioCtx.currentTime);
    gainNode.gain.linearRampToValueAtTime(0.04, audioCtx.currentTime + 0.008);
    gainNode.gain.exponentialRampToValueAtTime(0.0001, audioCtx.currentTime + 0.3);

    osc.connect(gainNode);
    gainNode.connect(audioCtx.destination);
    osc.start();
    osc.stop(audioCtx.currentTime + 0.35);
  } catch (e) {}
}

/**
 * 播放确认选择成功提示旋律
 */
function playConfirmMelody() {
  try {
    if (!audioCtx) {
      audioCtx = new (window.AudioContext || (window as any).webkitAudioContext)();
    }
    const now = audioCtx.currentTime;
    const sequence = [392.00, 523.25, 659.25, 783.99]; // G4-C5-E5-G5 琶音向上
    sequence.forEach((f, idx) => {
      const osc = audioCtx.createOscillator();
      const gainNode = audioCtx.createGain();
      osc.type = 'triangle';
      osc.frequency.setValueAtTime(f, now + idx * 0.08);

      gainNode.gain.setValueAtTime(0, now + idx * 0.08);
      gainNode.gain.linearRampToValueAtTime(0.04, now + idx * 0.08 + 0.015);
      gainNode.gain.exponentialRampToValueAtTime(0.0001, now + idx * 0.08 + 0.8);

      osc.connect(gainNode);
      gainNode.connect(audioCtx.destination);
      osc.start(now + idx * 0.08);
      osc.stop(now + idx * 0.08 + 0.9);
    });
  } catch (e) {}
}

/**
 * 切换衣橱礼服和背景配色系统
 */
function selectTheme(themeId: 'pink' | 'blue' | 'gold') {
  if (selectedTheme.value === themeId) return;
  selectedTheme.value = themeId;
  playSwitchSound(themeId);

  // 1. GSAP 渐变过度修改礼服色彩
  const targetColor = activeThemeData.value.color;
  dressGroup.traverse((child) => {
    if (child instanceof THREE.Mesh) {
      const mat = child.material as THREE.MeshStandardMaterial;
      if (mat && mat.color) {
        gsap.to(mat.color, {
          r: ((targetColor >> 16) & 255) / 255,
          g: ((targetColor >> 8) & 255) / 255,
          b: (targetColor & 255) / 255,
          duration: 0.8,
          ease: 'power2.out'
        });
      }
    }
  });

  // 2. 修改粒子大小与主色调
  let particleColorHex = 0xffa0b4;
  if (themeId === 'blue') particleColorHex = 0x60e0ff;
  else if (themeId === 'gold') particleColorHex = 0xffebaf;
  
  if (particleMaterial && particleMaterial.color) {
    gsap.to(particleMaterial.color, {
      r: ((particleColorHex >> 16) & 255) / 255,
      g: ((particleColorHex >> 8) & 255) / 255,
      b: (particleColorHex & 255) / 255,
      duration: 0.8,
      ease: 'power2.out'
    });
  }

  // 3. 重置并重新规划粒子的出生分布
  for (let i = 0; i < maxParticles; i++) {
    resetParticle(i);
  }
}

/**
 * 确认选择礼服，派发成功事件
 */
function confirmSelection() {
  playConfirmMelody();

  // 1. 礼服粒子大爆发动画特效
  const posAttr = particleGeometry.getAttribute('position') as THREE.BufferAttribute;
  const pArr = posAttr.array as Float32Array;
  const currentPositions = Array.from(pArr);
  const explosionTargets: number[] = [];

  for (let i = 0; i < maxParticles; i++) {
    const i3 = i * 3;
    const px = currentPositions[i3];
    const py = currentPositions[i3 + 1];
    const pz = currentPositions[i3 + 2];
    
    // 朝外发散扩散
    const length = Math.sqrt(px * px + pz * pz) || 1;
    const speed = 25 + Math.random() * 20;

    explosionTargets.push(
      (px / length) * speed,
      py + (Math.random() - 0.5) * 10,
      (pz / length) * speed
    );
  }

  const items = { t: 0 };
  gsap.to(items, {
    t: 1,
    duration: 0.6,
    ease: 'power2.out',
    onUpdate: () => {
      for (let i = 0; i < maxParticles; i++) {
        const i3 = i * 3;
        pArr[i3] = currentPositions[i3] * (1 - items.t) + explosionTargets[i3] * items.t;
        pArr[i3 + 1] = currentPositions[i3 + 1] * (1 - items.t) + explosionTargets[i3 + 1] * items.t;
        pArr[i3 + 2] = currentPositions[i3 + 2] * (1 - items.t) + explosionTargets[i3 + 2] * items.t;
      }
      posAttr.needsUpdate = true;
    }
  });

  // 2. 缩放相机穿过镜面，平滑切换阶段
  gsap.to(camera.position, {
    z: -10,
    duration: 1.2,
    ease: 'power2.in',
    onComplete: () => {
      // 触发向父级派发所选的主题
      emit('select-theme', selectedTheme.value);
      emit('next');
    }
  });
}
</script>

<style scoped>
.hologram-grid {
  background-image: 
    radial-gradient(circle at 50% 50%, rgba(255, 94, 140, 0.04) 0%, transparent 70%),
    linear-gradient(rgba(255, 255, 255, 0.007) 1px, transparent 1px),
    linear-gradient(90deg, rgba(255, 255, 255, 0.007) 1px, transparent 1px);
  background-size: 100% 100%, 20px 20px, 20px 20px;
}
</style>

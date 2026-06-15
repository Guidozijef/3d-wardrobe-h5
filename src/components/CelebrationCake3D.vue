<template>
  <div class="relative w-full h-full overflow-hidden bg-[#140411] flex flex-col items-center justify-between p-6 hologram-grid">
    
    <!-- 3D 渲染画布容器 -->
    <div ref="canvasContainer" class="absolute inset-0 w-full h-full z-0 cursor-pointer"></div>

    <!-- 上层 UI 交互覆盖层 -->
    <div class="relative z-10 w-full max-w-md mx-auto flex flex-col items-center justify-between h-full pointer-events-none">
      
      <!-- 顶部标题 HUD -->
      <div class="w-full text-center mt-12 py-3 border-y border-[#ffd27a]/25 bg-black/55 backdrop-blur-md px-4 rounded-xl shadow-[0_4px_25px_rgba(255,210,122,0.12)]">
        <div class="font-serif italic text-xs text-[#ffd27a] tracking-[0.2em] mb-1">
          Sweet Birthday Celebration // 生日专属惊喜
        </div>
        <h2 class="text-xl font-bold font-sans tracking-wide text-white drop-shadow-[0_2px_4px_rgba(0,0,0,0.5)]">
          生日蜡烛与心愿星空 🎂
        </h2>
      </div>

      <!-- 提示文字提示：吹熄蜡烛 -->
      <div v-if="!isBlownOut" class="my-auto flex flex-col items-center gap-3 animate-pulse pointer-events-auto">
        <div class="w-16 h-16 rounded-full border border-[#ffd27a] flex items-center justify-center bg-black/60 shadow-[0_0_20px_rgba(255,210,122,0.3)]">
          <svg class="w-8 h-8 text-[#ffd27a] animate-bounce" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.8" d="M13 10V3L4 14h7v7l9-11h-7z" />
          </svg>
        </div>
        <p class="font-sans text-xs text-[#ffd27a] text-center tracking-wider max-w-[260px] bg-black/75 px-4 py-2.5 rounded-xl border border-[#ffd27a]/30">
          ✨ 闭上眼睛许个心愿，然后点击或长按蜡烛火苗“吹熄”它吧！
        </p>
      </div>

      <!-- 蜡烛熄灭后的表白信信封 (淡入) -->
      <transition name="letter-fade">
        <div 
          v-if="isBlownOut"
          class="w-full p-6 my-auto rounded-3xl bg-black/85 border border-[#ffd27a]/45 backdrop-blur-xl shadow-[0_0_50px_rgba(255,210,122,0.25)] flex flex-col items-center justify-center text-center pointer-events-auto transform transition-all duration-1000 scale-100 relative"
        >
          <span class="px-3.5 py-1 rounded-full bg-[#ffd27a]/15 text-[#ffd27a] font-sans text-[10px] tracking-widest border border-[#ffd27a]/40 mb-4 animate-pulse uppercase">
            ❤ Dearest Wife // 挚爱手写信
          </span>

          <!-- 手写信文字区域 -->
          <div class="relative w-full py-4 px-2 font-sans text-left border-l border-r border-[#ffd27a]/20 px-4">
            <p class="text-[13px] text-gray-200 leading-relaxed indent-6">
              亲爱的老婆，祝你生日快乐！
            </p>
            <p class="text-[13px] text-gray-200 leading-relaxed indent-6 mt-2">
              时光在星轨中悄然流淌，而我对你的爱，却被编译成了永恒不变的绝对路径。谢谢你陪伴我度过每一个平凡而又灿烂的日子。
            </p>
            <p class="text-[13px] text-gray-200 leading-relaxed indent-6 mt-2">
              愿你的世界永远充满粉色的花瓣与温柔的星光，无论未来宇宙如何膨胀，我都会永远做你的地心引力，守护在你的身旁。
            </p>
            <p class="text-[11px] text-[#ffd27a] text-right mt-6 font-bold tracking-wider">
              —— 爱你的老公 💖
            </p>

            <!-- 信纸上的印章盖印效果 -->
            <div v-if="isSealed" class="absolute bottom-4 right-10 w-20 h-20 rounded-full border-4 border-red-500/70 flex items-center justify-center font-bold text-red-500 rotate-[-15deg] select-none pointer-events-none animate-[stampZoom_0.35s_ease-out]" style="border-style: double;">
              <div class="text-[8px] leading-tight flex flex-col items-center scale-90">
                <span>★ ★ ★</span>
                <span>老公承诺</span>
                <span class="text-[9px] border-t border-red-500 mt-0.5 pt-0.5">终生有效</span>
              </div>
            </div>
          </div>

          <!-- 签署老公终身保证条约 -->
          <div class="w-full mt-4">
            <button 
              @click="openPromiseModal"
              class="w-full py-2.5 rounded-xl border border-[#ffd27a] bg-[#ffd27a]/5 hover:bg-[#ffd27a]/15 text-[#ffd27a] font-bold text-xs tracking-widest uppercase cursor-pointer transition-all flex items-center justify-center gap-1.5"
              id="open_promise_btn"
            >
              <span>{{ isSealed ? '📝 查看保证书条款' : '📜 签署老公终身保证条约' }}</span>
            </button>
          </div>

          <!-- 重温旅程按钮 -->
          <div class="w-full mt-2">
            <button 
              @click="restartJourney"
              class="w-full py-2.5 rounded-xl border border-[#ff5e8c] bg-gradient-to-r from-[#ff5e8c] to-[#c84095] text-white font-bold tracking-widest text-xs uppercase cursor-pointer transform hover:scale-102 transition-all shadow-[0_4px_20px_rgba(255,94,140,0.3)]"
              id="restart_h5_btn"
            >
              ↩ 重温甜蜜星河旅程
            </button>
          </div>
        </div>
      </transition>

      <!-- 搞笑保证书弹窗 (模态框) -->
      <transition name="modal-fade">
        <div 
          v-if="showPromiseModal" 
          class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-black/85 backdrop-blur-md pointer-events-auto"
        >
          <!-- 保证书主体卡片 -->
          <div 
            class="relative w-full max-w-sm rounded-3xl bg-neutral-950 border border-[#ffd27a]/40 p-6 shadow-[0_0_60px_rgba(255,210,122,0.2)] flex flex-col text-center"
            @click.stop
          >
            <!-- 关闭按钮 -->
            <button 
              @click="closePromiseModal"
              class="absolute top-4 right-4 w-7 h-7 flex items-center justify-center rounded-full bg-white/5 border border-white/10 text-gray-400 hover:text-white transition-colors cursor-pointer"
            >
              ✕
            </button>

            <!-- 保证书标题 -->
            <span class="font-serif italic text-xs text-[#ffd27a] tracking-[0.15em] mb-1">
              Treaty of Sweet Responsibility // 甜蜜盟约
            </span>
            <h3 class="text-lg font-bold text-white tracking-widest mb-4">
              老公终身行为规范保证书
            </h3>

            <!-- 保证条款列表 (一屏全部展示，不使用滚动条) -->
            <div class="space-y-2.5 py-3 border-y border-[#ffd27a]/20 font-sans text-left text-[11px] text-gray-300 leading-normal">
              <div class="flex gap-1.5">
                <span class="text-[#ffd27a] font-mono">01.</span>
                <p><strong>首席赞美官：</strong> 老公每日须变着花样赞美老婆三次以上，老婆换新衣服或发型时须在三秒内给出满分好评。</p>
              </div>
              <div class="flex gap-1.5">
                <span class="text-[#ffd27a] font-mono">02.</span>
                <p><strong>财务绝对主权：</strong> 所有零花钱按时上缴老婆审批，不藏私房钱，老婆看中的心仪礼物必须光速付款。</p>
              </div>
              <div class="flex gap-1.5">
                <span class="text-[#ffd27a] font-mono">03.</span>
                <p><strong>宠溺投降准则：</strong> 吵架时老婆说的一切永远正确！若老婆不开心，老公必须在三分钟内主动哄抱，无条件投降。</p>
              </div>
              <div class="flex gap-1.5">
                <span class="text-[#ffd27a] font-mono">04.</span>
                <p><strong>深夜暖心外卖：</strong> 只要老婆深夜觉得饿，老公必须光速下单或下楼跑腿采购夜宵，双手热气腾腾地呈递床前。</p>
              </div>
              <div class="flex gap-1.5">
                <span class="text-[#ffd27a] font-mono">05.</span>
                <p><strong>专属情绪树洞：</strong> 老婆不开心时，老公必须做满分倾听者，禁止讲任何大道理，坚决且盲目地站在老婆这边！</p>
              </div>
            </div>

            <!-- 印章展示容器 -->
            <div class="relative h-16 w-full flex items-center justify-center mt-2">
              <transition name="stamp-pop">
                <div 
                  v-if="isSealed" 
                  class="absolute z-10 w-24 h-24 rounded-full border-4 border-red-500/80 flex items-center justify-center font-bold text-red-500 rotate-[-12deg] tracking-widest text-center shadow-[0_0_15px_rgba(239,68,68,0.2)] bg-neutral-950/60 backdrop-blur-xs select-none"
                  style="border-style: double;"
                >
                  <div class="text-[10px] leading-tight flex flex-col items-center">
                    <span>★ ★ ★</span>
                    <span>老公承诺</span>
                    <span class="text-xs border-t border-red-500 mt-0.5 pt-0.5">终生有效</span>
                  </div>
                </div>
              </transition>
            </div>

            <!-- 盖章按钮 -->
            <div class="mt-4 flex flex-col gap-2">
              <button 
                v-if="!isSealed"
                @click="applySeal"
                class="w-full py-3 rounded-xl bg-gradient-to-r from-red-600 via-rose-500 to-red-600 text-white font-bold uppercase tracking-wider text-xs active:scale-98 transition-all cursor-pointer text-center animate-pulse shadow-[0_4px_15px_rgba(239,68,68,0.4)]"
                id="apply_seal_btn"
              >
                🛑 一键盖章生效
              </button>
              <button 
                v-else
                @click="closePromiseModal"
                class="w-full py-3 rounded-xl bg-gradient-to-r from-emerald-600 to-teal-600 text-white font-bold uppercase tracking-wider text-xs active:scale-98 transition-all cursor-pointer text-center shadow-[0_4px_15px_rgba(16,185,129,0.3)]"
                id="apply_seal_done_btn"
              >
                ✅ 协议已生效 (放入回忆柜)
              </button>
            </div>
          </div>
        </div>
      </transition>

      <!-- 飘散表情雨容器 -->
      <div class="fixed inset-0 pointer-events-none z-50 overflow-hidden" ref="particlesContainer"></div>

      <!-- 底部版权信息 -->
      <div class="w-full text-center mb-8">
        <span class="font-serif italic text-[10px] text-gray-500 tracking-[0.1em]">
          Happy Birthday To My Queen // 👑
        </span>
      </div>

    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';
import * as THREE from 'three';
import { gsap } from 'gsap';

// 声明外部 Props
const props = withDefaults(defineProps<{
  selectedTheme?: 'pink' | 'blue' | 'gold';
}>(), {
  selectedTheme: 'pink'
});

const emit = defineEmits<{
  (e: 'next'): void;
}>();

const canvasContainer = ref<HTMLDivElement | null>(null);
const isBlownOut = ref(false);

// 保证书模态框状态与盖章状态
const showPromiseModal = ref(false);
const isSealed = ref(false);
const particlesContainer = ref<HTMLDivElement | null>(null);

// Three.js 核心对象（非响应式，保障高性能）
let scene: THREE.Scene;
let camera: THREE.PerspectiveCamera;
let renderer: THREE.WebGLRenderer;
let animationFrameId: number;

// 3D 蛋糕与火焰 Mesh 引用
let cakeGroup: THREE.Group;
let flameMesh: THREE.Mesh | null = null;
let candleMesh: THREE.Mesh;

// 3D Balloons structure
interface BalloonInstance {
  group: THREE.Group;
  vx: number;
  vy: number;
  vz: number;
  rotSpeed: number;
}
const activeBalloons: BalloonInstance[] = [];
let smokeTimer: any = null;
let balloonTimer: any = null;

// 烟花粒子系统核心数据结构
interface FireworkParticle {
  x: number;
  y: number;
  z: number;
  vx: number;
  vy: number;
  vz: number;
  r: number;
  g: number;
  b: number;
  alpha: number;
  size: number;
  gravity: number;
  drag: number;
}
const activeParticles: FireworkParticle[] = [];

// 烟花粒子系统 WebGL Points 配套
let fireworkGeometry: THREE.BufferGeometry;
let fireworkMaterial: THREE.PointsMaterial;
let fireworkPoints: THREE.Points;
const maxFireworkParticles = 4000;
let particlePositions: Float32Array;
let particleColors: Float32Array;

// Web Audio API 实例
let audioCtx: AudioContext | null = null;

// 音符频率数据 map，C大调《祝你生日快乐》
const birthdaySong = [
  { note: 392.00, duration: 0.3 }, // G4
  { note: 392.00, duration: 0.1 }, // G4
  { note: 440.00, duration: 0.4 }, // A4
  { note: 392.00, duration: 0.4 }, // G4
  { note: 523.25, duration: 0.4 }, // C5
  { note: 493.88, duration: 0.8 }, // B4
  
  { note: 392.00, duration: 0.3 }, // G4
  { note: 392.00, duration: 0.1 }, // G4
  { note: 440.00, duration: 0.4 }, // A4
  { note: 392.00, duration: 0.4 }, // G4
  { note: 587.33, duration: 0.4 }, // D5
  { note: 523.25, duration: 0.8 }, // C5
  
  { note: 392.00, duration: 0.3 }, // G4
  { note: 392.00, duration: 0.1 }, // G4
  { note: 783.99, duration: 0.4 }, // G5
  { note: 659.25, duration: 0.4 }, // E5
  { note: 523.25, duration: 0.4 }, // C5
  { note: 493.88, duration: 0.4 }, // B4
  { note: 440.00, duration: 0.8 }, // A4
  
  { note: 698.46, duration: 0.3 }, // F5
  { note: 698.46, duration: 0.1 }, // F5
  { note: 659.25, duration: 0.4 }, // E5
  { note: 523.25, duration: 0.4 }, // C5
  { note: 587.33, duration: 0.4 }, // D5
  { note: 523.25, duration: 0.8 }, // C5
];

onMounted(() => {
  initThree();
  window.addEventListener('resize', handleResize);
  
  // 绑定点击事件，判断是否点击到蜡烛/画布来触发吹熄
  if (renderer && renderer.domElement) {
    renderer.domElement.addEventListener('click', onCanvasClick);
  }
});

onUnmounted(() => {
  window.removeEventListener('resize', handleResize);
  cancelAnimationFrame(animationFrameId);

  // 清除点击监听及定时器
  if (renderer && renderer.domElement) {
    renderer.domElement.removeEventListener('click', onCanvasClick);
  }
  if (smokeTimer) clearInterval(smokeTimer);
  if (balloonTimer) clearInterval(balloonTimer);

  // 释放 WebGL 资源
  if (fireworkGeometry) fireworkGeometry.dispose();
  if (fireworkMaterial) fireworkMaterial.dispose();
  
  if (cakeGroup) {
    cakeGroup.traverse((child) => {
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

  // 释放气球资源
  activeBalloons.forEach((b) => {
    scene.remove(b.group);
    b.group.traverse((child) => {
      if (child instanceof THREE.Mesh) {
        if (child.geometry) child.geometry.dispose();
        if (child.material) {
          if (Array.isArray(child.material)) child.material.forEach(m => m.dispose());
          else child.material.dispose();
        }
      }
    });
  });
  activeBalloons.length = 0;

  if (renderer) {
    renderer.dispose();
    renderer.forceContextLoss();
    if (renderer.domElement && canvasContainer.value && canvasContainer.value.contains(renderer.domElement)) {
      canvasContainer.value.removeChild(renderer.domElement);
    }
  }

  // 关闭音频上下文
  if (audioCtx) {
    audioCtx.close();
  }
});

/**
 * 产生柔和的渐变粒子贴图
 */
function createSparkleTexture(): THREE.Texture {
  const canvas = document.createElement('canvas');
  canvas.width = 32;
  canvas.height = 32;
  const ctx = canvas.getContext('2d');
  if (ctx) {
    const gradient = ctx.createRadialGradient(16, 16, 0, 16, 16, 16);
    gradient.addColorStop(0, 'rgba(255, 255, 255, 1)');
    gradient.addColorStop(0.2, 'rgba(255, 220, 140, 0.9)');
    gradient.addColorStop(0.5, 'rgba(255, 90, 140, 0.4)');
    gradient.addColorStop(1, 'rgba(0, 0, 0, 0)');
    ctx.fillStyle = gradient;
    ctx.fillRect(0, 0, 32, 32);
  }
  return new THREE.CanvasTexture(canvas);
}

/**
 * 初始化 3D 场景与模型
 */
function initThree() {
  if (!canvasContainer.value) return;

  const width = canvasContainer.value.clientWidth || window.innerWidth;
  const height = canvasContainer.value.clientHeight || window.innerHeight;

  // 场景设置
  scene = new THREE.Scene();
  scene.fog = new THREE.FogExp2('#140411', 0.002);

  // 相机设置
  camera = new THREE.PerspectiveCamera(50, width / height, 1, 1000);
  const aspect = width / height;
  const zPos = 150 * (aspect < 0.75 ? 0.75 / aspect : 1.0);
  camera.position.set(0, 50, zPos);
  camera.lookAt(0, 15, 0);

  // 渲染器设置
  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true, powerPreference: "high-performance" });
  renderer.setSize(width, height);
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
  canvasContainer.value.appendChild(renderer.domElement);

  // 灯光配置
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.35);
  scene.add(ambientLight);

  const dirLight = new THREE.DirectionalLight(0xffe2ab, 0.8);
  dirLight.position.set(10, 80, 50);
  scene.add(dirLight);

  const candleLight = new THREE.PointLight(0xff9d3b, 1.5, 120);
  candleLight.position.set(0, 36, 0);
  scene.add(candleLight);

  // ------------------ 绘制 3D 生日蛋糕 ------------------
  cakeGroup = new THREE.Group();

  // 1. 托盘
  const plateGeom = new THREE.CylinderGeometry(32, 34, 1.5, 32);
  const plateMat = new THREE.MeshStandardMaterial({ color: 0xe0e0e0, roughness: 0.1 });
  const plate = new THREE.Mesh(plateGeom, plateMat);
  plate.position.y = 0.75;
  cakeGroup.add(plate);

  // 根据衣橱所选的主题自适应蛋糕奶油和蜡烛颜色
  const theme = props.selectedTheme || 'pink';
  let creamColor = 0xffa0b4; // 草莓粉奶油
  let candleColor = 0x47a3ff; // 蓝色条纹感蜡烛
  if (theme === 'blue') {
    creamColor = 0x7b68ee; // 蓝莓紫奶油
    candleColor = 0xff5e8c; // 粉红色蜡烛
  } else if (theme === 'gold') {
    creamColor = 0xffd27a; // 芒果黄奶油
    candleColor = 0xffa500; // 金橙色蜡烛
  }

  // 2. 蛋糕底部第一层
  const layer1Geom = new THREE.CylinderGeometry(24, 25, 12, 32);
  const layer1Mat = new THREE.MeshStandardMaterial({ 
    color: creamColor,
    roughness: 0.6,
    metalness: 0.1
  });
  const layer1 = new THREE.Mesh(layer1Geom, layer1Mat);
  layer1.position.y = 7.5;
  cakeGroup.add(layer1);

  // 3. 蛋糕底部第二层 (缩小一层)
  const layer2Geom = new THREE.CylinderGeometry(17, 18, 10, 32);
  const layer2Mat = new THREE.MeshStandardMaterial({ 
    color: 0xfffcf0, // 香草白奶油
    roughness: 0.5 
  });
  const layer2 = new THREE.Mesh(layer2Geom, layer2Mat);
  layer2.position.y = 18.5;
  cakeGroup.add(layer2);

  // 4. 巧克力淋边饰品
  const chocolateGeom = new THREE.CylinderGeometry(17.2, 17.2, 1.2, 32);
  const chocolateMat = new THREE.MeshStandardMaterial({ color: 0x5c2e16, roughness: 0.3 });
  const drip = new THREE.Mesh(chocolateGeom, chocolateMat);
  drip.position.y = 23.5;
  cakeGroup.add(drip);

  // 5. 顶部插的草莓 (小圆球表示)
  const strawberryGeom = new THREE.SphereGeometry(1.6, 12, 12);
  const strawberryMat = new THREE.MeshStandardMaterial({ color: 0xd82828, roughness: 0.4 });
  for (let i = 0; i < 8; i++) {
    const angle = (i / 8) * Math.PI * 2;
    const strawberry = new THREE.Mesh(strawberryGeom, strawberryMat);
    strawberry.position.set(Math.cos(angle) * 12, 24.3, Math.sin(angle) * 12);
    // 稍微随机摆放旋转更具真实感
    strawberry.rotation.set(Math.random() * 0.2, 0, Math.random() * 0.2);
    cakeGroup.add(strawberry);
  }

  // 6. 蜡烛
  const candleGeom = new THREE.CylinderGeometry(0.8, 0.8, 10, 16);
  const candleMat = new THREE.MeshStandardMaterial({ color: candleColor, roughness: 0.5 });
  candleMesh = new THREE.Mesh(candleGeom, candleMat);
  candleMesh.position.set(0, 28.5, 0);
  cakeGroup.add(candleMesh);

  // 7. 蜡烛火苗 (采用 Cone 网格模拟)
  const flameGeom = new THREE.ConeGeometry(1.2, 3.8, 12);
  const flameMat = new THREE.MeshBasicMaterial({ 
    color: 0xff9a1f, 
    transparent: true,
    opacity: 0.95
  });
  flameMesh = new THREE.Mesh(flameGeom, flameMat);
  flameMesh.position.set(0, 35.4, 0);
  cakeGroup.add(flameMesh);

  scene.add(cakeGroup);

  // ------------------ 建立烟花粒子系统 WebGL 结构 ------------------
  fireworkGeometry = new THREE.BufferGeometry();
  particlePositions = new Float32Array(maxFireworkParticles * 3);
  particleColors = new Float32Array(maxFireworkParticles * 4); // RGBA

  // 填充默认值
  for (let i = 0; i < maxFireworkParticles; i++) {
    particlePositions[i * 3] = 0;
    particlePositions[i * 3 + 1] = -9999; // 隐藏在视区外
    particlePositions[i * 3 + 2] = 0;

    particleColors[i * 4] = 1.0;
    particleColors[i * 4 + 1] = 1.0;
    particleColors[i * 4 + 2] = 1.0;
    particleColors[i * 4 + 3] = 0.0;
  }

  fireworkGeometry.setAttribute('position', new THREE.BufferAttribute(particlePositions, 3));
  fireworkGeometry.setAttribute('color', new THREE.BufferAttribute(particleColors, 4));

  fireworkMaterial = new THREE.PointsMaterial({
    size: 2.2,
    map: createSparkleTexture(),
    transparent: true,
    depthWrite: false,
    blending: THREE.AdditiveBlending,
    vertexColors: true
  });

  fireworkPoints = new THREE.Points(fireworkGeometry, fireworkMaterial);
  scene.add(fireworkPoints);

  // 开启帧渲染循环
  animate();
}

/**
 * 屏幕缩放适配
 */
function handleResize() {
  if (!canvasContainer.value || !renderer || !camera) return;
  const width = canvasContainer.value.clientWidth;
  const height = canvasContainer.value.clientHeight;
  
  const aspect = width / height;
  camera.aspect = aspect;
  const zPos = 150 * (aspect < 0.75 ? 0.75 / aspect : 1.0);
  camera.position.set(0, 50, zPos);
  camera.updateProjectionMatrix();
  renderer.setSize(width, height);
}

/**
 * 帧渲染循环更新
 */
function animate() {
  animationFrameId = requestAnimationFrame(animate);

  const time = Date.now();

  // 1. 蛋糕缓慢旋转，展示 3D 立体空间感
  if (cakeGroup) {
    cakeGroup.rotation.y = Math.sin(time * 0.0003) * 0.25;
  }

  // 2. 火苗的微风闪烁与顶点波动效果
  if (flameMesh && !isBlownOut.value) {
    const flicker = 0.95 + Math.sin(time * 0.04) * 0.08;
    const waveY = Math.cos(time * 0.06) * 0.15;
    flameMesh.scale.set(flicker, flicker + waveY, flicker);
    
    const pos = flameMesh.geometry.attributes.position;
    const arr = pos.array as Float32Array;
    // 蜡烛火苗顶点摆动
    for (let i = 0; i < pos.count; i++) {
      const yVal = pos.getY(i);
      if (yVal > 0) { // 仅晃动火苗上半部分
        arr[i * 3] = Math.sin(time * 0.035) * 0.12 * yVal; 
        arr[i * 3 + 2] = Math.cos(time * 0.035) * 0.12 * yVal; 
      }
    }
    pos.needsUpdate = true;
  }

  // 3. 气球物理学仿真与循环更新
  for (let i = activeBalloons.length - 1; i >= 0; i--) {
    const b = activeBalloons[i];
    b.group.position.x += b.vx;
    b.group.position.y += b.vy;
    b.group.position.z += b.vz;
    b.group.rotation.y += b.rotSpeed;
    
    // 微弱的左右飘摆
    b.group.position.x += Math.sin(time * 0.0012 + i) * 0.04;
    
    // 飞出视窗后销毁
    if (b.group.position.y > 80) {
      scene.remove(b.group);
      b.group.traverse((child) => {
        if (child instanceof THREE.Mesh) {
          if (child.geometry) child.geometry.dispose();
          if (child.material) (child.material as THREE.Material).dispose();
        }
      });
      activeBalloons.splice(i, 1);
    }
  }

  // 4. 烟花粒子物理仿真与更新
  updateFireworksPhysics();

  renderer.render(scene, camera);
}

/**
 * 烟花粒子物理仿真更新 (重力、摩擦力阻尼、淡出销毁)
 */
function updateFireworksPhysics() {
  const positionsAttr = fireworkGeometry.getAttribute('position') as THREE.BufferAttribute;
  const colorsAttr = fireworkGeometry.getAttribute('color') as THREE.BufferAttribute;
  const posArr = positionsAttr.array as Float32Array;
  const colArr = colorsAttr.array as Float32Array;

  // 周期性从底部自动发射烟花（在吹灭蜡烛后）
  if (isBlownOut.value && Math.random() < 0.045) {
    launchFirework();
  }

  for (let i = 0; i < maxFireworkParticles; i++) {
    const idx3 = i * 3;
    const idx4 = i * 4;

    // 检查粒子是否还活着
    if (colArr[idx4 + 3] > 0) {
      // 获取当前位置与速度
      let px = posArr[idx3];
      let py = posArr[idx3 + 1];
      let pz = posArr[idx3 + 2];

      const p = activeParticles[i];
      if (p) {
        // 更新位置
        px += p.vx;
        py += p.vy;
        pz += p.vz;

        // 重力影响
        p.vy -= p.gravity;

        // 阻力影响
        p.vx *= p.drag;
        p.vy *= p.drag;
        p.vz *= p.drag;

        // 生命周期减少与淡出
        p.alpha -= 0.012;
        if (p.alpha <= 0) {
          p.alpha = 0;
          py = -9999; // 隐藏
        }

        // 写回缓存数组
        posArr[idx3] = px;
        posArr[idx3 + 1] = py;
        posArr[idx3 + 2] = pz;

        colArr[idx4 + 3] = p.alpha;
      }
    }
  }

  positionsAttr.needsUpdate = true;
  colorsAttr.needsUpdate = true;
}

/**
 * 发射一枚烟花，上升至空中，然后爆炸
 */
function launchFirework() {
  // 发射源位置（底部随机偏移）
  const startX = (Math.random() - 0.5) * 50;
  const startZ = (Math.random() - 0.5) * 40;
  const peakHeight = 45 + Math.random() * 35; // 爆炸点高度

  // 选取烟花的随机色彩系
  const colorPalette = [
    { r: 1.0, g: 0.3, b: 0.55 }, // 玫瑰粉
    { r: 1.0, g: 0.82, b: 0.3 }, // 香槟金
    { r: 0.28, g: 0.65, b: 1.0 }, // 天空蓝
    { r: 0.35, g: 0.9, b: 0.55 }, // 翡翠绿
    { r: 0.95, g: 0.95, b: 1.0 }  // 白银光
  ];
  const color = colorPalette[Math.floor(Math.random() * colorPalette.length)];

  // 发射单点引导上升，这里为了简化，我们直接在空中生成爆炸点，并呈球状炸开
  explodeFireworkAt(startX, peakHeight, startZ, color);
}

/**
 * 在指定 3D 坐标处爆炸生成一簇彩色烟花粒子
 */
function explodeFireworkAt(ex: number, ey: number, ez: number, color: { r: number, g: number, b: number }) {
  const particlesCount = 80 + Math.floor(Math.random() * 40);

  let spawned = 0;
  // 查找空闲粒子通道进行填充
  const colorsAttr = fireworkGeometry.getAttribute('color') as THREE.BufferAttribute;
  const colArr = colorsAttr.array as Float32Array;

  for (let i = 0; i < maxFireworkParticles; i++) {
    const idx4 = i * 4;
    // 如果粒子透明度为0，代表该通道空闲
    if (colArr[idx4 + 3] <= 0 && spawned < particlesCount) {
      // 随机分配球形扩散速度矢量
      const theta = Math.random() * Math.PI * 2;
      const phi = Math.acos((Math.random() * 2) - 1);
      const speed = 1.2 + Math.random() * 2.2;

      const vx = Math.sin(phi) * Math.cos(theta) * speed;
      const vy = Math.cos(phi) * speed + 0.4; // 给予轻微向上的初始冲力
      const vz = Math.sin(phi) * Math.sin(theta) * speed;

      // 覆盖当前粒子属性
      activeParticles[i] = {
        x: ex,
        y: ey,
        z: ez,
        vx,
        vy,
        vz,
        r: color.r,
        g: color.g,
        b: color.b,
        alpha: 1.0,
        size: 1.5 + Math.random() * 1.5,
        gravity: 0.03 + Math.random() * 0.025, // 坠落重力加速度
        drag: 0.97 + Math.random() * 0.015     // 空气阻力阻尼
      };

      // 写入 Three.js 顶点颜色缓存区
      colArr[idx4] = color.r;
      colArr[idx4 + 1] = color.g;
      colArr[idx4 + 2] = color.b;
      colArr[idx4 + 3] = 1.0;

      // 写入顶点位置缓存区
      const idx3 = i * 3;
      const posArr = fireworkGeometry.getAttribute('position').array as Float32Array;
      posArr[idx3] = ex;
      posArr[idx3 + 1] = ey;
      posArr[idx3 + 2] = ez;

      spawned++;
    }
  }
}

/**
 * 产生吹熄蜡烛时的烟雾粒子，从蜡烛芯处飘散
 */
function spawnSmokeAtWick() {
  const smokeCount = 3; 
  let spawned = 0;
  const colorsAttr = fireworkGeometry.getAttribute('color') as THREE.BufferAttribute;
  const colArr = colorsAttr.array as Float32Array;
  
  for (let i = 0; i < maxFireworkParticles; i++) {
    const idx4 = i * 4;
    if (colArr[idx4 + 3] <= 0 && spawned < smokeCount) {
      const vx = (Math.random() - 0.5) * 0.25;
      const vy = 0.3 + Math.random() * 0.4;
      const vz = (Math.random() - 0.5) * 0.25;
      
      activeParticles[i] = {
        x: 0,
        y: 35.4,
        z: 0,
        vx,
        vy,
        vz,
        r: 0.75, // 灰白烟雾
        g: 0.75,
        b: 0.8,
        alpha: 0.85,
        size: 1.8 + Math.random() * 2.2,
        gravity: -0.006, // 缓缓上升
        drag: 0.97
      };
      
      colArr[idx4] = 0.75;
      colArr[idx4 + 1] = 0.75;
      colArr[idx4 + 2] = 0.8;
      colArr[idx4 + 3] = 0.85;
      
      const idx3 = i * 3;
      const posArr = fireworkGeometry.getAttribute('position').array as Float32Array;
      posArr[idx3] = 0;
      posArr[idx3 + 1] = 35.4;
      posArr[idx3 + 2] = 0;
      
      spawned++;
    }
  }
}

/**
 * 在 3D 场景中动态生成一个漂浮许愿气球
 */
function spawnBalloon() {
  const balloonGroup = new THREE.Group();
  
  // 气球球体 (Sphere Stretched vertically)
  const bodyGeom = new THREE.SphereGeometry(3.5, 16, 16);
  // 从喜庆浪漫色系中选取
  const colors = [0xff5e8c, 0xffd27a, 0x47a3ff, 0xc84095];
  const chosenColor = colors[Math.floor(Math.random() * colors.length)];
  const bodyMat = new THREE.MeshStandardMaterial({
    color: chosenColor,
    roughness: 0.1,
    metalness: 0.15,
    transparent: true,
    opacity: 0.82
  });
  const body = new THREE.Mesh(bodyGeom, bodyMat);
  body.scale.set(1.0, 1.25, 1.0);
  balloonGroup.add(body);
  
  // 气球底结 (Cone)
  const knotGeom = new THREE.ConeGeometry(0.5, 1.0, 8);
  const knot = new THREE.Mesh(knotGeom, bodyMat);
  knot.position.y = -4.5;
  knot.rotation.z = Math.PI;
  balloonGroup.add(knot);
  
  // 气球拉线 (Line)
  const points = [
    new THREE.Vector3(0, -5.0, 0),
    new THREE.Vector3(0, -13.0, 0)
  ];
  const stringGeom = new THREE.BufferGeometry().setFromPoints(points);
  const stringMat = new THREE.LineBasicMaterial({
    color: 0xcccccc,
    transparent: true,
    opacity: 0.35
  });
  const stringLine = new THREE.Line(stringGeom, stringMat);
  balloonGroup.add(stringLine);
  
  // 随机在底部外边缘分布
  balloonGroup.position.set(
    (Math.random() - 0.5) * 80,
    -40,
    -10 + (Math.random() - 0.5) * 40
  );
  
  scene.add(balloonGroup);
  
  activeBalloons.push({
    group: balloonGroup,
    vx: (Math.random() - 0.5) * 0.15,
    vy: 0.16 + Math.random() * 0.12, // 缓缓升空
    vz: (Math.random() - 0.5) * 0.1,
    rotSpeed: (Math.random() - 0.5) * 0.005
  });
}

/**
 * 监听画布的点击事件，当点击屏幕时，判断是否要吹熄火焰
 */
function onCanvasClick() {
  if (isBlownOut.value) return;

  // 吹熄状态翻转
  isBlownOut.value = true;

  // 1. 移除火苗 Mesh，并播放白烟特效
  if (flameMesh) {
    gsap.to(flameMesh.scale, {
      x: 0,
      y: 0,
      z: 0,
      duration: 0.25,
      ease: 'power2.in',
      onComplete: () => {
        if (flameMesh) scene.remove(flameMesh);
      }
    });
  }

  // 1.5 启动定时器：高频发射蜡烛熄灭的灰白青烟，持续 1.5 秒
  let smokeTicks = 0;
  smokeTimer = setInterval(() => {
    if (smokeTicks < 30) {
      spawnSmokeAtWick();
      smokeTicks++;
    } else {
      clearInterval(smokeTimer);
    }
  }, 50);

  // 1.8 启动定时器：周期性从底部放飞彩色许愿气球
  balloonTimer = setInterval(() => {
    if (isBlownOut.value) {
      spawnBalloon();
    } else {
      clearInterval(balloonTimer);
    }
  }, 1500);

  // 2. 激发一次特大核心烟花爆炸庆祝
  explodeFireworkAt(0, 36, 0, { r: 1.0, g: 0.82, b: 0.3 }); // 蜡烛熄灭点金色烟火
  for (let k = 0; k < 6; k++) {
    setTimeout(() => {
      explodeFireworkAt(
        (Math.random() - 0.5) * 80,
        50 + Math.random() * 30,
        (Math.random() - 0.5) * 50,
        [{ r: 1.0, g: 0.3, b: 0.5 }, { r: 0.3, g: 0.7, b: 1.0 }, { r: 1.0, g: 0.8, b: 0.2 }][k % 3]
      );
    }, 200 + k * 350);
  }

  // 3. 启动 Web Audio 合成播放生日快乐旋律
  playBirthdayMelody();
}

/**
 * 播放合成版《祝你生日快乐》旋律
 */
function playBirthdayMelody() {
  // 懒加载初始化 AudioContext
  if (!audioCtx) {
    audioCtx = new (window.AudioContext || (window as any).webkitAudioContext)();
  }

  if (audioCtx.state === 'suspended') {
    audioCtx.resume();
  }

  const playNote = (freq: number, startTime: number, duration: number) => {
    if (!audioCtx) return;
    
    // 主旋律振荡器
    const osc = audioCtx.createOscillator();
    // 伴奏/泛音振荡器，增加声音饱满感
    const subOsc = audioCtx.createOscillator();
    const gainNode = audioCtx.createGain();

    osc.type = 'triangle'; // 温暖柔和的波形，接近八音盒
    osc.frequency.setValueAtTime(freq, startTime);

    subOsc.type = 'sine'; // 正弦波作为基音共振
    subOsc.frequency.setValueAtTime(freq / 2, startTime); // 低一个八度伴奏

    // 音量包络线：淡入-维持-长衰减
    gainNode.gain.setValueAtTime(0, startTime);
    gainNode.gain.linearRampToValueAtTime(0.065, startTime + 0.03); // 快速 attack
    gainNode.gain.setValueAtTime(0.065, startTime + duration - 0.05);
    gainNode.gain.exponentialRampToValueAtTime(0.0001, startTime + duration); // 优雅的余音 decay

    osc.connect(gainNode);
    subOsc.connect(gainNode);
    gainNode.connect(audioCtx.destination);

    osc.start(startTime);
    subOsc.start(startTime);
    
    osc.stop(startTime + duration + 0.05);
    subOsc.stop(startTime + duration + 0.05);
  };

  const now = audioCtx.currentTime;
  let accumTime = now + 0.15; // 稍作停顿后正式演奏

  // 遍历音符数组顺序调度
  birthdaySong.forEach((item) => {
    // 将持续时间略微放慢，演奏出轻柔舒缓的叙事感觉
    const slowDuration = item.duration * 2.2;
    playNote(item.note, accumTime, slowDuration);
    // 累加时间，加上小幅的休止断奏
    accumTime += slowDuration + 0.08;
  });
}

/**
 * 打开保证书弹窗
 */
function openPromiseModal() {
  showPromiseModal.value = true;
}

/**
 * 关闭保证书弹窗
 */
function closePromiseModal() {
  showPromiseModal.value = false;
}

/**
 * 一键盖章生效
 */
function applySeal() {
  if (isSealed.value) return;
  isSealed.value = true;
  
  // 播放合成重击音效
  playStampSound();
  
  // 激发全屏飘散心心表情雨
  spawnEmojiParticles();
}

/**
 * 使用 Web Audio 合成模拟木质印章敲下重击音效 (低频扫频 + 滤波带通白噪声)
 */
function playStampSound() {
  try {
    if (!audioCtx) {
      audioCtx = new (window.AudioContext || (window as any).webkitAudioContext)();
    }
    if (audioCtx.state === 'suspended') {
      audioCtx.resume();
    }

    const now = audioCtx.currentTime;
    
    // 1. 低频撞击扫频三角波 (150Hz -> 40Hz)
    const osc = audioCtx.createOscillator();
    const gainNode = audioCtx.createGain();
    
    osc.type = 'triangle';
    osc.frequency.setValueAtTime(150, now);
    osc.frequency.exponentialRampToValueAtTime(40, now + 0.15);
    
    gainNode.gain.setValueAtTime(0, now);
    gainNode.gain.linearRampToValueAtTime(0.35, now + 0.01);
    gainNode.gain.exponentialRampToValueAtTime(0.0001, now + 0.25);
    
    osc.connect(gainNode);
    gainNode.connect(audioCtx.destination);
    
    osc.start();
    osc.stop(now + 0.26);

    // 2. 模拟物理摩擦的噪声声部，增加敲击木质感
    const bufferSize = audioCtx.sampleRate * 0.1; 
    const buffer = audioCtx.createBuffer(1, bufferSize, audioCtx.sampleRate);
    const data = buffer.getChannelData(0);
    for (let i = 0; i < bufferSize; i++) {
      data[i] = Math.random() * 2 - 1;
    }
    
    const noiseSource = audioCtx.createBufferSource();
    noiseSource.buffer = buffer;
    
    // 采用带通滤波器让噪声偏向中低频率，模拟纸张震动
    const filter = audioCtx.createBiquadFilter();
    filter.type = 'bandpass';
    filter.frequency.value = 300;
    
    const noiseGain = audioCtx.createGain();
    noiseGain.gain.setValueAtTime(0.12, now);
    noiseGain.gain.exponentialRampToValueAtTime(0.0001, now + 0.08);
    
    noiseSource.connect(filter);
    filter.connect(noiseGain);
    noiseGain.connect(audioCtx.destination);
    
    noiseSource.start();
    noiseSource.stop(now + 0.1);
  } catch (e) {
    console.error('播放盖章音效失败:', e);
  }
}

/**
 * 屏幕中心激发生动的爱心/玫瑰/飞吻表情粒子爆炸扩散，随后随重力优雅飘落
 */
function spawnEmojiParticles() {
  if (!particlesContainer.value) return;
  
  const emojis = ['💖', '🌸', '😘', '✨', '🌹', '💑', '🍬', '🥰', '💌'];
  const count = 35;
  const rect = particlesContainer.value.getBoundingClientRect();
  const centerX = rect.width / 2;
  const centerY = rect.height / 2;

  for (let i = 0; i < count; i++) {
    const el = document.createElement('div');
    el.innerText = emojis[Math.floor(Math.random() * emojis.length)];
    el.style.position = 'absolute';
    el.style.left = `${centerX}px`;
    el.style.top = `${centerY}px`;
    el.style.fontSize = `${16 + Math.random() * 24}px`;
    el.style.userSelect = 'none';
    el.style.pointerEvents = 'none';
    el.style.zIndex = '99';
    particlesContainer.value.appendChild(el);

    // 计算随机喷射角度与距离
    const angle = Math.random() * Math.PI * 2;
    const distance = 80 + Math.random() * 220;
    
    const tl = gsap.timeline({
      onComplete: () => {
        el.remove();
      }
    });

    // 1. 瞬间爆开喷射
    tl.to(el, {
      x: Math.cos(angle) * distance,
      y: Math.sin(angle) * distance,
      rotation: (Math.random() - 0.5) * 360,
      opacity: 1,
      duration: 0.45,
      ease: 'power2.out'
    });

    // 2. 接着重力飘散下落
    tl.to(el, {
      y: rect.height + 40,
      opacity: 0,
      rotation: (Math.random() - 0.5) * 720,
      duration: 2.2 + Math.random() * 1.5,
      ease: 'sine.in'
    }, '+=0.1');
  }
}

/**
 * 重置 H5 回到起点重温
 */
function restartJourney() {
  emit('next');
}
</script>

<style scoped>
/* 信封与表白信面板的渐入过渡动效 */
.letter-fade-enter-active {
  transition: all 1.2s cubic-bezier(0.16, 1, 0.3, 1);
}
.letter-fade-enter-from {
  opacity: 0;
  transform: translate(-50%, 40px) scale(0.92);
}

.hologram-grid {
  background-image: 
    radial-gradient(circle at 50% 50%, rgba(255, 94, 140, 0.04) 0%, transparent 70%),
    linear-gradient(rgba(255, 255, 255, 0.007) 1px, transparent 1px),
    linear-gradient(90deg, rgba(255, 255, 255, 0.007) 1px, transparent 1px);
  background-size: 100% 100%, 20px 20px, 20px 20px;
}

/* 模态框渐入渐出 */
.modal-fade-enter-active, .modal-fade-leave-active {
  transition: opacity 0.35s ease, transform 0.35s ease;
}
.modal-fade-enter-from, .modal-fade-leave-to {
  opacity: 0;
}

/* 印章敲下缩放震动动画 */
@keyframes stampZoom {
  0% {
    transform: scale(3.5) rotate(-45deg);
    opacity: 0;
    filter: blur(4px);
  }
  65% {
    transform: scale(0.9) rotate(-10deg);
    opacity: 0.9;
  }
  85% {
    transform: scale(1.1) rotate(-14deg);
    opacity: 0.95;
  }
  100% {
    transform: scale(1) rotate(-12deg);
    opacity: 1;
    filter: blur(0);
  }
}

/* 弹窗内印章入场 */
.stamp-pop-enter-active {
  animation: stampZoom 0.45s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards;
}

</style>

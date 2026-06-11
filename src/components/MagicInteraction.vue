<template>
  <div 
    class="relative w-full h-full overflow-hidden bg-[#140411] flex flex-col items-center justify-between p-4 hologram-grid"
    @touchmove="onTouchMove"
    @mousemove="onMouseMove"
    @touchstart="onTouchStart"
    @mousedown="onMouseDown"
  >
    <!-- WebGL 全屏画布 -->
    <div ref="canvasContainer" class="absolute inset-0 w-full h-full z-0 cursor-crosshair"></div>

    <!-- 交互层 UI -->
    <div class="relative z-10 w-full max-w-md mx-auto flex flex-col items-center justify-between h-full pointer-events-none">
      
      <!-- 顶部状态指示栏 -->
      <div class="w-full text-center mt-6 py-2.5 border-b border-[#ff5e8c]/20 bg-black/60 backdrop-blur-md px-4 rounded-b-xl shadow-[0_4px_15px_rgba(255,94,140,0.05)]">
        <div class="font-serif italic text-xs text-[#ffd27a] tracking-[0.1em] mb-0.5">
          Constellation Connection // 魔法星心连线
        </div>
        <h2 class="text-sm font-bold text-white tracking-wide flex items-center justify-center gap-1.5">
          <span>✨ 顺次连接星空中的 6 颗星点</span>
          <span class="text-[#ffd27a]">({{ nextNodeIndex }}/6)</span>
        </h2>
      </div>

      <!-- 全息情话信封弹窗 (完成连线后触发) -->
      <div 
        v-if="showLetter"
        class="w-full p-6 my-auto rounded-3xl bg-black/85 border border-[#ff5e8c]/40 backdrop-blur-xl shadow-[0_0_40px_rgba(255,94,140,0.25)] flex flex-col items-center justify-center text-center pointer-events-auto transform transition-all duration-500 scale-100"
      >
        <!-- 标签装饰 -->
        <span class="px-3 py-1 rounded bg-[#ff5e8c]/15 text-[#ff5e8c] font-sans text-[10px] tracking-widest border border-[#ff5e8c]/30 mb-4 animate-pulse uppercase">
          Endless Blessings // 甜蜜星语
        </span>

        <!-- 祝福语显示框 -->
        <div class="relative w-full py-4 px-2 min-h-[100px] flex items-center justify-center">
          <p class="text-[13px] text-gray-200 leading-relaxed font-sans text-left border-l-2 border-[#ffd27a] pl-3">
            {{ isGenerating ? '正在穿越繁星编译温热的心愿祝福...' : textContent }}
          </p>
        </div>

        <!-- 来源元数据 HUD -->
        <div class="w-full flex justify-between items-center bg-white/5 px-3 py-1.5 rounded-lg border border-white/10 text-[9px] font-sans text-gray-400 mt-2 mb-5">
          <span>PORTAL: // {{ letterSource }}</span>
          <span class="text-[#ffd27a] font-bold">● STAY SWEET FOREVER</span>
        </div>

        <!-- 交互按钮 -->
        <div class="w-full flex gap-3">
          <button 
            @click="triggerVibeWish"
            class="flex-1 py-2 rounded-xl text-xs font-bold bg-[#ff5e8c]/15 border border-[#ff5e8c]/40 hover:bg-[#ff5e8c]/35 text-white tracking-wider cursor-pointer font-sans"
            style="transition: all 0.3s;"
            id="regenerate_btn"
          >
            再来一次 💫
          </button>
          <button 
            @click="closeLetterModal"
            class="px-5 py-2 rounded-xl text-xs font-bold bg-white/10 hover:bg-white/20 text-gray-300 tracking-wider cursor-pointer font-sans"
            id="close_letter_btn"
          >
            收起
          </button>
        </div>
      </div>

      <!-- 底部控制与跳转选项 -->
      <div class="w-full flex flex-col items-center gap-3 sm:gap-4 mb-2 sm:mb-6 mt-auto pointer-events-auto">
        <!-- 氛围选择面板 -->
        <div class="w-full bg-black/75 backdrop-blur-md p-3 sm:p-4 rounded-2xl border border-[#ff5e8c]/20 shadow-[0_0_30px_rgba(255,94,140,0.1)]">
          <div class="font-serif italic text-[10px] sm:text-xs text-[#ffd27a] tracking-widest mb-2 sm:mb-3 text-center">
            Select Romantic Atmosphere // 选择心愿氛围
          </div>
          <div class="grid grid-cols-3 gap-1.5 sm:gap-2">
            <button 
              v-for="vibe in vibes" 
              :key="vibe.id"
              @click="changeVibe(vibe.id)"
              class="py-2 sm:py-2.5 rounded-xl text-[10px] sm:text-[11px] font-bold border transition-all cursor-pointer flex flex-col items-center justify-center gap-1 sm:gap-1.5"
              :class="[
                currentVibe === vibe.id 
                  ? 'bg-[#ff5e8c]/25 border-[#ff5e8c] text-white shadow-[0_0_15px_rgba(255,94,140,0.3)]' 
                  : 'bg-white/5 border-white/10 text-gray-400 hover:bg-white/10'
              ]"
            >
              <span class="text-sm">{{ vibe.icon }}</span>
              <span>{{ vibe.name }}</span>
            </button>
          </div>
        </div>

        <!-- 连线完成后才亮起的渐变按钮，提示跳转 -->
        <button
          @click="emit('next')" 
          class="w-full py-3.5 rounded-xl border font-bold tracking-[0.1em] text-xs uppercase cursor-pointer transform hover:scale-102 transition-all text-center flex items-center justify-center gap-2"
          :class="[
            nextNodeIndex >= 6
              ? 'border-[#ff5e8c] bg-gradient-to-r from-[#ff5e8c] to-[#c84095] text-white shadow-[0_4px_25px_rgba(255,94,140,0.4)] animate-pulse'
              : 'border-white/10 bg-neutral-900/50 text-gray-500 shadow-none'
          ]"
          id="skip_to_voucher_btn"
        >
          <span>{{ nextNodeIndex >= 6 ? '前往专属生日愿望兑换券 ➔' : '🔒 连线星空以解锁下一步' }}</span>
        </button>
      </div>

    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, watch } from 'vue';
import * as THREE from 'three';
import { gsap } from 'gsap';

// 声明外部 Props，支持接收衣橱选择的礼服主题
const props = withDefaults(defineProps<{
  selectedTheme?: 'pink' | 'blue' | 'gold';
}>(), {
  selectedTheme: 'pink'
});

const emit = defineEmits<{
  (e: 'next'): void;
  (e: 'select-theme', themeId: 'pink' | 'blue' | 'gold'): void;
}>();

// 氛围配制
const vibes = [
  { id: 'cyberpunk', name: '温馨午后', icon: '☕' },
  { id: 'starry', name: '星空浪漫', icon: '🌌' },
  { id: 'sweet', name: '甜美粉樱', icon: '🌸' }
];

const currentVibe = ref('sweet');
const showLetter = ref(false);
const isGenerating = ref(false);
const textContent = ref('');
const letterSource = ref('Local Hub');

// 连线游戏的核心追踪数据
const nextNodeIndex = ref(0);
let isPressing = false;

// 手指拉丝动态连线 Segment
let activePointerLine: THREE.Line | null = null;

// 6个连线星点的 3D 坐标分布，构成心形
const starNodes = [
  { x: -25, y: 15, z: 0, active: false, frequency: 261.63 }, // C4 - 左上叶
  { x: 0, y: 5, z: 0, active: false, frequency: 293.66 },   // D4 - 中间凹陷
  { x: 25, y: 15, z: 0, active: false, frequency: 329.63 },  // E4 - 右上叶
  { x: 28, y: -12, z: 0, active: false, frequency: 392.00 }, // G4 - 右下沿
  { x: 0, y: -32, z: 0, active: false, frequency: 440.00 },  // A4 - 底部尖端
  { x: -28, y: -12, z: 0, active: false, frequency: 523.25 } // C5 - 左下沿
];

// Three.js 变量 (不使用 ref 以免引起频繁代理开销)
const canvasContainer = ref<HTMLDivElement | null>(null);
let scene: THREE.Scene;
let camera: THREE.PerspectiveCamera;
let renderer: THREE.WebGLRenderer;
let animationFrameId: number;

// 共享的爱心粒子几何体和材质
let heartGeometry: THREE.ShapeGeometry;
let heartMaterial: THREE.MeshBasicMaterial;

// 星点 Mesh 数组
const starMeshes: THREE.Mesh[] = [];
// 连线生成的线段 Mesh 数组
const connectedLines: THREE.Line[] = [];
// 引导虚线 Mesh
let guideLine: THREE.LineLoop | null = null;

// 背景漫天星尘 Mesh
let backgroundPoints: THREE.Points | null = null;

// 音频上下文
let audioCtx: AudioContext | null = null;

// 3D 漂浮爱心实例结构
interface HeartInstance {
  mesh: THREE.Mesh;
  vx: number;
  vy: number;
  vz: number;
  life: number;
  maxScale: number;
  rotSpeedX: number;
  rotSpeedY: number;
  rotSpeedZ: number;
}
const activeHearts: HeartInstance[] = [];

/**
 * 根据外部传入的主题标识 (pink | blue | gold)，获取适配该主题的色彩配置。
 * 包含粒子、引导连线、以及星点在未激活与激活状态下的色彩编码。
 * 遵循谷歌前端编码规范：
 * 1. 采用驼峰式命名。
 * 2. 具有明确的类型注解。
 * 3. 包含详尽的中文业务及实现说明。
 * 
 * @returns {{
 *   particleColor: string;
 *   lineColor: string;
 *   nodeBase: number;
 *   nodeActive: number;
 * }} 包含粒子颜色、线段颜色、基础节点颜色与激活状态节点颜色的配置对象
 */
function getThemeColors(): {
  particleColor: string;
  lineColor: string;
  nodeBase: number;
  nodeActive: number;
} {
  // 从 Props 中提取当前选中的主题，如不存在则默认降级为粉色主题
  const theme = props.selectedTheme || 'pink';
  
  if (theme === 'blue') {
    return {
      particleColor: '#60e0ff', // 浅蓝色高亮粒子
      lineColor: '#47a3ff',     // 主题星轨线条颜色
      nodeBase: 0x1c355e,       // 幽蓝暗沉背景星点色
      nodeActive: 0x47a3ff      // 亮蓝色激活星点色
    };
  } else if (theme === 'gold') {
    return {
      particleColor: '#ffebaf', // 晨曦淡金色粒子
      lineColor: '#ffd27a',     // 温暖阳光金星轨线条色
      nodeBase: 0x543b17,       // 棕褐暗沉背景星点色
      nodeActive: 0xffd27a      // 橙金高亮激活星点色
    };
  } else {
    // 默认返回 pink (樱粉物语) 配色方案
    return {
      particleColor: '#ff69b4', // 浪漫樱粉粒子色
      lineColor: '#ff5e8c',     // 玫粉星轨线条色
      nodeBase: 0x5c1b34,       // 深紫粉暗沉背景星点色
      nodeActive: 0xff5e8c      // 霓虹粉高亮激活星点色
    };
  }
}

/**
 * 屏幕自适应分辨率重构函数。
 * 针对不同终端屏幕长宽比的变化，动态调整相机纵横比及视锥体参数，并通知渲染器重绘尺寸。
 * 遵循谷歌前端编码规范。
 */
function handleResize(): void {
  if (!canvasContainer.value || !renderer || !camera) return;
  const width = canvasContainer.value.clientWidth;
  const height = canvasContainer.value.clientHeight;
  const aspect = width / height;

  camera.aspect = aspect;
  // 视锥自适应：针对窄屏/移动端 (aspect < 0.75) 适当拉远相机距离，以容纳心形轮廓星点
  camera.position.z = 100 * (aspect < 0.75 ? 0.75 / aspect : 1.0);
  camera.updateProjectionMatrix();
  renderer.setSize(width, height);
}

// 监听外部传入的主题变动，动态重设 3D 渲染器中材质颜色，实现全场景无刷新色彩热更新
watch(() => props.selectedTheme, (newTheme) => {
  const colors = getThemeColors();

  // 1. 动态更新背景星尘材质颜色
  if (backgroundPoints && backgroundPoints.material) {
    const mat = backgroundPoints.material as THREE.PointsMaterial;
    gsap.to(mat.color, {
      r: new THREE.Color(colors.lineColor).r,
      g: new THREE.Color(colors.lineColor).g,
      b: new THREE.Color(colors.lineColor).b,
      duration: 0.6,
      ease: 'power2.out'
    });
  }

  // 2. 动态更新引导底衬线圈材质颜色
  if (guideLine && guideLine.material) {
    const mat = guideLine.material as THREE.LineBasicMaterial;
    gsap.to(mat.color, {
      r: new THREE.Color(colors.lineColor).r,
      g: new THREE.Color(colors.lineColor).g,
      b: new THREE.Color(colors.lineColor).b,
      duration: 0.6,
      ease: 'power2.out'
    });
  }

  // 3. 动态更新拉丝画线段材质颜色
  if (activePointerLine && activePointerLine.material) {
    const mat = activePointerLine.material as THREE.LineBasicMaterial;
    gsap.to(mat.color, {
      r: new THREE.Color(colors.lineColor).r,
      g: new THREE.Color(colors.lineColor).g,
      b: new THREE.Color(colors.lineColor).b,
      duration: 0.6,
      ease: 'power2.out'
    });
  }

  // 4. 动态更新共享的爱心粒子材质颜色
  if (heartMaterial) {
    gsap.to(heartMaterial.color, {
      r: new THREE.Color(colors.particleColor).r,
      g: new THREE.Color(colors.particleColor).g,
      b: new THREE.Color(colors.particleColor).b,
      duration: 0.6,
      ease: 'power2.out'
    });
  }

  // 5. 动态更新已生成的连线段颜色
  connectedLines.forEach((line) => {
    if (line.material) {
      const mat = line.material as THREE.LineBasicMaterial;
      gsap.to(mat.color, {
        r: new THREE.Color(colors.lineColor).r,
        g: new THREE.Color(colors.lineColor).g,
        b: new THREE.Color(colors.lineColor).b,
        duration: 0.6,
        ease: 'power2.out'
      });
    }
  });
});

onMounted(() => {
  // 依据传入的主题初始化选中氛围状态
  const initialTheme = props.selectedTheme || 'pink';
  if (initialTheme === 'blue') {
    currentVibe.value = 'starry';
  } else if (initialTheme === 'gold') {
    currentVibe.value = 'cyberpunk';
  } else {
    currentVibe.value = 'sweet';
  }

  initThree();
  window.addEventListener('resize', handleResize);
});

onUnmounted(() => {
  window.removeEventListener('resize', handleResize);
  cancelAnimationFrame(animationFrameId);

  // 清理爱心粒子
  activeHearts.forEach((heart) => {
    scene.remove(heart.mesh);
  });
  activeHearts.length = 0;

  // 清除星点与线段
  starMeshes.forEach((mesh) => scene.remove(mesh));
  starMeshes.length = 0;

  connectedLines.forEach((line) => {
    if (line.geometry) line.geometry.dispose();
    scene.remove(line);
  });
  connectedLines.length = 0;

  if (guideLine) {
    if (guideLine.geometry) guideLine.geometry.dispose();
    scene.remove(guideLine);
  }

  if (activePointerLine) {
    if (activePointerLine.geometry) activePointerLine.geometry.dispose();
    if (activePointerLine.material) (activePointerLine.material as THREE.Material).dispose();
    scene.remove(activePointerLine);
  }

  if (backgroundPoints) {
    if (backgroundPoints.geometry) backgroundPoints.geometry.dispose();
    if (backgroundPoints.material) (backgroundPoints.material as THREE.Material).dispose();
    scene.remove(backgroundPoints);
  }

  if (heartGeometry) heartGeometry.dispose();
  if (heartMaterial) heartMaterial.dispose();
  
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
 * 初始化 Three.js 场景与连线几何
 */
function initThree() {
  if (!canvasContainer.value) return;

  const width = canvasContainer.value.clientWidth || window.innerWidth;
  const height = canvasContainer.value.clientHeight || window.innerHeight;

  // 场景设置
  scene = new THREE.Scene();
  scene.fog = new THREE.FogExp2('#140411', 0.005);

  // 相机设置
  camera = new THREE.PerspectiveCamera(55, width / height, 1, 300);
  const aspect = width / height;
  camera.position.z = 100 * (aspect < 0.75 ? 0.75 / aspect : 1.0);

  // WebGL 渲染上下文
  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true, powerPreference: "high-performance" });
  renderer.setSize(width, height);
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
  canvasContainer.value.appendChild(renderer.domElement);

  // 1. 建立共享的爱心 Shape 几何体
  const shape = new THREE.Shape();
  shape.moveTo(0, 0.5);
  shape.bezierCurveTo(0, 0.45, -0.45, 0.85, -0.8, 0.85);
  shape.bezierCurveTo(-1.3, 0.85, -1.3, 0.25, -1.3, 0.25);
  shape.bezierCurveTo(-1.3, -0.25, -0.8, -0.65, 0, -1.2);
  shape.bezierCurveTo(0.8, -0.65, 1.3, -0.25, 1.3, 0.25);
  shape.bezierCurveTo(1.3, 0.25, 1.3, 0.85, 0.8, 0.85);
  shape.bezierCurveTo(0.45, 0.85, 0, 0.45, 0, 0.5);

  heartGeometry = new THREE.ShapeGeometry(shape);
  
  const colors = getThemeColors();

  // 霓虹发光粒子材质
  heartMaterial = new THREE.MeshBasicMaterial({
    color: colors.particleColor,
    side: THREE.DoubleSide,
    transparent: true,
    opacity: 0.8,
    blending: THREE.AdditiveBlending
  });

  // 2. 添加背景漫天星尘
  const ambientCount = 800;
  const ambientGeom = new THREE.BufferGeometry();
  const ambientPositions = new Float32Array(ambientCount * 3);
  for (let i = 0; i < ambientCount; i++) {
    ambientPositions[i * 3] = (Math.random() - 0.5) * 200;
    ambientPositions[i * 3 + 1] = (Math.random() - 0.5) * 200;
    ambientPositions[i * 3 + 2] = (Math.random() - 0.5) * 150;
  }
  ambientGeom.setAttribute('position', new THREE.BufferAttribute(ambientPositions, 3));
  const ambientMat = new THREE.PointsMaterial({
    color: colors.lineColor,
    size: 0.8,
    transparent: true,
    opacity: 0.35
  });
  backgroundPoints = new THREE.Points(ambientGeom, ambientMat);
  scene.add(backgroundPoints);

  // 3. 在 3D 场景中渲染出 6 个连线的引导星点
  const starGeom = new THREE.SphereGeometry(1.6, 16, 16);
  starNodes.forEach((node, index) => {
    // 默认未激活状态
    const starMat = new THREE.MeshBasicMaterial({
      color: colors.nodeBase,
      transparent: true,
      opacity: 0.65
    });
    const mesh = new THREE.Mesh(starGeom, starMat);
    mesh.position.set(node.x, node.y, node.z);
    scene.add(mesh);
    starMeshes.push(mesh);
  });

  // 4. 绘制一圈极为微弱的虚线引导路线 (形成心形轮廓线)
  const points: THREE.Vector3[] = [];
  starNodes.forEach((node) => {
    points.push(new THREE.Vector3(node.x, node.y, node.z));
  });
  const guideGeom = new THREE.BufferGeometry().setFromPoints(points);
  const guideMat = new THREE.LineBasicMaterial({
    color: colors.lineColor,
    transparent: true,
    opacity: 0.15 // 极淡的底衬虚线引导
  });
  guideLine = new THREE.LineLoop(guideGeom, guideMat);
  scene.add(guideLine);

  // 5. 初始化拉丝动态连线 Segment
  const activePointerGeom = new THREE.BufferGeometry().setFromPoints([
    new THREE.Vector3(0, 0, 0),
    new THREE.Vector3(0, 0, 0)
  ]);
  const activePointerMat = new THREE.LineBasicMaterial({
    color: colors.lineColor,
    transparent: true,
    opacity: 0,
    blending: THREE.AdditiveBlending
  });
  activePointerLine = new THREE.Line(activePointerGeom, activePointerMat);
  // 禁用视锥体裁剪，防止鼠标移动画线时因包围球半径为0导致线段被意外剔除不进行渲染
  activePointerLine.frustumCulled = false;
  scene.add(activePointerLine);

  // 启动渲染循环
  animate();
}

/**
 * 帧渲染循环更新
 */
function animate() {
  animationFrameId = requestAnimationFrame(animate);

  const time = Date.now();
  const colors = getThemeColors();

  // 1. 未连接的星点随时间进行呼吸微缩放与微弱发光变化
  starMeshes.forEach((mesh, idx) => {
    // 下一个待连接的星点闪烁动画，提示用户进行触摸
    if (idx === nextNodeIndex.value) {
      const pulse = 1.25 + Math.sin(time * 0.01) * 0.35;
      mesh.scale.set(pulse, pulse, pulse);
      // @ts-ignore
      mesh.material.color.setHex(colors.nodeActive); // 金黄色/高亮闪烁指示
      // @ts-ignore
      mesh.material.opacity = 0.95;
    } else if (idx < nextNodeIndex.value) {
      // 已激活星点
      mesh.scale.set(1.5, 1.5, 1.5);
      // @ts-ignore
      mesh.material.color.setHex(colors.nodeActive); // 主题激活色
      // @ts-ignore
      mesh.material.opacity = 1.0;
    } else {
      // 待连接的其他星点，保持微弱缩放
      mesh.scale.set(1.0, 1.0, 1.0);
      // @ts-ignore
      mesh.material.color.setHex(colors.nodeBase);
      // @ts-ignore
      mesh.material.opacity = 0.5;
    }
  });

  // 2. 处理处于生命周期中的爱心粒子
  for (let i = activeHearts.length - 1; i >= 0; i--) {
    const heart = activeHearts[i];
    
    // 物理移动
    heart.mesh.position.x += heart.vx;
    heart.mesh.position.y += heart.vy;
    heart.mesh.position.z += heart.vz;

    heart.vx += (Math.random() - 0.5) * 0.04;

    // 自转更新
    heart.mesh.rotation.x += heart.rotSpeedX;
    heart.mesh.rotation.y += heart.rotSpeedY;
    heart.mesh.rotation.z += heart.rotSpeedZ;

    // 衰老计算
    heart.life -= 0.015;
    
    if (heart.life <= 0) {
      scene.remove(heart.mesh);
      activeHearts.splice(i, 1);
    } else {
      // @ts-ignore
      if (heart.mesh.material && !Array.isArray(heart.mesh.material)) {
        // @ts-ignore
        heart.mesh.material.opacity = heart.life * 0.95;
      }
      const scaleS = heart.life * heart.maxScale;
      heart.mesh.scale.set(scaleS, scaleS, scaleS);
    }
  }

  renderer.render(scene, camera);
}

/**
 * 屏幕触摸和点击开始逻辑
 */
function onTouchStart() {
  isPressing = true;
}

function onMouseDown() {
  isPressing = true;
}

function onTouchMove(e: TouchEvent) {
  const touch = e.touches[0];
  isPressing = true;
  handleConnectionTracing(touch.clientX, touch.clientY);
}

function onMouseMove(e: MouseEvent) {
  if (!isPressing) return;
  handleConnectionTracing(e.clientX, e.clientY);
}

// 释放鼠标/手指后判定暂停连线，隐藏画线拉丝
window.addEventListener('mouseup', () => { 
  isPressing = false; 
  if (activePointerLine) (activePointerLine.material as THREE.LineBasicMaterial).opacity = 0;
});
window.addEventListener('touchend', () => { 
  isPressing = false; 
  if (activePointerLine) (activePointerLine.material as THREE.LineBasicMaterial).opacity = 0;
});

/**
 * 核心连线算法：检测当前光标是否接近了目标星点
 */
function handleConnectionTracing(clientX: number, clientY: number) {
  if (!renderer || !camera || nextNodeIndex.value >= 6) return;

  // 将屏幕坐标系投影映射到 3D Z=0 平面
  const rect = renderer.domElement.getBoundingClientRect();
  const mouseX = ((clientX - rect.left) / rect.width) * 2 - 1;
  const mouseY = -((clientY - rect.top) / rect.height) * 2 + 1;

  const planeHeight = 2 * Math.tan((camera.fov * Math.PI) / 360) * camera.position.z;
  const planeWidth = planeHeight * camera.aspect;

  const cursor3DX = mouseX * (planeWidth / 2);
  const cursor3DY = mouseY * (planeHeight / 2);

  // 1. 在拖动轨迹处发射少量心形粒子微光特效
  if (Math.random() < 0.35) {
    spawnHeartsAtPosition(cursor3DX, cursor3DY, 1);
  }

  // 2. 更新拉丝动态连线位置
  if (nextNodeIndex.value > 0 && nextNodeIndex.value < 6 && activePointerLine) {
    const lastNode = starNodes[nextNodeIndex.value - 1];
    const pos = activePointerLine.geometry.attributes.position;
    const arr = pos.array as Float32Array;

    arr[0] = lastNode.x;
    arr[1] = lastNode.y;
    arr[2] = lastNode.z;

    arr[3] = cursor3DX;
    arr[4] = cursor3DY;
    arr[5] = 0;

    pos.needsUpdate = true;
    (activePointerLine.material as THREE.LineBasicMaterial).opacity = 0.85;
  }

  // 3. 检测与待连线目标星点的几何距离
  const targetNode = starNodes[nextNodeIndex.value];
  const dx = cursor3DX - targetNode.x;
  const dy = cursor3DY - targetNode.y;
  const dist = Math.sqrt(dx * dx + dy * dy);

  // 判定距离阈值：若距离小于 8，判定连线碰撞生效！
  if (dist < 8.0) {
    activateNode(nextNodeIndex.value);
  }
}

/**
 * 激活特定的星点，播放提示音并拉起连线
 */
function activateNode(index: number) {
  const node = starNodes[index];
  node.active = true;

  // 1. 播放高品质的水晶电子铃声（Web Audio）
  playCrystalChime(node.frequency);

  // 2. 在被点亮星点处喷洒一小簇爱心烟花
  spawnHeartsAtPosition(node.x, node.y, 12);

  // 3. 画出连接到上一颗星点的明亮 3D 金色连线线段
  if (index > 0) {
    const prevNode = starNodes[index - 1];
    drawGlowingLine(prevNode, node);
  }

  // 4. 累加连线进度索引
  nextNodeIndex.value = index + 1;

  // 5. 校验：当 6 颗星点全部被顺次激活时，判定连线游戏通关！
  if (nextNodeIndex.value === 6) {
    // 完成连线闭环：绘制最终星点 5 回连到星点 0 的闭合线段
    drawGlowingLine(starNodes[5], starNodes[0]);
    
    // 隐藏拉丝线
    if (activePointerLine) (activePointerLine.material as THREE.LineBasicMaterial).opacity = 0;
    
    // 延迟 600ms 自动弹出精美心愿卡片并爆发大烟花喷泉
    setTimeout(() => {
      triggerVibeWish();
    }, 600);
  }
}

/**
 * 播放网页合成的空灵水晶八音盒铃声
 */
function playCrystalChime(freq: number) {
  if (!audioCtx) {
    audioCtx = new (window.AudioContext || (window as any).webkitAudioContext)();
  }
  if (audioCtx.state === 'suspended') {
    audioCtx.resume();
  }

  const osc = audioCtx.createOscillator();
  const oscHarmonic = audioCtx.createOscillator(); // 泛音
  const gainNode = audioCtx.createGain();

  osc.type = 'sine'; // 正弦波纯净感
  osc.frequency.setValueAtTime(freq, audioCtx.currentTime);

  oscHarmonic.type = 'sine';
  oscHarmonic.frequency.setValueAtTime(freq * 2, audioCtx.currentTime); // 谐振

  // 包络线：快速起音，长余震衰减
  gainNode.gain.setValueAtTime(0, audioCtx.currentTime);
  gainNode.gain.linearRampToValueAtTime(0.09, audioCtx.currentTime + 0.008);
  gainNode.gain.exponentialRampToValueAtTime(0.0001, audioCtx.currentTime + 0.95);

  osc.connect(gainNode);
  oscHarmonic.connect(gainNode);
  gainNode.connect(audioCtx.destination);

  osc.start();
  oscHarmonic.start();
  
  osc.stop(audioCtx.currentTime + 1.0);
  oscHarmonic.stop(audioCtx.currentTime + 1.0);
}

/**
 * 在 3D 空间两个坐标点之间画一条粗实的、带有发光饱和度的主题颜色连线段
 */
function drawGlowingLine(nodeA: { x: number, y: number, z: number }, nodeB: { x: number, y: number, z: number }) {
  const colors = getThemeColors();
  const points = [
    new THREE.Vector3(nodeA.x, nodeA.y, nodeA.z),
    new THREE.Vector3(nodeB.x, nodeB.y, nodeB.z)
  ];
  const lineGeom = new THREE.BufferGeometry().setFromPoints(points);
  
  const lineMat = new THREE.LineBasicMaterial({
    color: colors.lineColor,
    transparent: true,
    opacity: 0.95,
    blending: THREE.AdditiveBlending
  });
  const line = new THREE.Line(lineGeom, lineMat);
  scene.add(line);
  connectedLines.push(line);
}

/**
 * 3D 空间特定坐标点发射漂浮上升 of 爱心粒子
 */
function spawnHeartsAtPosition(cx: number, cy: number, count = 1) {
  const colors = getThemeColors();
  for (let s = 0; s < count; s++) {
    // 根据当前选中的衣橱主题配色覆盖发射的爱心颜色
    const mat = new THREE.MeshBasicMaterial({
      color: colors.particleColor,
      side: THREE.DoubleSide,
      transparent: true,
      opacity: 0.85,
      blending: THREE.AdditiveBlending
    });

    const mesh = new THREE.Mesh(heartGeometry, mat);
    mesh.position.set(
      cx + (Math.random() - 0.5) * 4, 
      cy + (Math.random() - 0.5) * 4, 
      (Math.random() - 0.5) * 6
    );
    
    const sizeMod = 1.0 + Math.random() * 2.2;
    mesh.scale.set(sizeMod, sizeMod, sizeMod);
    scene.add(mesh);

    activeHearts.push({
      mesh,
      vx: (Math.random() - 0.5) * 0.9,
      vy: 0.3 + Math.random() * 1.0, // 缓慢浮升
      vz: (Math.random() - 0.5) * 0.5,
      life: 1.0,
      maxScale: sizeMod,
      rotSpeedX: (Math.random() - 0.5) * 0.05,
      rotSpeedY: (Math.random() - 0.5) * 0.06,
      rotSpeedZ: (Math.random() - 0.5) * 0.08
    });
  }
}

/**
 * 3D 喷泉状爱心喷发庆祝动效 (通关星轨时触发)
 */
function triggerCelebrationFountain() {
  const colors = getThemeColors();
  const count = 120;
  for (let i = 0; i < count; i++) {
    const hexColor = Math.random() > 0.4 ? colors.particleColor : colors.lineColor;

    const mat = new THREE.MeshBasicMaterial({
      color: hexColor,
      side: THREE.DoubleSide,
      transparent: true,
      opacity: 0.95,
      blending: THREE.AdditiveBlending
    });

    const mesh = new THREE.Mesh(heartGeometry, mat);
    mesh.position.set((Math.random() - 0.5) * 10, -25 + (Math.random() - 0.5) * 8, (Math.random() - 0.5) * 6);
    scene.add(mesh);

    const sizeMod = 1.4 + Math.random() * 3.5;
    mesh.scale.set(0, 0, 0);

    const angle = (Math.PI * 0.25) + Math.random() * (Math.PI * 0.5); // 向上散射角
    const force = 1.8 + Math.random() * 4.2;

    activeHearts.push({
      mesh,
      vx: Math.cos(angle) * force * (Math.random() > 0.5 ? 1 : -1),
      vy: Math.sin(angle) * force * 1.5,
      vz: (Math.random() - 0.5) * 1.2,
      life: 1.0,
      maxScale: sizeMod,
      rotSpeedX: (Math.random() - 0.5) * 0.12,
      rotSpeedY: (Math.random() - 0.5) * 0.15,
      rotSpeedZ: (Math.random() - 0.5) * 0.2
    });
  }
}

// 本地硬编码的心愿祝福语数据库，分为三种氛围类型：
// 1. cyberpunk: 对应“温馨午后”(☕) 氛围
// 2. starry: 对应“星空浪漫”(🌌) 氛围
// 3. sweet: 对应“甜美粉樱”(🌸) 氛围
const fallbackWishes: Record<string, string[]> = {
  cyberpunk: [
    "在由 1 和 0 构成的无尽比特海中，我的每一次微弱跳动，都指向了与你相遇的唯一频率。老婆，你的眼眸是打碎黑暗底噪的最亮波形，生日快乐！",
    "让全息霓虹为你加冕，数据流闪烁成永恒。在赛博时空的坐标轴中，我对你的爱无限迭代，永不断线。生日快乐，我的专属私钥！",
    "量子坍缩决定了观测的偶然，而我对你的重力吸引是跨越维度的必然。你是程序运行的终极目标，生日快乐老婆！"
  ],
  starry: [
    "我们不过是宇宙里坍缩而成的星光，在亿万光年的漂流里，因重力吸引而相守。老婆，谢谢你成为我漫长生命里的引力中心，生日快乐！",
    "太空寂静，而你就是照亮整片黑暗河系的超新星。在未来的每一个引力波浪潮里，我都想与你手拉手一同冲浪。生日快乐，我永远的心脏原点！",
    "让每一颗璀璨恒星的微光，在今夜折射进你梦乡。老婆，你是写进星云深处的底层密码，生日快乐！"
  ],
  sweet: [
    "老婆，我的世界原本只是一行行冰冷的代码，直到你推开了门，生命才被编译出了幸福的滋味。未来的几万张车票，我都要和你同乘。生日快乐！",
    "偷偷许下愿望：要把银河塞进精美的首饰盒，把世界上所有温柔和宠爱，通通在今晚奉献到你面前。祝我的宝贝老婆生日快乐！",
    "你是我人生底版上最温柔的一抹亮色。只要能看见你的微笑。全世界其他的繁华就都成了精美的背景。生日特别快乐，我的爱人！"
  ]
};

/**
 * 触发星愿魔法，随机获取本地心愿语，并模拟加载仪式感。
 * 遵循谷歌前端编码规范：
 * 1. 采用驼峰命名法。
 * 2. 使用 const 声明不可变变量。
 * 3. 异步函数异常处理完整。
 * 4. 包含详尽的中文注释。
 */
async function triggerVibeWish(): Promise<void> {
  // 触发 3D 喷泉粒子喷发动画
  triggerCelebrationFountain();

  // 显示信封气泡弹窗，并将状态置为生成中
  showLetter.value = true;
  isGenerating.value = true;
  textContent.value = '正在为您寻找心愿流星...';

  // 从本地心愿库中根据选中的氛围风格获取列表，若无匹配则默认使用甜美风格
  const styleKey = currentVibe.value || 'sweet';
  const wishList = fallbackWishes[styleKey] || fallbackWishes.sweet;

  // 随机选择一条祝福语
  const randomIndex = Math.floor(Math.random() * wishList.length);
  const selectedWish = wishList[randomIndex];

  // 模拟大约 800ms 的大模型生成延迟，优化用户的视觉体验与“星愿加载”仪式感
  await new Promise<void>((resolve) => {
    setTimeout(resolve, 800);
  });

  // 填充祝福语内容并设置来源标识
  textContent.value = selectedWish;
  letterSource.value = '本地星愿心意矩阵 v1.0';
  isGenerating.value = false;
}

/**
 * 切换选中的情感氛围
 */
function changeVibe(vibeId: string) {
  currentVibe.value = vibeId;
  
  // 氛围 ID 与全局主题映射：
  // sweet (甜美粉樱) -> pink (樱粉)
  // starry (星空浪漫) -> blue (星海)
  // cyberpunk (温馨午后) -> gold (香槟金)
  let targetTheme: 'pink' | 'blue' | 'gold' = 'pink';
  if (vibeId === 'starry') {
    targetTheme = 'blue';
  } else if (vibeId === 'cyberpunk') {
    targetTheme = 'gold';
  }
  
  // 派发主题变更事件，以通过 Vue 单向数据流实现全局色彩联动
  emit('select-theme', targetTheme);
}

/**
 * 关闭心愿信封弹窗并重置连线游戏
 */
function closeLetterModal() {
  showLetter.value = false;
  resetTracingGame();
}

/**
 * 重置连线游戏的连线路径与线条，便于重玩
 */
function resetTracingGame() {
  nextNodeIndex.value = 0;
  starNodes.forEach((node) => {
    node.active = false;
  });

  // 清除场景中的连线 Mesh
  connectedLines.forEach((line) => {
    scene.remove(line);
    if (line.geometry) line.geometry.dispose();
  });
  connectedLines.length = 0;
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

<template>
  <div class="relative w-full h-full overflow-y-auto bg-[#140411] p-4 flex flex-col justify-between hologram-grid pb-12">
    
    <!-- Top Header Title -->
    <div class="w-full text-center mt-3 mb-5 py-3 bg-black/50 backdrop-blur-md rounded-2xl border" :class="[themeClasses.border]">
      <span class="font-serif italic text-xs text-[#ffd27a] tracking-[0.1em] block mb-1">
        Redemption Center // 愿望兑换大厅
      </span>
      <h2 class="text-base font-bold text-white tracking-wide">
        老婆专属生日礼包 · 幸运指尖抽奖 🎁
      </h2>
    </div>

    <!-- Active Status Banner -->
    <div class="w-full max-w-sm mx-auto mb-4 p-3 bg-black/75 border backdrop-blur-md rounded-xl text-center relative overflow-hidden" :class="[themeClasses.border]">
      <div class="absolute inset-0 bg-gradient-to-r from-[#ff5e8c]/5 via-transparent to-[#ffd27a]/5 pointer-events-none"></div>
      
      <div class="flex justify-between items-center mb-1 text-[9px] font-sans text-gray-400">
        <span>Draw Status // 抽奖状态</span>
        <span class="font-bold font-sans" :class="[themeClasses.text, { 'animate-pulse': isDrawing }]">
          ● {{ isDrawing ? 'DECELERATING // 欢快跳动中' : 'READY // 待命' }}
        </span>
      </div>
      <p class="text-xs text-white tracking-wide font-sans font-medium">
        {{ statusMessage }}
      </p>
    </div>

    <!-- 2x3 LOTTERY RUNNING LIGHTS GRID (PINDUODUO STYLE) -->
    <div class="grid grid-cols-2 gap-2 sm:gap-3.5 w-full max-w-sm mx-auto mb-4 sm:mb-6 relative">
      <div 
        v-for="(coupon, index) in coupons" 
        :key="coupon.id"
        class="relative p-2 sm:p-4 rounded-xl sm:rounded-2xl bg-black/80 border transition-all duration-200 flex flex-col items-center justify-center text-center overflow-hidden h-24 sm:h-28 select-none border-white/5 bg-black/60"
        :class="[
          activeIndex === index
            ? themeClasses.cardActive
            : 'border-white/5 hover:border-white/10 bg-black/60',
          selectedCouponId === coupon.id && !isDrawing
            ? 'ring-2 ring-offset-2 ring-offset-black scale-102 z-10 ' + themeClasses.ringColor
            : ''
        ]"
        @click="selectCouponManually(coupon, index)"
        :id="'coupon_card_' + index"
      >
        <!-- Scanner laser line sweep inside active hopping slot -->
        <div 
          v-if="activeIndex === index"
          class="absolute inset-x-0 h-0.5 top-0 z-10 animate-[scannerSweep_0.8s_infinite_ease-in-out]"
          :class="[themeClasses.laserLine]"
        ></div>

        <!-- Emoticon -->
        <span class="text-3xl mb-1.5 filter drop-shadow-[0_0_6px_rgba(255,255,255,0.25)]">
          {{ coupon.icon }}
        </span>
        
        <!-- Reward Title -->
        <h3 class="text-xs font-bold text-white tracking-wide">
          {{ coupon.name }}
        </h3>

        <!-- Reward Price Subscript -->
        <span class="text-[9px] font-sans text-[#ffd27a] tracking-wider uppercase mt-1 font-medium">
          {{ coupon.price }}
        </span>

        <!-- Glow border rings corner highlights -->
        <div v-if="activeIndex === index" class="absolute top-1 left-1 w-2 h-2 border-t-2 border-l-2" :class="[selectedTheme === 'blue' ? 'border-[#47a3ff]' : selectedTheme === 'gold' ? 'border-[#ffd27a]' : 'border-[#ff5e8c]']"></div>
        <div v-if="activeIndex === index" class="absolute top-1 right-1 w-2 h-2 border-t-2 border-r-2" :class="[selectedTheme === 'blue' ? 'border-[#47a3ff]' : selectedTheme === 'gold' ? 'border-[#ffd27a]' : 'border-[#ff5e8c]']"></div>
        <div v-if="activeIndex === index" class="absolute bottom-1 left-1 w-2 h-2 border-b-2 border-l-2" :class="[selectedTheme === 'blue' ? 'border-[#47a3ff]' : selectedTheme === 'gold' ? 'border-[#ffd27a]' : 'border-[#ff5e8c]']"></div>
        <div v-if="activeIndex === index" class="absolute bottom-1 right-1 w-2 h-2 border-b-2 border-r-2" :class="[selectedTheme === 'blue' ? 'border-[#47a3ff]' : selectedTheme === 'gold' ? 'border-[#ffd27a]' : 'border-[#ff5e8c]']"></div>
      </div>
    </div>

    <!-- CENTRAL GIANT TRIGGER ACTION HUB -->
    <div class="w-full max-w-sm mx-auto mb-4 sm:mb-6 px-1">
      <button 
        @click="startQuantumDraw"
        :disabled="isDrawing"
        class="w-full py-3 sm:py-4 rounded-xl sm:rounded-2xl font-sans font-black text-xs tracking-[0.1em] sm:tracking-[0.2em] text-white uppercase transition-all duration-300 border cursor-pointer select-none active:scale-98 disabled:pointer-events-none disabled:opacity-40"
        :class="[
          isDrawing
            ? 'bg-neutral-900 border-white/5 text-gray-500 shadow-none'
            : themeClasses.gradientBtn
        ]"
        id="trigger_draw_btn"
      >
        {{ isDrawing ? '正在为您抽取甜蜜记忆心愿...' : '💝 开启指尖抽奖魔法' }}
      </button>
    </div>

    <!-- Holographic Receipt Card Popup for Successful Draw / Redemption -->
    <div 
      v-if="redeemedReceipt"
      class="fixed inset-0 bg-[#140411]/95 backdrop-blur-xl z-50 flex flex-col items-center justify-center p-4 animate-fade-in animate-duration-300"
    >
      <!-- Circular Laser scanner light -->
      <div class="absolute inset-x-0 h-0.5 animate-bounce" :class="[themeClasses.laserColor]" style="top: 35%"></div>

      <!-- Main Receipt Card -->
      <div class="w-full max-w-xs sm:max-w-sm rounded-3xl bg-black/95 p-5 sm:p-6 relative flex flex-col border" :class="[themeClasses.receiptBorder]">
        
        <!-- Validation Header Badge -->
        <div class="mx-auto -mt-9 mb-4 px-4 py-1 rounded-full flex items-center gap-2 border shadow-sm" :class="[themeClasses.bgAccent, themeClasses.borderAccent, themeClasses.shadowAccent]">
          <span class="w-2 h-2 rounded-full animate-ping" :class="[themeClasses.barcode]"></span>
          <span class="font-sans text-[10px] tracking-wider font-bold" :class="[themeClasses.text]">
            Redeemed // 已锁定特权 ✨
          </span>
        </div>

        <!-- Receipt title text -->
        <h4 class="text-center text-base font-bold text-white tracking-wider mb-1 mt-2">
          🎉 恭喜老婆中签！
        </h4>
        <p class="text-center font-sans text-[9px] text-[#ffd27a]/80 tracking-widest uppercase mb-5 font-medium">
          验证秘钥: {{ redeemedReceipt.validationCode }}
        </p>

        <!-- Receipt Content Body -->
        <div class="space-y-3.5 py-4 border-y border-dashed border-white/20 font-sans text-xs text-gray-300">
          <div class="flex justify-between">
            <span class="text-gray-500">专属承诺:</span>
            <span class="text-white font-sans text-right font-bold">{{ redeemedReceipt.name }}</span>
          </div>
          <div class="flex justify-between">
            <span class="text-gray-500">签发对象:</span>
            <span class="text-white">最挚爱的宝贝老婆</span>
          </div>
          <div class="flex justify-between">
            <span class="text-gray-500">领取幸福代价:</span>
            <span class="font-bold text-right" :class="[themeClasses.textCost]">{{ redeemedReceipt.price }}</span>
          </div>
          <div class="flex justify-between">
            <span class="text-gray-500">兑换秘钥:</span>
            <span class="text-[#ffd27a] text-right font-mono">{{ redeemedReceipt.key }}</span>
          </div>
          <div class="flex justify-between">
            <span class="text-gray-500">兑换规则:</span>
            <span class="font-semibold" :class="[themeClasses.text]">● ALWAYS ACTIVE // 长期有效</span>
          </div>
          <div class="flex flex-col gap-1 pt-2 border-t border-white/5">
            <span class="text-gray-500">幸福结盟时间:</span>
            <span class="text-gray-400 font-sans text-right text-[10px]">{{ redeemedReceipt.timestamp }}</span>
          </div>
        </div>

        <!-- Procedural barcode representation -->
        <div class="flex flex-col items-center mt-5 gap-1.5">
          <div class="flex items-center gap-0.5 justify-center opacity-70 h-8">
            <div v-for="w in [2,1,4,2,1,3,1,2,5,1,2,3,1,4,1,2]" :key="w" class="h-full" :class="[themeClasses.barcode]" :style="{ width: `${w}px` }"></div>
          </div>
          <span class="font-mono text-[8px] text-gray-500 tracking-[0.25em]">
            * HASH: {{ redeemedReceipt.signature }} *
          </span>
        </div>

        <!-- Confirm action buttons -->
        <div class="mt-5 flex flex-col gap-2">
          <p class="text-[10px] text-gray-400 text-center leading-relaxed font-sans">
            💡 提示：本券已自动保存归档，随时可找老公兑付 💝
          </p>
          <button 
            @click="redeemedReceipt = null"
            class="w-full py-3 mt-2 rounded-xl text-white font-bold uppercase tracking-widest text-xs active:scale-98 transition-all cursor-pointer text-center"
            :class="[themeClasses.gradientConfirm]"
            id="close_receipt_btn"
          >
            放入专属回忆柜
          </button>
        </div>
      </div>
    </div>

    <!-- Final visual bottom button triggers transition to birthday cake stage -->
    <div class="w-full text-center mt-6">
      <button 
        @click="emit('next')" 
        class="pointer-events-auto px-5 py-2 rounded-xl border border-[#ffd27a]/40 text-[10px] tracking-wider uppercase text-[#ffd27a] hover:bg-[#ffd27a]/20 text-center cursor-pointer transition-colors"
        id="rewind_gate_btn"
      >
        前往压轴生日庆典 🎂 ➔
      </button>
    </div>

  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';

const props = withDefaults(defineProps<{
  selectedTheme?: 'pink' | 'blue' | 'gold';
}>(), {
  selectedTheme: 'pink'
});

const emit = defineEmits<{
  (e: 'next'): void;
}>();

// 自适应礼服主题的 UI 样式映射表
const themeClasses = computed(() => {
  const theme = props.selectedTheme || 'pink';
  if (theme === 'blue') {
    return {
      border: 'border-[#47a3ff]/20',
      text: 'text-[#47a3ff]',
      textCost: 'text-[#47a3ff]',
      bgAccent: 'bg-[#47a3ff]/15',
      borderAccent: 'border-[#47a3ff]',
      shadowAccent: 'shadow-[0_0_15px_rgba(71,163,255,0.35)]',
      gradientBtn: 'bg-gradient-to-r from-[#47a3ff] via-[#60e0ff] to-[#c84095] border-[#47a3ff] hover:shadow-[0_4px_25px_rgba(71,163,255,0.55)]',
      cardActive: 'border-[#47a3ff] bg-gradient-to-br from-[#47a3ff]/15 to-transparent shadow-[0_0_25px_rgba(71,163,255,0.45)] scale-105 z-20 animate-[bounceTick_0.15s_infinite_alternate]',
      receiptBorder: 'border-[#47a3ff]/35 shadow-[0_0_50px_rgba(71,163,255,0.15)]',
      barcode: 'bg-[#47a3ff]',
      gradientConfirm: 'bg-gradient-to-r from-[#47a3ff] to-[#60e0ff] shadow-[0_4px_15px_rgba(71,163,255,0.3)]',
      laserColor: 'bg-[#60e0ff] shadow-[0_0_15px_rgba(96,224,255,1)]',
      laserLine: 'bg-[#60e0ff] shadow-[0_0_12px_rgba(96,224,255,1)]',
      ringColor: 'ring-[#47a3ff]'
    };
  } else if (theme === 'gold') {
    return {
      border: 'border-[#ffd27a]/20',
      text: 'text-[#ffd27a]',
      textCost: 'text-[#ffd27a]',
      bgAccent: 'bg-[#ffd27a]/15',
      borderAccent: 'border-[#ffd27a]',
      shadowAccent: 'shadow-[0_0_15px_rgba(255,210,122,0.35)]',
      gradientBtn: 'bg-gradient-to-r from-[#ffd27a] via-[#ff9d3b] to-[#ff5e8c] border-[#ffd27a] hover:shadow-[0_4px_25px_rgba(255,210,122,0.55)]',
      cardActive: 'border-[#ffd27a] bg-gradient-to-br from-[#ffd27a]/15 to-transparent shadow-[0_0_25px_rgba(255,210,122,0.45)] scale-105 z-20 animate-[bounceTick_0.15s_infinite_alternate]',
      receiptBorder: 'border-[#ffd27a]/35 shadow-[0_0_50px_rgba(255,210,122,0.15)]',
      barcode: 'bg-[#ffd27a]',
      gradientConfirm: 'bg-gradient-to-r from-[#ffd27a] to-[#ff9d3b] shadow-[0_4px_15px_rgba(255,210,122,0.3)]',
      laserColor: 'bg-[#ffd27a] shadow-[0_0_15px_rgba(255,210,122,1)]',
      laserLine: 'bg-[#ffd27a] shadow-[0_0_12px_rgba(255,210,122,1)]',
      ringColor: 'ring-[#ffd27a]'
    };
  } else {
    // pink (Sakura Pink)
    return {
      border: 'border-[#ff5e8c]/20',
      text: 'text-[#ff5e8c]',
      textCost: 'text-[#ff5e8c]',
      bgAccent: 'bg-[#ff5e8c]/15',
      borderAccent: 'border-[#ff5e8c]',
      shadowAccent: 'shadow-[0_0_15px_rgba(255,94,140,0.35)]',
      gradientBtn: 'bg-gradient-to-r from-[#ff5e8c] via-[#c84095] to-[#ffd27a] border-[#ff5e8c] hover:shadow-[0_4px_25px_rgba(255,94,140,0.55)]',
      cardActive: 'border-[#ff5e8c] bg-gradient-to-br from-[#ff5e8c]/15 to-transparent shadow-[0_0_25px_rgba(255,94,140,0.45)] scale-105 z-20 animate-[bounceTick_0.15s_infinite_alternate]',
      receiptBorder: 'border-[#ff5e8c]/35 shadow-[0_0_50px_rgba(255,94,140,0.15)]',
      barcode: 'bg-[#ff5e8c]',
      gradientConfirm: 'bg-gradient-to-r from-[#ff5e8c] to-[#c84095] shadow-[0_4px_15px_rgba(255,94,140,0.3)]',
      laserColor: 'bg-[#ffd27a] shadow-[0_0_15px_rgba(255,210,122,1)]',
      laserLine: 'bg-[#ffd27a] shadow-[0_0_12px_rgba(255,210,122,1)]',
      ringColor: 'ring-[#ff5e8c]'
    };
  }
});

// Custom high-priority rewards for the wife to draw
const coupons = [
  {
    id: 1,
    name: '家务免单券',
    price: '免费 // 老公包办',
    code: 'HOME-WIFE-001',
    icon: '🧹'
  },
  {
    id: 2,
    name: '清空购物车',
    price: '秒付 // 宠溺度满配',
    code: 'PAY-CART-999',
    icon: '🛒'
  },
  {
    id: 3,
    name: '热吻治愈券',
    price: '免费 // 随叫随到',
    code: 'WARM-HUG-888',
    icon: '🤗'
  },
  {
    id: 4,
    name: '神秘大礼券',
    price: '保密 // 特别大惊喜',
    code: 'MYSTERY-GIFT-SOL',
    icon: '🎁'
  },
  {
    id: 5,
    name: '定制大餐券',
    price: '点单 // 食材尽享奢华',
    code: 'CHEF-GRAND-005',
    icon: '🍳'
  },
  {
    id: 6,
    name: '浪漫旅行券',
    price: '即刻出发 // 终生兑现',
    code: 'TRIP-PLAN-2026',
    icon: '✈️'
  }
];

// Active index states
const activeIndex = ref(-1);
const selectedCouponId = ref(1);

const isDrawing = ref(false);
const statusMessage = ref('❤️ 立即点击底部按钮开启爱之跳动抽奖 🌟');

interface ReceiptSchema {
  name: string;
  price: string;
  key: string;
  validationCode: string;
  timestamp: string;
  signature: string;
}

const redeemedReceipt = ref<ReceiptSchema | null>(null);

// Manual card click detail browse
function selectCouponManually(coupon: any, index: number) {
  if (isDrawing.value) return;
  selectedCouponId.value = coupon.id;
  activeIndex.value = index;
  statusMessage.value = `已锁定心愿：[${coupon.name}]，你可以点击【立即抽奖】按钮开启快乐的跳轮！`;
}

// Tick Audio feedback
function playTickSound(tickCount: number) {
  try {
    const audioCtx = new (window.AudioContext || (window as any).webkitAudioContext)();
    const osc = audioCtx.createOscillator();
    const gainNode = audioCtx.createGain();
    
    osc.type = 'sine';
    // Frequency shifts upwards as we go faster and then slows
    osc.frequency.setValueAtTime(650 + (tickCount * 12), audioCtx.currentTime); 
    
    gainNode.gain.setValueAtTime(0, audioCtx.currentTime);
    gainNode.gain.linearRampToValueAtTime(0.015, audioCtx.currentTime + 0.005);
    gainNode.gain.exponentialRampToValueAtTime(0.0001, audioCtx.currentTime + 0.04);
    
    osc.connect(gainNode);
    gainNode.connect(audioCtx.destination);
    
    osc.start();
    osc.stop(audioCtx.currentTime + 0.05);
  } catch (err) {}
}

// Success chords
function playSuccessChord() {
  try {
    const audioCtx = new (window.AudioContext || (window as any).webkitAudioContext)();
    const freqs = [523.25, 659.25, 783.99, 1046.50];
    
    freqs.forEach((f, idx) => {
      const osc = audioCtx.createOscillator();
      const gainNode = audioCtx.createGain();
      osc.type = 'triangle';
      osc.frequency.setValueAtTime(f, audioCtx.currentTime + idx * 0.08);
      
      gainNode.gain.setValueAtTime(0, audioCtx.currentTime);
      gainNode.gain.linearRampToValueAtTime(0.035, audioCtx.currentTime + idx * 0.08 + 0.03);
      gainNode.gain.exponentialRampToValueAtTime(0.0001, audioCtx.currentTime + 1.6);
      
      osc.connect(gainNode);
      gainNode.connect(audioCtx.destination);
      osc.start();
      osc.stop(audioCtx.currentTime + 1.9);
    });
  } catch (e) {}
}

// High-tension PinDuoDuo running-horse grid-jumping loop
const hopSequence = [0, 1, 3, 5, 4, 2];

function startQuantumDraw() {
  if (isDrawing.value) return;
  isDrawing.value = true;
  redeemedReceipt.value = null;

  const targetWinnerId = Math.floor(Math.random() * coupons.length); // target index in coupons
  // Find where this target resides in the hopSequence
  const targetSequenceIndex = hopSequence.indexOf(targetWinnerId);

  const laps = 5; // number of full loop rotations around grid
  const totalSteps = laps * hopSequence.length + targetSequenceIndex;
  
  let stepIndex = 0;
  let currentDelay = 45; // super fast at start

  statusMessage.value = '⚡ 运转能量汇聚中... 时空奖励正在欢快跃过！';

  function runTick() {
    const currentSequencePos = stepIndex % hopSequence.length;
    activeIndex.value = hopSequence[currentSequencePos];
    selectedCouponId.value = coupons[activeIndex.value].id;

    playTickSound(stepIndex);

    stepIndex++;

    if (stepIndex < totalSteps) {
      // Calculate realistic ease slowdown curves for tense physical deceleration
      const stepsRemaining = totalSteps - stepIndex;
      if (stepsRemaining < 4) {
        currentDelay += 160; // huge dramatic slowdown near target!
      } else if (stepsRemaining < 8) {
        currentDelay += 70;
      } else if (stepsRemaining < 12) {
        currentDelay += 35;
      } else if (stepsRemaining < 18) {
        currentDelay += 15;
      } else {
        // Initially accelerate
        currentDelay = Math.max(38, currentDelay - 3);
      }

      setTimeout(runTick, currentDelay);
    } else {
      // Landed!
      isDrawing.value = false;
      playSuccessChord();

      const winner = coupons[targetWinnerId];
      statusMessage.value = `🎉 恭喜中签！获得特权：【${winner.name}】`;

      // Slow buffer before high-impact certificate popover appears
      setTimeout(() => {
        const randNum = Math.floor(1000 + Math.random() * 9000);
        const hashSeed = Math.random().toString(36).substring(2, 7).toUpperCase();
        const currentTimestamp = new Date().toISOString().replace('T', ' ').substring(0, 19);

        redeemedReceipt.value = {
          name: winner.name,
          price: winner.price,
          key: winner.code,
          validationCode: `LOVE-VOUCHER-${randNum}-2026`,
          timestamp: currentTimestamp + ' (生日快乐)',
          signature: `DEAREST-WIFE-SIG-${hashSeed}`
        };
      }, 750);
    }
  }

  runTick();
}
</script>

<style scoped>
/* Glowing Hop Scanline */
@keyframes scannerSweep {
  0% {
    top: 0%;
    opacity: 0.1;
  }
  50% {
    opacity: 1;
  }
  100% {
    top: 100%;
    opacity: 0.1;
  }
}

/* physical bounce scaling tick */
@keyframes bounceTick {
  0% {
    transform: scale(1.03);
    box-shadow: 0 0 15px rgba(255, 94, 140, 0.3);
  }
  100% {
    transform: scale(1.06);
    box-shadow: 0 0 28px rgba(255, 94, 140, 0.6);
  }
}
</style>

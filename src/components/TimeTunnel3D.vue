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
import { ref, onMounted, onUnmounted, computed, watch } from 'vue';
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
    year: '2023.04',
    phase: '初识星空',
    title: '时光初遇 · 灵魂引力交错',
    photo: 'https://images.unsplash.com/photo-1506318137071-a8e063b4bec0?auto=format&fit=crop&w=400&q=80',
    content: '第一次见面认识，当时前面已经聊了好几个月了，已经很了解了，一起骑绿道，当时好像一起骑了20公里，后面好像去了一个公园，被你闺蜜偷拍了我两的第一张照片，虽然只是一个背影',
    depth: 350 // Z goes from 350 to 290
  },
  {
    year: '2023.06',
    phase: '浪漫执手',
    title: '温馨执手 · 彼此轨道同步',
    photo: 'https://images.unsplash.com/photo-1518199266791-5375a83190b7?auto=format&fit=crop&w=400&q=80',
    content: '你刚毕业，毕业快乐',
    depth: 290 // Z goes from 290 to 230
  },
  {
    year: '2023.07',
    phase: '携手旅途',
    title: '逃离地心 · 奔赴蔚蓝星海',
    photo: 'https://images.unsplash.com/photo-1529156069898-49953e39b3ac?auto=format&fit=crop&w=400&q=80',
    content: '一起租了一个房子《一品天成》，周末一起去看油菜花，没想到你会主动和我一起拍照，这是我们两个的第一张合照（准确的说是第二张，但是第一张没正脸）',
    depth: 230 // Z goes from 230 to 170
  },
  {
    year: '2023.08',
    phase: '携手旅途',
    title: '逃离地心 · 奔赴蔚蓝星海',
    photo: 'https://images.unsplash.com/photo-1529156069898-49953e39b3ac?auto=format&fit=crop&w=400&q=80',
    content: '一起去了成都自然博物馆，但是那一次好像没拍合照',
    depth: 230 // Z goes from 230 to 170
  },
  {
    year: '2023.08',
    phase: '携手旅途',
    title: '逃离地心 · 奔赴蔚蓝星海',
    photo: 'https://images.unsplash.com/photo-1529156069898-49953e39b3ac?auto=format&fit=crop&w=400&q=80',
    content: '追了大半年终于在一起了，这天是七夕，但是某人说我没准备好，也没提前订饭店，但是我确实也没经验，也没想到当时的万达广场没什么玩的，但是某人就很生气，让你走也不走，说实话当时我也很委屈，但是后面还是找了一个西餐厅，某人吃得很高兴，晚上我在楼下广场深情终于把某人拿下了，正式确定男女朋友关系。',
    depth: 230 // Z goes from 230 to 170
  },
  {
    year: '2023.09',
    phase: '携手旅途',
    title: '逃离地心 · 奔赴蔚蓝星海',
    photo: 'https://images.unsplash.com/photo-1529156069898-49953e39b3ac?auto=format&fit=crop&w=400&q=80',
    content: '出门的日常，带你出去玩，吃好吃的，',
    depth: 230 // Z goes from 230 to 170
  },
  {
    year: '2023.10',
    phase: '携手旅途',
    title: '逃离地心 · 奔赴蔚蓝星海',
    photo: 'https://images.unsplash.com/photo-1529156069898-49953e39b3ac?auto=format&fit=crop&w=400&q=80',
    content: '十月一国庆节，你爸说要见我，你带我回重庆去了，当时第一次去很紧张，但是嘴还是很硬的，后面我们好像去一个公园玩耍去了',
    depth: 230 // Z goes from 230 to 170
  },
  {
    year: '2023.10',
    phase: '携手旅途',
    title: '逃离地心 · 奔赴蔚蓝星海',
    photo: 'https://images.unsplash.com/photo-1529156069898-49953e39b3ac?auto=format&fit=crop&w=400&q=80',
    content: '已经忘记了这是回重庆还是离开重庆回成都了，也不知道是什么时候拍的这张照片',
    depth: 230 // Z goes from 230 to 170
  },
  {
    year: '2023.10',
    phase: '携手旅途',
    title: '逃离地心 · 奔赴蔚蓝星海',
    photo: 'https://images.unsplash.com/photo-1529156069898-49953e39b3ac?auto=format&fit=crop&w=400&q=80',
    content: '非要带我来试试西装，没想到我随便一穿就把某人迷到了，倒是要1000多块，最后还是没买。',
    depth: 230 // Z goes from 230 to 170
  },
  {
    year: '2023.10',
    phase: '携手旅途',
    title: '逃离地心 · 奔赴蔚蓝星海',
    photo: 'https://images.unsplash.com/photo-1529156069898-49953e39b3ac?auto=format&fit=crop&w=400&q=80',
    content: '一起逛宽窄巷子，但是某人一路免费试吃，特别是在银耳汤那个店，每一个品类都要品尝一下，喝饱了就跑',
    depth: 230 // Z goes from 230 to 170
  },
  {
    year: '2023.10',
    phase: '携手旅途',
    title: '逃离地心 · 奔赴蔚蓝星海',
    photo: 'https://images.unsplash.com/photo-1529156069898-49953e39b3ac?auto=format&fit=crop&w=400&q=80',
    content: '第一次一起出来玩碰碰车，我也是第一次玩，但是某人特别喜欢玩这个，里面还有很多其他的玩的，还在里面打上保龄球，也是过上了高端人士的生活了，',
    depth: 230 // Z goes from 230 to 170
  },
  {
    year: '2023.11',
    phase: '携手旅途',
    title: '逃离地心 · 奔赴蔚蓝星海',
    photo: 'https://images.unsplash.com/photo-1529156069898-49953e39b3ac?auto=format&fit=crop&w=400&q=80',
    content: '这好像是在吃烤肉，但是为什么某人这么高兴，我已经忘了，但是你开心就好',
    depth: 230 // Z goes from 230 to 170
  },
  {
    year: '2023.11',
    phase: '携手旅途',
    title: '逃离地心 · 奔赴蔚蓝星海',
    photo: 'https://images.unsplash.com/photo-1529156069898-49953e39b3ac?auto=format&fit=crop&w=400&q=80',
    content: '给我买的情侣睡衣，长这么大第一次穿情侣睡衣，真的很感动，虽然睡衣有点大了，但是也不影响',
    depth: 230 // Z goes from 230 to 170
  },
  {
    year: '2023.11',
    phase: '携手旅途',
    title: '逃离地心 · 奔赴蔚蓝星海',
    photo: 'https://images.unsplash.com/photo-1529156069898-49953e39b3ac?auto=format&fit=crop&w=400&q=80',
    content: '这是你骑车我怕你冷，专门给你买的，还有一副手套，不知道现在还在不，上一次手套掉了一只，被我说了，还好被我找到了',
    depth: 230 // Z goes from 230 to 170
  },
  {
    year: '2023.11',
    phase: '携手旅途',
    title: '逃离地心 · 奔赴蔚蓝星海',
    photo: 'https://images.unsplash.com/photo-1529156069898-49953e39b3ac?auto=format&fit=crop&w=400&q=80',
    content: '这是我给你买的情侣鞋子，这可是实拍图，真的还蛮好看的，就是质量不太行，以后有机会我再买一双，买个质量好点的，主要是好看的太难找了',
    depth: 230 // Z goes from 230 to 170
  },
  {
    year: '2023.12',
    phase: '携手旅途',
    title: '逃离地心 · 奔赴蔚蓝星海',
    photo: 'https://images.unsplash.com/photo-1529156069898-49953e39b3ac?auto=format&fit=crop&w=400&q=80',
    content: '大早上送你去学校，这张照片是早上6点21拍的，天还没亮就起来了，真的挺心疼的要起来这么早，',
    depth: 230 // Z goes from 230 to 170
  },
  {
    year: '2023.12',
    phase: '携手旅途',
    title: '逃离地心 · 奔赴蔚蓝星海',
    photo: 'https://images.unsplash.com/photo-1529156069898-49953e39b3ac?auto=format&fit=crop&w=400&q=80',
    content: '这张照片挺好看的，好看的衣服果然都是我买的',
    depth: 230 // Z goes from 230 to 170
  },
  {
    year: '2024.1',
    phase: '温馨筑巢',
    title: '橘光筑巢 · 烟火琐碎也甜',
    photo: 'https://images.unsplash.com/photo-1513694203232-719a280e022f?auto=format&fit=crop&w=400&q=80',
    content: '一起去人民公园，在里面划船，我其实有点怕，我可不会水呀，但是后面就没啥事，毕竟在某人面前我要装一点',
    depth: 170 // Z goes from 170 to 110
  },
  {
    year: '2024.2',
    phase: '温馨筑巢',
    title: '橘光筑巢 · 烟火琐碎也甜',
    photo: 'https://images.unsplash.com/photo-1513694203232-719a280e022f?auto=format&fit=crop&w=400&q=80',
    content: '第一次带某人回老家过年，看着车一直在上山，在车上怕被我卖了，但是在我家呆着也挺舒服了，还吃了烧烤，太松弛了，太阳也很好，nice',
    depth: 170 // Z goes from 170 to 110
  },
  {
    year: '2024.2',
    phase: '温馨筑巢',
    title: '橘光筑巢 · 烟火琐碎也甜',
    photo: 'https://images.unsplash.com/photo-1513694203232-719a280e022f?auto=format&fit=crop&w=400&q=80',
    content: '后面又去了重庆，一起去了重庆海洋馆好像，还拍了我们第一张实体照片，现在还在呢',
    depth: 170 // Z goes from 170 to 110
  },
  {
    year: '2024.3',
    phase: '温馨筑巢',
    title: '橘光筑巢 · 烟火琐碎也甜',
    photo: 'https://images.unsplash.com/photo-1513694203232-719a280e022f?auto=format&fit=crop&w=400&q=80',
    content: '好像又去那儿玩了，但是这次好像某人因为不满意我的拍照技术特别生气，当天都没怎么说话，一直到最好才拍了这张照片，但是对于我来说每一张照片都是有意义的，只要是我们在一起就好了',
    depth: 170 // Z goes from 170 to 110
  },
  {
    year: '2024.3',
    phase: '温馨筑巢',
    title: '橘光筑巢 · 烟火琐碎也甜',
    photo: 'https://images.unsplash.com/photo-1513694203232-719a280e022f?auto=format&fit=crop&w=400&q=80',
    content: '已经忘记了是什么原因你没回来，好像在外面露营，睡的帐篷，我跟你在视频，我还在加班，后面你睡了，我就挂了',
    depth: 170 // Z goes from 170 to 110
  },
  {
    year: '2024.3',
    phase: '温馨筑巢',
    title: '橘光筑巢 · 烟火琐碎也甜',
    photo: 'https://images.unsplash.com/photo-1513694203232-719a280e022f?auto=format&fit=crop&w=400&q=80',
    content: '我又忘了这是什么节日还是什么了，带你去了双子塔，拍了些照片，离我上班还挺近的',
    depth: 170 // Z goes from 170 to 110
  },
  {
    year: '2024.8',
    phase: '温馨筑巢',
    title: '橘光筑巢 · 烟火琐碎也甜',
    photo: 'https://images.unsplash.com/photo-1513694203232-719a280e022f?auto=format&fit=crop&w=400&q=80',
    content: '这都到8月份了，但是我们怎么没有什么照片，这是你带我去温泉酒店游泳拍的，其实还挺好玩的，就是泡温泉男女要分开不好，我还是挺帅的',
    depth: 170 // Z goes from 170 to 110
  },
  {
    year: '2024.8',
    phase: '温馨筑巢',
    title: '橘光筑巢 · 烟火琐碎也甜',
    photo: 'https://images.unsplash.com/photo-1513694203232-719a280e022f?auto=format&fit=crop&w=400&q=80',
    content: '这都到8月份了，但是我们怎么没有什么照片，这时候已经搬家到《燕南国际》了，这是你带我去温泉酒店游泳拍的，其实还挺好玩的，就是泡温泉男女要分开不好',
    depth: 170 // Z goes from 170 to 110
  },
  {
    year: '2024.10',
    phase: '温馨筑巢',
    title: '橘光筑巢 · 烟火琐碎也甜',
    photo: 'https://images.unsplash.com/photo-1513694203232-719a280e022f?auto=format&fit=crop&w=400&q=80',
    content: '又到国庆了，这次我们又去重庆玩去了，我社牛的喊人给我们拍的照片，这是朝天门，人真的好多呀，',
    depth: 170 // Z goes from 170 to 110
  },
  {
    year: '2024.11',
    phase: '温馨筑巢',
    title: '橘光筑巢 · 烟火琐碎也甜',
    photo: 'https://images.unsplash.com/photo-1513694203232-719a280e022f?auto=format&fit=crop&w=400&q=80',
    content: '第一次来新房子，有自己的房子了，还天天忙着看装修风格和家具，两个人什么都不懂，还好我感觉我们运气还挺好的，走到那儿那儿打折促销，买的东西性价比都挺高的，也挺好看的，装修了大半年，到现在床头柜和电视柜都还没有',
    depth: 170 // Z goes from 170 to 110
  },
  {
    year: '2025.2',
    phase: '温馨筑巢',
    title: '橘光筑巢 · 烟火琐碎也甜',
    photo: 'https://images.unsplash.com/photo-1513694203232-719a280e022f?auto=format&fit=crop&w=400&q=80',
    content: '计划结婚了，订婚宴上给你买的花花，真的太好看了，又有排面，我从重庆一直拿到成都来了，可惜后面用来泡脚了，可惜订婚宴上也没给你买上好看的衣服，但是结婚大局已定，正式成为我的未婚妻了',
    depth: 170 // Z goes from 170 to 110
  },
  {
    year: '2025.03',
    phase: '誓言誓约',
    title: '宿命加冕 · 交换未来密钥',
    photo: 'https://images.unsplash.com/photo-1469371670807-013ccf25f16a?auto=format&fit=crop&w=400&q=80',
    content: '一起去了东郊记忆，但是好像没什么玩的，玩什么不重要，开心就好',
    depth: 110 // Z goes from 110 to 50
  },
  {
    year: '2025.04',
    phase: '誓言誓约',
    title: '宿命加冕 · 交换未来密钥',
    photo: 'https://images.unsplash.com/photo-1469371670807-013ccf25f16a?auto=format&fit=crop&w=400&q=80',
    content: '这张艺术照很有感觉，真不错',
    depth: 110 // Z goes from 110 to 50
  },
  {
    year: '2025.05',
    phase: '誓言誓约',
    title: '宿命加冕 · 交换未来密钥',
    photo: 'https://images.unsplash.com/photo-1469371670807-013ccf25f16a?auto=format&fit=crop&w=400&q=80',
    content: '拍婚纱照了，大老远跑去青白江拍婚纱照了，累坏了，拍的也还行，也不算贵，只是后期处理没处理好，好在把照片源文件全都拿回来了，以后可以自己来修图',
    depth: 110 // Z goes from 110 to 50
  },
  {
    year: '2025.05',
    phase: '誓言誓约',
    title: '宿命加冕 · 交换未来密钥',
    photo: 'https://images.unsplash.com/photo-1469371670807-013ccf25f16a?auto=format&fit=crop&w=400&q=80',
    content: '这是在那儿玩，我已经忘记了，但是开心就好，不对我想起来了，好像是欢乐谷，那过山车真的吓坏了，某人眼睛都不敢睁开，再也不去了，太吓人了',
    depth: 110 // Z goes from 110 to 50
  },
  {
    year: '2025.05',
    phase: '誓言誓约',
    title: '宿命加冕 · 交换未来密钥',
    photo: 'https://images.unsplash.com/photo-1469371670807-013ccf25f16a?auto=format&fit=crop&w=400&q=80',
    content: '恭喜老板，喜提一辆玛莎拉蒂，太有牌面了，哈哈（以后有钱高低跟你整一辆）',
    depth: 110 // Z goes from 110 to 50
  },
  {
    year: '2025.05',
    phase: '誓言誓约',
    title: '宿命加冕 · 交换未来密钥',
    photo: 'https://images.unsplash.com/photo-1469371670807-013ccf25f16a?auto=format&fit=crop&w=400&q=80',
    content: '520领结婚证啦，正式成为有证夫妻，这下可跑不掉了，哈哈哈哈，我那天怎么忘记挂胡子了，我专门准备的头纱，到现场有现成的，不过后面用完就退了',
    depth: 110 // Z goes from 110 to 50
  },
  {
    year: '2025.05',
    phase: '誓言誓约',
    title: '宿命加冕 · 交换未来密钥',
    photo: 'https://images.unsplash.com/photo-1469371670807-013ccf25f16a?auto=format&fit=crop&w=400&q=80',
    content: '海底捞凭结婚证打折了，开心，带你吃顿好的，还有仪仗队，仪式感拉满了',
    depth: 110 // Z goes from 110 to 50
  },
  {
    year: '2025.08',
    phase: '誓言誓约',
    title: '宿命加冕 · 交换未来密钥',
    photo: 'https://images.unsplash.com/photo-1469371670807-013ccf25f16a?auto=format&fit=crop&w=400&q=80',
    content: '筹备婚礼用品了，宝宝辛苦了，全程自己筹备，，给宝宝点个赞，这套睡衣太喜庆了，马上结婚啦',
    depth: 110 // Z goes from 110 to 50
  },
  {
    year: '2025.09',
    phase: '誓言誓约',
    title: '宿命加冕 · 交换未来密钥',
    photo: 'https://images.unsplash.com/photo-1469371670807-013ccf25f16a?auto=format&fit=crop&w=400&q=80',
    content: '这张照片也特别好看，郎才女貌，太好看了',
    depth: 110 // Z goes from 110 to 50
  },
  {
    year: '2025.10',
    phase: '誓言誓约',
    title: '宿命加冕 · 交换未来密钥',
    photo: 'https://images.unsplash.com/photo-1469371670807-013ccf25f16a?auto=format&fit=crop&w=400&q=80',
    content: '结婚啦，抢亲了，太激动了，跑楼道摔了我一跟头，赶紧让摄影师砍掉，本来想临场发挥让你感动的，结果拉一坨大的，没想到你还准备得挺好的，说得挺不错的',
    depth: 110 // Z goes from 110 to 50
  },
  {
    year: '2025.12',
    phase: '誓言誓约',
    title: '宿命加冕 · 交换未来密钥',
    photo: 'https://images.unsplash.com/photo-1469371670807-013ccf25f16a?auto=format&fit=crop&w=400&q=80',
    content: '偷一张你的大图，就当总结了',
    depth: 110 // Z goes from 110 to 50
  },
  {
    year: '2026.01',
    phase: '誓言誓约',
    title: '宿命加冕 · 交换未来密钥',
    photo: 'https://images.unsplash.com/photo-1469371670807-013ccf25f16a?auto=format&fit=crop&w=400&q=80',
    content: '加油小孙，马上有财，马上有钱',
    depth: 110 // Z goes from 110 to 50
  },
  {
    year: '2026.01',
    phase: '誓言誓约',
    title: '宿命加冕 · 交换未来密钥',
    photo: 'https://images.unsplash.com/photo-1469371670807-013ccf25f16a?auto=format&fit=crop&w=400&q=80',
    content: '大家一起跨年，我很喜欢这种感觉，上班太累了，大家聚在一起开开心心，真是难得的时光',
    depth: 110 // Z goes from 110 to 50
  },
  {
    year: '2026.02',
    phase: '誓言誓约',
    title: '宿命加冕 · 交换未来密钥',
    photo: 'https://images.unsplash.com/photo-1469371670807-013ccf25f16a?auto=format&fit=crop&w=400&q=80',
    content: '又被我套路了，你还是太年轻了，哈哈哈哈',
    depth: 110 // Z goes from 110 to 50
  },
  {
    year: '2026.02',
    phase: '誓言誓约',
    title: '宿命加冕 · 交换未来密钥',
    photo: 'https://images.unsplash.com/photo-1469371670807-013ccf25f16a?auto=format&fit=crop&w=400&q=80',
    content: '又回老家啦，跟送女儿上学一样，开开心心，今年过年都没好好休息，时间过得太快了，假期又短',
    depth: 110 // Z goes from 110 to 50
  },
  {
    year: '2026.03',
    phase: '誓言誓约',
    title: '宿命加冕 · 交换未来密钥',
    photo: 'https://images.unsplash.com/photo-1469371670807-013ccf25f16a?auto=format&fit=crop&w=400&q=80',
    content: '今年就是日常生活啦，照片也变少了，两个人都发福了，生活慢慢回归于平淡的生活，但是我对你的热爱永远不变，爱你哟，老婆',
    depth: 110 // Z goes from 110 to 50
  },
  {
    year: '2026.06',
    phase: '璀璨今朝',
    title: '璀璨周年 · 热爱永无边界',
    photo: 'https://images.unsplash.com/photo-1531747118685-ca8fa6e08806?auto=format&fit=crop&w=400&q=80',
    content: '老婆，祝你生日快乐！要天天开心哟，永远爱你的老公！',
    depth: 50 // Z < 50
  }
];

// Interactive depth states
// Z ranges from 400 (start) down to 0 (end)
const cameraZOffset = ref(400); 
const depthPercent = computed(() => {
  return Math.min(Math.max(((400 - cameraZOffset.value) / 380) * 100, 0), 100);
});

// 计算当前视窗（CameraZOffset）最贴近的卡片索引，用以作为卡片翻页边界触发判定
const activeCardIndex = computed(() => {
  let minDistance = Infinity;
  let activeIdx = 0;
  timelineCards.forEach((card, idx) => {
    const dist = Math.abs(cameraZOffset.value - card.depth);
    if (dist < minDistance) {
      minDistance = dist;
      activeIdx = idx;
    }
  });
  return activeIdx;
});

// 音频上下文声明，用以支持网页音频程序化物理合成
let audioCtx: AudioContext | null = null;

// 监听当前高亮卡片索引变动，触发逼真的翻书摩擦沙沙声
watch(activeCardIndex, (newIdx, oldIdx) => {
  if (oldIdx !== undefined) {
    playPageTurnSound();
  }
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

  if (audioCtx) {
    audioCtx.close();
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

/**
 * 实时物理声学合成：模拟类似翻动书本、纸张沙沙摩擦的音效。
 * 运用 Web Audio API 动态构建噪声源并施加包络阻尼和频率扫频滤波，彻底免除外部资源 CORS 限制。
 * 遵循谷歌前端编码规范。
 */
function playPageTurnSound(): void {
  try {
    // 懒加载初始化音频上下文，绕过移动端浏览器自动播放限制
    if (!audioCtx) {
      audioCtx = new (window.AudioContext || (window as any).webkitAudioContext)();
    }
    if (audioCtx.state === 'suspended') {
      audioCtx.resume();
    }

    const now = audioCtx.currentTime;
    
    // 1. 产生 0.35 秒长的随机白噪波缓存区数据，用以重现纸张相互摩擦产生的粗糙噪音
    const bufferSize = audioCtx.sampleRate * 0.35;
    const buffer = audioCtx.createBuffer(1, bufferSize, audioCtx.sampleRate);
    const channelData = buffer.getChannelData(0);
    for (let i = 0; i < bufferSize; i++) {
      channelData[i] = Math.random() * 2 - 1;
    }
    
    const noiseSource = audioCtx.createBufferSource();
    noiseSource.buffer = buffer;
    
    // 2. 建立二阶带通滤波器 (Bandpass Filter) 限制频率范围，滤除刺耳的高低频
    const bandpassFilter = audioCtx.createBiquadFilter();
    bandpassFilter.type = 'bandpass';
    bandpassFilter.Q.setValueAtTime(4.0, now);
    
    // 扫频包络：起始为较清脆的 1500Hz 摩擦声，随后下降至 350Hz 纸张扑动低音，拟真物理运动
    bandpassFilter.frequency.setValueAtTime(1500, now);
    bandpassFilter.frequency.exponentialRampToValueAtTime(350, now + 0.32);
    
    // 3. 音量包络设计：快速淡入（5ms），配合平滑的指数级衰退（350ms）
    // 关键优化：将峰值音量从 0.06 提高到 0.22，使翻书音效更加清晰响亮
    const gainNode = audioCtx.createGain();
    gainNode.gain.setValueAtTime(0, now);
    gainNode.gain.linearRampToValueAtTime(0.5, now + 0.05);
    gainNode.gain.exponentialRampToValueAtTime(0.0001, now + 0.35);
    
    // 4. 链路路由及播放
    noiseSource.connect(bandpassFilter);
    bandpassFilter.connect(gainNode);
    gainNode.connect(audioCtx.destination);
    
    noiseSource.start(now);
    // 播放周期结束后彻底关闭该音源缓冲
    noiseSource.stop(now + 0.35);
  } catch (err) {
    // 降级防御，静默处理由于特殊移动终端音频被强行挂起导致的报错
  }
}
</script>

<style scoped>
/* Floating/Perspective transformations styling */
.translate-y-0 {
  transform: translateY(0);
}
</style>

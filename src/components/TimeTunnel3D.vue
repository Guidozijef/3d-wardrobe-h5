<template>
  <div class="relative w-full h-full overflow-hidden bg-[#140411] flex flex-col items-center justify-between p-4 hologram-grid touch-none" style="touch-action: none; overscroll-behavior: contain" @touchstart="onTouchStart" @touchmove="onTouchMove" @touchend="onTouchEnd" @touchcancel="onTouchEnd" @mousedown="onMouseDown" @mousemove="onMouseMove" @mouseup="onMouseUp" @mouseleave="onMouseUp" @wheel.prevent="onWheel">
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
        <button @click="emit('next')" class="pointer-events-auto px-3 py-1.5 text-[11px] rounded border border-[#ff5e8c]/40 text-[#ff5e8c] hover:bg-[#ff5e8c]/20 cursor-pointer transition-all uppercase tracking-widest" id="skip_tunnel_btn">跳过</button>
      </div>

      <!-- Center Space: Milestone Cards Display (Rotated / Floating based on scroll depth index) -->
      <div class="relative w-full flex-grow flex items-center justify-center my-4 overflow-visible">
        <div v-for="(card, index) in timelineCards" :key="card.year" class="absolute w-full max-w-[285px] sm:max-w-[310px] p-3 sm:p-4 rounded-2xl bg-[#140411]/95 border border-[#ff5e8c]/30 shadow-[0_0_35px_rgba(255,94,140,0.18)] transition-all duration-700 ease-out flex flex-col items-center pointer-events-auto select-none" :class="[getCardClass(index)]">
          <!-- Elegant Badge -->
          <span class="px-2.5 py-0.5 rounded-full text-[9px] font-sans font-bold tracking-widest bg-[#ff5e8c]/15 text-[#ff5e8c] border border-[#ff5e8c]/30 mb-2 sm:mb-3">💌 {{ card.year }} // {{ card.phase }}</span>

          <!-- Photo Window -->
          <div @click="openLightbox(card.photo)" class="relative w-full rounded-lg overflow-hidden border border-white/10 mb-2 sm:mb-3 bg-neutral-950 group cursor-pointer flex items-center justify-center min-h-[150px] max-h-[280px] sm:max-h-[340px]">
            <!-- 悬浮态磨砂提示框，引导用户点击查看长图/大图，增强交互体验 -->
            <div class="absolute inset-0 bg-black/40 opacity-0 group-hover:opacity-100 transition-opacity duration-300 flex items-center justify-center z-20 backdrop-blur-[2px]">
              <div class="px-3 py-1.5 rounded-lg bg-black/60 border border-white/20 flex items-center gap-1.5 shadow-lg scale-90 group-hover:scale-100 transition-transform duration-300">
                <svg class="w-3.5 h-3.5 text-[#ff5e8c]" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0zM10 7v6m4-3H6" />
                </svg>
                <span class="text-[10px] text-white font-sans tracking-wider font-semibold">点击查看完整长图</span>
              </div>
            </div>

            <div class="absolute inset-0 bg-gradient-to-t from-black/50 via-transparent to-transparent pointer-events-none z-10"></div>

            <img :src="getLocalImage(card.photo)" alt="Memory Node Visual" class="max-w-full max-h-[280px] sm:max-h-[340px] object-contain transition-transform duration-1000 group-hover:scale-103 filter saturate-[1.1] block" />

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
        <div v-if="depthPercent < 15" class="absolute bottom-8 flex flex-col items-center justify-center animate-bounce pointer-events-none">
          <svg class="w-7 h-7 text-[#ff5e8c] animate-pulse" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M19 14l-7 7m0 0l-7-7m7 7V3" />
          </svg>
          <span class="font-sans text-[10px] tracking-wider text-[#ff5e8c] uppercase mt-1 bg-black/60 px-3 py-1 rounded-full">向上轻滑屏幕，追溯甜蜜往事 ❤️</span>
        </div>
      </div>

      <!-- Bottom Interactive Controls & progress HUD -->
      <div class="w-full flex flex-col items-center mb-8 px-4 pointer-events-auto">
        <!-- Progress Bar representing camera position depth -->
        <div class="w-full bg-white/5 border border-white/10 rounded-full h-2.5 p-0.5 relative mb-4">
          <div class="h-full bg-gradient-to-r from-[#ff5e8c] via-[#c84095] to-[#ffd27a] rounded-full transition-all duration-100 shadow-[0_0_8px_rgba(255,140,180,0.6)]" :style="{ width: `${depthPercent}%` }"></div>
          <div class="absolute -top-5 right-0 font-sans text-[9px] text-gray-300">❤ 相伴的每一步，都是风景</div>
        </div>

        <!-- Warp Depth / Actions -->
        <div class="w-full flex justify-between items-center font-sans text-[10px] text-gray-400 mb-2">
          <span>相伴进度 // {{ Math.round(depthPercent) }}%</span>
          <span>时光穿梭 // {{ Math.round(startZ - cameraZOffset) }} 光年</span>
        </div>

        <!-- Full Transition button once reached end of tunnel -->
        <button v-if="depthPercent >= 98" @click="emit('next')" class="w-full py-3.5 rounded-xl border border-[#ff5e8c] bg-[#ff5e8c]/10 hover:bg-[#ff5e8c]/20 text-[#ff5e8c] font-bold tracking-widest text-xs uppercase animate-pulse cursor-pointer transform hover:scale-102 transition-all shadow-[0_0_20px_rgba(255,94,140,0.4)]" id="tunnel_complete_btn">探索终点：开启指尖祈愿 💫</button>
      </div>
    </div>

    <!-- 全屏灯箱模态框：支持滚动条垂直滑动，确保纵向长图能完整浏览 -->
    <div v-if="isLightboxOpen" class="fixed inset-0 z-[100] bg-black/90 backdrop-blur-md flex flex-col items-center justify-center p-4 cursor-zoom-out pointer-events-auto" @click="closeLightbox" @wheel.stop.prevent>
      <!-- 右上角悬浮关闭按钮 -->
      <button @click.stop="closeLightbox" class="absolute top-6 right-6 text-white/70 hover:text-white p-2.5 rounded-full bg-white/10 border border-white/20 hover:bg-white/20 hover:scale-110 active:scale-95 transition-all cursor-pointer z-50 shadow-lg" id="close_lightbox_btn" aria-label="关闭预览">
        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
        </svg>
      </button>

      <!-- 滚动图片容器：提供 overflow-y-auto 属性以支持长图滚动 -->
      <div class="w-full max-w-xl max-h-[85vh] overflow-y-auto rounded-xl flex justify-center custom-scrollbar border border-white/10 bg-neutral-950/40 shadow-2xl p-1 touch-pan-y" style="touch-action: pan-y" @click.stop>
        <img :src="getLocalImage(lightboxPhoto)" alt="Full Memory Node Visual Preview" class="w-auto h-auto max-w-full rounded-lg select-none shadow-2xl" />
      </div>

      <!-- 底部操作引导提示 -->
      <span class="text-[10px] text-gray-400 mt-6 tracking-widest uppercase animate-pulse select-none bg-black/40 px-3.5 py-1.5 rounded-full border border-white/5">点击任意空白处或右上角关闭预览 // CLICK OUTSIDE TO CLOSE</span>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, computed, watch } from "vue";
import * as THREE from "three";
import { gsap } from "gsap";

const emit = defineEmits<{
  (e: "next"): void;
}>();

// Refs
const canvasContainer = ref<HTMLDivElement | null>(null);

/**
 * 获取本地照片的动态 URL 路径。
 * 符合 Google 编码规范，使用 Vite 推荐的 new URL(path, import.meta.url) 模式动态解析本地资源。
 *
 * @param {string} fileName 图片文件名，如 '1.jpg'。
 * @return {string} 解析后的完整 URL 字符串。
 */
const getLocalImage = (fileName: string): string => {
  return new URL(`../imgs/${fileName}`, import.meta.url).href;
};

// 星空时光隧道及浪漫里程碑卡片数据配置
const timelineCards = [
  {
    year: "2023.04",
    phase: "时光初遇",
    title: "时光初遇 · 绿道骑行与林间背影",
    photo: "1.jpg",
    content: "这是我们的第一次见面，但是前面已经聊了好几个月了，已经很了解了，一起骑绿道，当时好像一起骑了20公里，后面好像去了一个公园，被你闺蜜偷拍了我两的第一张照片，虽然只是一个背影",
    depth: 350,
  },
  {
    year: "2023.06",
    phase: "青春礼赞",
    title: "青春致敬 · 祝我的女孩毕业快乐",
    photo: "2.jpg",
    content: "你刚毕业，毕业快乐",
    depth: 290,
  },
  {
    year: "2023.07",
    phase: "花田同框",
    title: "油菜花田 · 属于我们的第一张正面合影",
    photo: "3.jpg",
    content: "一起租了一个房子《一品天成》，周末一起去看油菜花，没想到你会主动和我一起拍照，这是我们两个的第一张合照（准确的说是第二张，但是第一张没正脸）",
    depth: 230,
  },
  {
    year: "2023.08",
    phase: "悄然定格",
    title: "自然博物馆 · 镜头里偷偷定格的你",
    photo: "4.jpg",
    content: "一起去了成都自然博物馆，但是那一次好像没拍合照，只有我偷拍的这一张，模模糊糊的",
    depth: 230,
  },
  {
    year: "2023.08",
    phase: "七夕告白",
    title: "七夕深情告白 · 正式确立恋爱关系",
    photo: "5.jpg",
    content: "追了大半年终于在一起了，这天是七夕，但是你说我没准备好，也没提前订饭店，但是我确实也没经验，也没想到当时的万达广场没什么玩的，但是你就很生气，让你走也不走，说实话当时我也很委屈，都以为今天要失败了，但是后面还是找了一个西餐厅，你吃得很高兴，晚上我在楼下广场深情告白，终于把你拿下了，正式确定男女朋友关系。",
    depth: 230,
  },
  {
    year: "2023.09",
    phase: "甜蜜日常",
    title: "甜蜜日常 · 搜寻世间的每一种美味",
    photo: "6.jpg",
    content: "出门的日常，带你出去玩，吃好吃的，",
    depth: 230,
  },
  {
    year: "2023.10",
    phase: "初登家门",
    title: "奔赴重庆 · 第一次见家长的紧张时光",
    photo: "7.jpg",
    content: "十月一国庆节，你爸说要见我，你带我回重庆去了，当时第一次去很紧张，但是嘴还是很硬的，后面我们好像去一个公园玩耍去了",
    depth: 230,
  },
  {
    year: "2023.10",
    phase: "浪漫旅途",
    title: "旅途剪影 · 双城往返的浪漫站台",
    photo: "8.jpg",
    content: "已经忘记了这是回重庆还是离开重庆回成都了，也不知道是什么时候拍的张照片",
    depth: 230,
  },
  {
    year: "2023.10",
    phase: "心动瞬间",
    title: "试穿西装 · 一瞬间让你心动的模样",
    photo: "9.jpg",
    content: "非要带我来试试西装，没想到我随便一穿就把你迷到了，只是要1000多块，最后还是没买。",
    depth: 230,
  },
  {
    year: "2023.10",
    phase: "可爱吃货",
    title: "宽窄巷子 · 免费试吃的可爱小吃货",
    photo: "10.jpg",
    content: "一起逛宽窄巷子，但是你一路免费试吃，特别是在银耳汤那个店，每一个品类都要品尝一下，喝饱了就跑",
    depth: 230,
  },
  {
    year: "2023.10",
    phase: "童趣飞扬",
    title: "速度与激情 · 畅玩碰碰车与保龄球",
    photo: "11.jpg",
    content: "第一次一起出来玩碰碰车，我也是第一次玩，但是你特别喜欢玩这个，里面还有很多其他的玩的，还在里面打上保龄球，也是过上了高端人士的生活了，",
    depth: 230,
  },
  {
    year: "2023.11",
    phase: "笑容璀璨",
    title: "烤肉聚餐 · 只要能让你开心就足够",
    photo: "12.jpg",
    content: "这好像是在吃烤肉，但是为什么你这么高兴，我已经忘了，但是你开心就好",
    depth: 230,
  },
  {
    year: "2023.11",
    phase: "温馨如家",
    title: "情侣睡衣 · 第一次拥有的贴心温暖",
    photo: "13.jpg",
    content: "给我买的情侣睡衣，长这么大第一次穿情侣睡衣，真的很感动，虽然睡衣有点大了，但是也不影响",
    depth: 230,
  },
  {
    year: "2023.11",
    phase: "温暖守护",
    title: "骑行守护 · 亲手为你戴上的避寒手套",
    photo: "14.jpg",
    content: "这是你骑车我怕你冷，专门给你买的，还有一副手套，不知道现在还在不，上一次手套掉了一只，被我说了，还好被我找到了",
    depth: 230,
  },
  {
    year: "2023.11",
    phase: "并肩同行",
    title: "情侣步履 · 记录我们并肩同行的足迹",
    photo: "15.jpg",
    content: "这是我给你买的情侣鞋子，这可是实拍图，真的还蛮好看的，就是质量不太行，以后有机会我再买一双，买个质量好点的，主要是好看的太难找了",
    depth: 230,
  },
  {
    year: "2023.12",
    phase: "晨曦微露",
    title: "晨光送行 · 披星戴月送你上学",
    photo: "16.jpg",
    content: "大早上送你去学校，这张照片是早上6点21拍的，天还没亮就起来了，真的挺心疼的要起来这么早，",
    depth: 230,
  },
  {
    year: "2023.12",
    phase: "独属美丽",
    title: "穿搭品味 · 镜头里最美的那个你",
    photo: "17.jpg",
    content: "这张照片挺好看的，好看的衣服果然都是我买的",
    depth: 230,
  },
  {
    year: "2024.1",
    phase: "碧波泛舟",
    title: "公园划船 · 在你面前努力装作勇敢",
    photo: "18.jpg",
    content: "一起去人民公园，在里面划船，我其实有点怕，我可不会水呀，但是后面就没啥事，毕竟在你面前我要装一点",
    depth: 170,
  },
  {
    year: "2024.2",
    phase: "温暖归乡",
    title: "老家过年 · 温暖山路与松弛日光",
    photo: "19.jpg",
    content: "第一次带你回老家过年，看着车一直在上山，在车上怕被我卖了，但是在我家呆着也挺舒服了，还吃了烧烤，太松弛了，太阳也很好，nice",
    depth: 170,
  },
  {
    year: "2024.2",
    phase: "蔚蓝印记",
    title: "海洋馆之旅 · 留存第一张珍贵实体照片",
    photo: "20.jpg",
    content: "后面又去了重庆，一起去了重庆海洋馆好像，还拍了我们第一张实体照片，现在还在呢",
    depth: 170,
  },
  {
    year: "2024.3",
    phase: "爱意释然",
    title: "拍照风波 · 执拗却无比珍贵的合影",
    photo: "21.jpg",
    content: "好像又去那儿玩了，但是这次好像你因为不满意我的拍照技术特别生气，当天都没怎么说话，一直到最后才拍了这张照片，但是对于我来说每一张照片都是有意义的，只要是我们在一起就好了",
    depth: 170,
  },
  {
    year: "2024.3",
    phase: "云端相守",
    title: "云端陪伴 · 加班与露营帐篷的视频通话",
    photo: "22.jpg",
    content: "已经忘记了是什么原因你没回来，好像在外面露营，睡的帐篷，我跟你在视频，我还在加班，后面你睡了，我就挂了",
    depth: 170,
  },
  {
    year: "2024.3",
    phase: "城市浪漫",
    title: "打卡双子塔 · 离工作最近的城市浪漫",
    photo: "23.jpg",
    content: "我又忘了这是什么节日还是什么了，带你去了双子塔，拍了些照片，离我上班还挺近的",
    depth: 170,
  },
  {
    year: "2024.8",
    phase: "温泉私语",
    title: "温泉度假 · 乔迁新居后的温馨水畔",
    photo: "24.jpg",
    content: "这都到8月份了，但是我们怎么没有什么照片，这时候已经搬家到《燕南国际》了，这是你带我去温泉酒店游泳拍的，其实还挺好玩的，就是泡温泉男女要分开不好",
    depth: 170,
  },
  {
    year: "2024.10",
    phase: "人潮相拥",
    title: "国庆朝天门 · 社牛路人帮忙的拥挤合影",
    photo: "25.jpg",
    content: "又到国庆了，这次我们又去重庆玩去了，我社牛的喊人给我们拍的照片，这是朝天门，人真的好多呀，",
    depth: 170,
  },
  {
    year: "2024.11",
    phase: "温馨筑巢",
    title: "筑巢引凤 · 有了自己的家与备婚琐碎",
    photo: "26.jpg",
    content: "第一次来新房子，有自己的房子了，还天天忙着看装修风格 and 家具，两个人什么都不懂，还好我感觉我们运气还挺好的，走到那儿那儿打折促销，买的东西性价比都挺高的，也挺好看的，装修了大半年，到现在床头柜 and 电视柜都还没有",
    depth: 170, // Z 轴坐标范围自 170 到 110
  },
  {
    year: "2025.2",
    phase: "温馨筑巢",
    title: "订婚盛宴 · 正式成为我的未婚妻",
    photo: "27.jpg",
    content: "计划结婚了，订婚宴上给你买的花花，真的太好看了，又有排面，我从重庆一直拿到成都来了，可惜后面用来泡脚了，可惜订婚宴上也没给你买上好看的衣服，但是结婚大局已定，正式成为我的未婚妻了",
    depth: 170, // Z 轴坐标范围自 170 到 110
  },
  {
    year: "2025.03",
    phase: "誓言誓约",
    title: "东郊记忆 · 只要有你陪伴去哪都开心",
    photo: "28.jpg",
    content: "一起去了东郊记忆，但是好像没什么玩的，玩什么不重要，开心就好",
    depth: 110, // Z 轴坐标范围自 110 到 50
  },
  {
    year: "2025.04",
    phase: "誓言誓约",
    title: "艺术写真 · 绝美定格的光影瞬间",
    photo: "29.jpg",
    content: "这张艺术照很有感觉，真不错",
    depth: 110, // Z 轴坐标范围自 110 到 50
  },
  {
    year: "2025.05",
    phase: "誓言誓约",
    title: "青白江婚纱照 · 累并快乐着的甜蜜拍摄",
    photo: "30.jpg",
    content: "拍婚纱照了，大老远跑去青白江拍婚纱照了，累坏了，拍的也还行，也不算贵，只是后期处理没处理好，好在把照片源文件全都拿回来了，以后可以自己来修图",
    depth: 110, // Z 轴坐标范围自 110 到 50
  },
  {
    year: "2025.05",
    phase: "誓言誓约",
    title: "欢乐谷狂欢 · 被过山车吓坏的可爱瞬间",
    photo: "31.jpg",
    content: "这是在那儿玩，我已经忘记了，但是开心就好，不对我想起来了，好像是欢乐谷，那过山车真的吓坏了，你眼睛都不敢睁开，再也不去了，太吓人了",
    depth: 110, // Z 轴坐标范围自 110 到 50
  },
  {
    year: "2025.05",
    phase: "誓言誓约",
    title: "许愿豪车 · 以后一定送你一辆玛莎拉蒂",
    photo: "32.jpg",
    content: "恭喜老板，喜提一辆玛莎拉蒂，太有牌面了，哈哈（以后有钱高低跟你整一辆）",
    depth: 110, // Z 轴坐标范围自 110 到 50
  },
  {
    year: "2025.05",
    phase: "誓言誓约",
    title: "520红色结婚证 · 正式结为合法夫妻",
    photo: "33.jpg",
    content: "520领结婚证啦，正式成为有证夫妻，这下可跑不掉了，哈哈哈哈，我那天怎么忘记挂胡子了，我专门准备的头纱，到现场有现成的，不过后面用完就退了",
    depth: 110, // Z 轴坐标范围自 110 到 50
  },
  {
    year: "2025.05",
    phase: "誓言誓约",
    title: "大红喜本 · 仪式感满满的海底捞欢庆",
    photo: "34.jpg",
    content: "海底捞凭结婚证打折了，开心，带你吃顿好的，还有仪仗队，仪式感拉满了",
    depth: 110, // Z 轴坐标范围自 110 到 50
  },
  {
    year: "2025.08",
    phase: "誓言誓约",
    title: "婚礼备置 · 喜庆睡衣与琐碎筹备",
    photo: "35.jpg",
    content: "筹备婚礼用品了，宝宝辛苦了，全程自己筹备，，给宝宝点个赞，这套睡衣太喜庆了，马上结婚啦",
    depth: 110, // Z 轴坐标范围自 110 到 50
  },
  {
    year: "2025.09",
    phase: "誓言誓约",
    title: "绝美同框 · 令人惊叹的郎才女貌",
    photo: "36.jpg",
    content: "这张照片也特别好看，郎才女貌，太好看了",
    depth: 110, // Z 轴坐标范围自 110 到 50
  },
  {
    year: "2025.10",
    phase: "誓言誓约",
    title: "世纪大婚 · 浪漫迎亲摔跤与真情誓言",
    photo: "37.jpg",
    content: "结婚啦，抢亲了，太激动了，跑楼道摔了我一跟头，赶紧让摄影师砍掉，本来想临场发挥让你感动的，结果拉一坨大的，没想到你还准备得挺好的，说得挺不错的",
    depth: 110, // Z 轴坐标范围自 110 到 50
  },
  {
    year: "2025.12",
    phase: "誓言誓约",
    title: "年度记忆总结 · 悄悄存下你的绝美单人照",
    photo: "38.jpg",
    content: "偷一张你的大图，就当总结了",
    depth: 110, // Z 轴坐标范围自 110 到 50
  },
  {
    year: "2026.01",
    phase: "誓言誓约",
    title: "新年期许 · 加油小孙，恭喜发财",
    photo: "39.jpg",
    content: "加油小孙，马上有财，马上有钱",
    depth: 110, // Z 轴坐标范围自 110 到 50
  },
  {
    year: "2026.01",
    phase: "誓言誓约",
    title: "快乐跨年 · 疲惫工作后的相聚温暖",
    photo: "40.jpg",
    content: "大家一起跨年，我很喜欢这种感觉，上班太累了，大家聚在一起开开心心，真是难得的时光",
    depth: 110, // Z 轴坐标范围自 110 到 50
  },
  {
    year: "2026.02",
    phase: "誓言誓约",
    title: "甜蜜博弈 · 再次被我的小套路拿下",
    photo: "41.jpg",
    content: "又被我套路了，你还是太年轻了，哈哈哈哈",
    depth: 110, // Z 轴坐标范围自 110 到 50
  },
  {
    year: "2026.02",
    phase: "誓言誓约",
    title: "年终返程 · 像送女儿上学一样的宠爱",
    photo: "42.jpg",
    content: "又回老家啦，跟送女儿上学一样，开开心心，今年过年都没好好休息，时间过得太快了，假期又短",
    depth: 110, // Z 轴坐标范围自 110 到 50
  },
  {
    year: "2026.03",
    phase: "誓言誓约",
    title: "细水长流 · 归于柴米油盐的永恒热忱",
    photo: "43.jpg",
    content: "今年就是日常生活啦，照片也变少了，两个人都发福了，生活慢慢回归于平淡的生活，但是我对你的热爱永远不变，爱你哟，老婆",
    depth: 110, // Z 轴坐标范围自 110 到 50
  },
  {
    year: "2026.06",
    phase: "璀璨今朝",
    title: "热烈庆生 · 给最亲爱老婆的生日祝福",
    photo: "44.jpg",
    content: "老婆，祝你生日快乐！要天天开心哟，永远爱你的老公！",
    depth: 50, // Z 轴坐标小于 50
  },
];

// 交互深度与三维空间参数配置
const startZ = 1800; // 穿梭起点深度坐标
const endZ = 20; // 穿梭终点深度坐标
const totalDepthRange = startZ - endZ; // 总穿梭距离

// 动态计算并平摊所有卡片的 Z 轴深度值，使其根据卡片数量完美均匀排布，防止重叠
timelineCards.forEach((card, idx) => {
  card.depth = startZ - idx * (totalDepthRange / (timelineCards.length - 1 || 1));
});

// Interactive depth states
// Z ranges from startZ down to endZ
const cameraZOffset = ref(startZ);
const depthPercent = computed(() => {
  return Math.min(Math.max(((startZ - cameraZOffset.value) / totalDepthRange) * 100, 0), 100);
});

// 当前高亮的时间节点索引（滑动时的目标卡片索引），符合 Google 编码规范，使用 ref 进行响应式跟踪
const currentIndex = ref(0);

// 动画锁：防止快速连续滑动或滚轮惯性导致一次滚过多张图片
const isAnimating = ref(false);

// 计算当前视窗最贴近的卡片索引，用以作为卡片翻页边界触发判定
// 这里直接绑定目标卡片索引，使翻页反馈与声效触发更为即时和准确
const activeCardIndex = computed(() => {
  return currentIndex.value;
});

// 全屏长图预览模态框控制状态，符合 Google 编码规范
const isLightboxOpen = ref(false);
const lightboxPhoto = ref("");

/**
 * 开启全屏大图预览模态框。
 * 符合 Google 编码规范，利用中文进行详尽功能性注释。
 *
 * @param {string} photoName 待放大的图片文件名。
 */
function openLightbox(photoName: string): void {
  lightboxPhoto.value = photoName;
  isLightboxOpen.value = true;
}

/**
 * 关闭全屏大图预览模态框。
 * 符合 Google 编码规范，利用中文进行详尽功能性注释。
 */
function closeLightbox(): void {
  isLightboxOpen.value = false;
  lightboxPhoto.value = "";
}

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
let hasSwiped = false; // 标记当前滑动手势内是否已完成切换卡片，防止单次长滑连续触发多张卡片

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
  window.addEventListener("resize", handleResize);
});

onUnmounted(() => {
  window.removeEventListener("resize", handleResize);
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
  const canvas = document.createElement("canvas");
  canvas.width = 32;
  canvas.height = 32;
  const ctx = canvas.getContext("2d");
  if (ctx) {
    const gradient = ctx.createRadialGradient(16, 16, 0, 16, 16, 16);
    gradient.addColorStop(0, "rgba(255, 255, 255, 1)");
    gradient.addColorStop(0.2, "rgba(255, 94, 140, 1)");
    gradient.addColorStop(0.5, "rgba(255, 210, 122, 0.45)");
    gradient.addColorStop(1, "rgba(0, 0, 0, 0)");
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
      size: sizeMod,
    });
  }
}

function initThree() {
  if (!canvasContainer.value) return;

  const width = canvasContainer.value.clientWidth || window.innerWidth;
  const height = canvasContainer.value.clientHeight || window.innerHeight;

  // Scene
  scene = new THREE.Scene();
  scene.fog = new THREE.FogExp2("#140411", 0.0035);

  // Camera settings
  // 增大远裁剪面到 1000 以看清更远的星空粒子
  camera = new THREE.PerspectiveCamera(60, width / height, 1, 1000);
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
    // 星空粒子分布在整个区间 [endZ - 100, startZ + 100]
    const z = Math.random() * (startZ + 200) - 100;

    starPositions[i * 3] = Math.cos(angle) * radius;
    starPositions[i * 3 + 1] = Math.sin(angle) * radius;
    starPositions[i * 3 + 2] = z;
  }

  starsGeometry.setAttribute("position", new THREE.BufferAttribute(starPositions, 3));

  starsMaterial = new THREE.PointsMaterial({
    size: 2.3,
    map: createTunnelStarTexture(),
    transparent: true,
    depthWrite: false,
    blending: THREE.AdditiveBlending,
    opacity: 0.9,
  });

  starsPoints = new THREE.Points(starsGeometry, starsMaterial);
  scene.add(starsPoints);

  // 2. 动态计算霓虹光圈的等距排布，自起点 startZ 平铺至终点 endZ
  neonGates = new THREE.Group();
  const ringGeom = new THREE.TorusGeometry(32, 0.3, 8, 32);
  const gateCount = 10;
  const gateDepths: number[] = [];
  for (let i = 0; i < gateCount; i++) {
    gateDepths.push(startZ - i * (totalDepthRange / (gateCount - 1)));
  }
  const ringColors = ["#ff5e8c", "#ffd27a", "#c84095", "#ffe08a"];

  gateDepths.forEach((zDepth, idx) => {
    const mat = new THREE.MeshBasicMaterial({
      color: new THREE.Color(ringColors[idx % ringColors.length]),
      transparent: true,
      opacity: 0.6,
      wireframe: true,
    });
    const ringMesh = new THREE.Mesh(ringGeom, mat);
    ringMesh.position.set(0, 0, zDepth);
    neonGates.add(ringMesh);

    // Inner glowing ring lines
    const glowRingGeom = new THREE.TorusGeometry(32.4, 0.05, 4, 32);
    const glowRing = new THREE.Mesh(
      glowRingGeom,
      new THREE.MeshBasicMaterial({
        color: new THREE.Color(ringColors[idx % ringColors.length]),
        transparent: true,
        opacity: 0.3,
      }),
    );
    glowRing.position.set(0, 0, zDepth);
    neonGates.add(glowRing);
  });
  scene.add(neonGates);

  // 3. Generate soft Space Nebula backgrounds
  const createNebulaTexture = (coreColor: string, haloColor: string): THREE.Texture => {
    const canvas = document.createElement("canvas");
    canvas.width = 128;
    canvas.height = 128;
    const ctx = canvas.getContext("2d");
    if (ctx) {
      const gradient = ctx.createRadialGradient(64, 64, 0, 64, 64, 64);
      gradient.addColorStop(0, coreColor);
      gradient.addColorStop(0.6, haloColor);
      gradient.addColorStop(1, "rgba(0,0,0,0)");
      ctx.fillStyle = gradient;
      ctx.fillRect(0, 0, 128, 128);
    }
    return new THREE.CanvasTexture(canvas);
  };

  nebulaGroup = new THREE.Group();
  const nebulaGeom = new THREE.PlaneGeometry(150, 150);
  const textures = [createNebulaTexture("rgba(255, 94, 140, 0.22)", "rgba(200, 64, 149, 0.08)"), createNebulaTexture("rgba(255, 210, 122, 0.18)", "rgba(255, 94, 140, 0.05)"), createNebulaTexture("rgba(200, 64, 149, 0.2)", "rgba(71, 163, 255, 0.06)")];

  // 动态等距排布星云深度
  const nebulaCount = 6;
  const nebulaZCoords: number[] = [];
  for (let i = 0; i < nebulaCount; i++) {
    nebulaZCoords.push(startZ - i * (totalDepthRange / (nebulaCount - 1)));
  }
  nebulaZCoords.forEach((zVal, idx) => {
    const mat = new THREE.MeshBasicMaterial({
      map: textures[idx % textures.length],
      transparent: true,
      depthWrite: false,
      blending: THREE.AdditiveBlending,
      opacity: 0.75,
    });
    const mesh = new THREE.Mesh(nebulaGeom, mat);
    mesh.position.set((Math.random() - 0.5) * 55, (Math.random() - 0.5) * 55, zVal);
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
    depthWrite: false,
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
  // 关键性能优化：如果该卡片不是当前活跃卡片，也不是前后相邻的卡片，
  // 则直接返回 'hidden' 将其在 DOM 树中隐藏（display: none）。
  // 这样做能让移动端浏览器的复合管理器（Compositor）免于在 WebGL 画布上方堆叠并混合计算数十个带有
  // 高对比度阴影和滤镜的透明卡片图层，从而彻底消除移动端常见的渲染黑影、闪烁、照片遮挡等 GPU 缺陷。
  if (Math.abs(index - currentIndex.value) > 1) {
    return "hidden";
  }

  const card = timelineCards[index];
  const distance = Math.abs(cameraZOffset.value - card.depth);

  if (distance < 35) {
    // Current Card: High glow, perfectly centered
    return "opacity-100 scale-100 translate-y-0 rotate-0 z-30 pointer-events-auto border-[#ffd27a]/60 shadow-[0_0_35px_rgba(255,210,122,0.45)]";
  } else if (cameraZOffset.value > card.depth) {
    // Upcoming Card: Placed behind, fading out, scaled down
    return "opacity-0 scale-75 translate-y-[120px] rotate-[-6deg] z-10 pointer-events-none";
  } else {
    // Passed Card: Zoomed past, rotated upwards, vanished
    return "opacity-0 scale-125 translate-y-[-140px] rotate-[6deg] z-0 pointer-events-none";
  }
}

// Touch Event Parsing
function onTouchStart(e: TouchEvent) {
  isDragging = true;
  startY = e.touches[0].clientY;
  hasSwiped = false;
}

function onTouchMove(e: TouchEvent) {
  // 核心优化：为了确保手机端滑动的流畅度，去除了 isAnimating.value 限制。
  // 这允许用户在上一张卡片过渡动画未结束时，立即通过下手势继续无缝翻页。
  // 同时，继续保持 hasSwiped 限制，确保单次滑动手势（从 TouchStart 到 TouchEnd 之间）只允许翻过 1 张图片，
  // 避免长距离滑动导致一次性滚过多张卡片。
  if (!isDragging || hasSwiped) {
    if (isDragging) {
      spawnTrailAt(e.touches[0].clientX, e.touches[0].clientY);
    }
    return;
  }
  currentY = e.touches[0].clientY;
  const deltaY = currentY - startY;

  // 设定触发翻页的阈值（50像素）
  if (deltaY < -50) {
    // 向上滑动，进入下一张卡片（Z 轴减少）
    const nextIdx = Math.min(currentIndex.value + 1, timelineCards.length - 1);
    if (nextIdx !== currentIndex.value) {
      currentIndex.value = nextIdx;
      animateToCard(nextIdx);
    }
    hasSwiped = true;
  } else if (deltaY > 50) {
    // 向下滑动，返回上一张卡片（Z 轴增加）
    const prevIdx = Math.max(currentIndex.value - 1, 0);
    if (prevIdx !== currentIndex.value) {
      currentIndex.value = prevIdx;
      animateToCard(prevIdx);
    }
    hasSwiped = true;
  }

  spawnTrailAt(e.touches[0].clientX, e.touches[0].clientY);
}

function onTouchEnd() {
  isDragging = false;
  hasSwiped = false;
}

// Mouse event support for desktop preview users
function onMouseDown(e: MouseEvent) {
  isDragging = true;
  startY = e.clientY;
  hasSwiped = false;
}

function onMouseMove(e: MouseEvent) {
  // 核心优化：同样对 PC 端拖曳手势去除 isAnimating.value 限制，以提升交互的连贯性和响应度。
  // 并依赖 hasSwiped 机制保障单次拖动（从MouseDown到MouseUp）只切换 1 张卡片。
  if (!isDragging || hasSwiped) {
    if (isDragging) {
      spawnTrailAt(e.clientX, e.clientY);
    }
    return;
  }
  currentY = e.clientY;
  const deltaY = currentY - startY;

  // 设定触发翻页的阈值（50像素）
  if (deltaY < -50) {
    // 向上滑动，进入下一张卡片
    const nextIdx = Math.min(currentIndex.value + 1, timelineCards.length - 1);
    if (nextIdx !== currentIndex.value) {
      currentIndex.value = nextIdx;
      animateToCard(nextIdx);
    }
    hasSwiped = true;
  } else if (deltaY > 50) {
    // 向下滑动，返回上一张卡片
    const prevIdx = Math.max(currentIndex.value - 1, 0);
    if (prevIdx !== currentIndex.value) {
      currentIndex.value = prevIdx;
      animateToCard(prevIdx);
    }
    hasSwiped = true;
  }

  spawnTrailAt(e.clientX, e.clientY);
}

function onMouseUp() {
  isDragging = false;
  hasSwiped = false;
}

/**
 * 启动 GSAP 平滑插值动画，将相机 Z 轴精确对齐到指定卡片的深度。
 * 使用 overwrite: 'auto' 防止高频滑动时的动画冲突，确保过渡自然流畅。
 * 在动画启动时激活锁定状态 (isAnimating = true)，动画结束 (onComplete) 后解除锁定。
 * 符合 Google 编码规范，使用详细的中文注释。
 *
 * @param {number} targetIdx 目标卡片的索引。
 */
function animateToCard(targetIdx: number): void {
  const targetDepth = timelineCards[targetIdx].depth;
  isAnimating.value = true;
  gsap.to(cameraZOffset, {
    value: targetDepth,
    duration: 0.65,
    ease: "power2.out",
    overwrite: "auto",
    onComplete: () => {
      isAnimating.value = false;
    },
  });
}

/**
 * 鼠标滚轮事件处理器。
 * 针对 PC 端提供精确、分步的滚轮切换效果，并加入了 isAnimating 动画锁机制，防止一次滚动连跳多张卡片。
 * 符合 Google 编码规范，使用详细的中文注释。
 *
 * @param {WheelEvent} e 滚轮事件对象。
 */
function onWheel(e: WheelEvent): void {
  // 如果当前正在播放动画，则直接拦截，防止惯性连续触发
  if (isAnimating.value) {
    return;
  }

  if (e.deltaY > 0) {
    // 滚轮向下滚：进入下一个时间节点
    const nextIdx = Math.min(currentIndex.value + 1, timelineCards.length - 1);
    if (nextIdx !== currentIndex.value) {
      currentIndex.value = nextIdx;
      animateToCard(nextIdx);
    }
  } else if (e.deltaY < 0) {
    // 滚轮向上滚：返回上一个时间节点
    const prevIdx = Math.max(currentIndex.value - 1, 0);
    if (prevIdx !== currentIndex.value) {
      currentIndex.value = prevIdx;
      animateToCard(prevIdx);
    }
  }
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
    if (audioCtx.state === "suspended") {
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
    bandpassFilter.type = "bandpass";
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

/* 自定义大图预览滚动条样式，深度契合黑金及霓虹粉整体视觉系统 */
.custom-scrollbar::-webkit-scrollbar {
  width: 5px;
}
.custom-scrollbar::-webkit-scrollbar-track {
  background: rgba(255, 255, 255, 0.03);
  border-radius: 4px;
}
.custom-scrollbar::-webkit-scrollbar-thumb {
  background: rgba(255, 94, 140, 0.4);
  border-radius: 4px;
}
.custom-scrollbar::-webkit-scrollbar-thumb:hover {
  background: rgba(255, 94, 140, 0.7);
}
</style>

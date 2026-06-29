<template>
  <div class="app-root">
    <Transition name="fade-stage">
      <div
        v-if="isPlaying"
        class="fullscreen-stage"
        :style="stageStyle"
        @dblclick="exitFullscreenLogic"
        @touchstart="onTouchStart"
        @touchend="onTouchEnd"
        @touchmove="onTouchMove"
      >
        <canvas
          v-if="config.fontEffect === 'pixel' || config.fontEffect === 'particle'"
          ref="particleCanvasRef"
          class="particle-canvas"
        ></canvas>
        <div class="marquee-wrapper" :class="{ 'is-static': config.scrollDir === 'static' }">
          <div ref="fullRibbonRef" class="marquee-ribbon">
            <span
              v-for="n in 6" :key="n"
              class="stage-glyph"
              :class="[
                { 'pixel-mode': config.fontEffect === 'pixel' },
                config.animateEffect ? 'animate__animated animate__infinite animate__' + config.animateEffect : ''
              ]"
              :style="textStyle"
            >{{ config.text }}</span>
          </div>
        </div>
      </div>
    </Transition>

    <Transition name="slide-editor">
      <div v-if="!isPlaying" class="editor-layout">
        <div class="preview-theater" :style="stageStyle">
          <div class="phone-frame">
            <div class="frame-screen" :style="stageStyle">
              <div class="mini-marquee" :class="{ 'is-static': config.scrollDir === 'static' }">
                <div ref="prevRibbonRef" class="mini-ribbon">
                  <span
                    v-for="n in 4" :key="n"
                    class="preview-glyph"
                    :class="[
                      { 'pixel-mode': config.fontEffect === 'pixel' },
                      config.animateEffect ? 'animate__animated animate__infinite animate__' + config.animateEffect : ''
                    ]"
                    :style="textStyle"
                  >{{ config.text }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>

        <div class="config-area">
          <div class="card glass-card">
            <span class="label">弹幕</span>
            <div class="input-row">
              <input v-model="config.text" class="neo-input" placeholder="输入应援弹幕..." />
              <button v-if="config.text" class="clear-inline" @click="config.text = ''">✕</button>
            </div>
            <div class="mt-12">
              <span class="sub-label">配置名称</span>
              <input v-model="presetName" class="neo-input sm" placeholder="例如：生日快乐2026" />
            </div>
          </div>

          <div class="card glass-card">
            <span class="label">配色方案</span>
            <div class="color-swatch-grid">
              <div
                v-for="(t, i) in themes" :key="i"
                class="swatch-item"
                :class="{ active: currentThemeIdx === i }"
                :style="{ background: t.bg, color: t.color, textShadow: currentThemeIdx === i ? 'none' : `0 0 4px ${t.glow}` }"
                @click="applyTheme(i)"
              >
                <span class="swatch-text">Aa</span>
              </div>
            </div>
          </div>

          <div class="card glass-card">
            <span class="label">字体特效</span>
            <div class="pill-group mini">
              <button class="pill sm" :class="{ active: config.fontEffect === 'normal' }" @click="setFontEffect('normal')">常规</button>
              <button class="pill sm" :class="{ active: config.fontEffect === 'pixel' }" @click="setFontEffect('pixel')">像素风</button>
              <button class="pill sm" :class="{ active: config.fontEffect === 'particle' }" @click="setFontEffect('particle')">粒子风</button>
            </div>
          </div>

          <div class="card glass-card">
            <div class="slider-row">
              <span class="label">文字填充模式</span>
              <div class="pill-group mini">
                <button class="pill sm" :class="{ active: config.textFillMode === 'solid' }" @click="config.textFillMode = 'solid'">纯色</button>
                <button class="pill sm" :class="{ active: config.textFillMode === 'gradient' }" @click="config.textFillMode = 'gradient'">渐变</button>
                <button class="pill sm" :class="{ active: config.textFillMode === 'rainbow' }" @click="config.textFillMode = 'rainbow'">彩虹</button>
              </div>
            </div>

            <div v-if="config.textFillMode === 'solid'" class="slider-row mt-12">
              <span class="label">文字颜色</span>
              <label class="color-swatch-sm" :style="{ background: config.textColor }">
                <input type="color" v-model="config.textColor" class="hidden-color-input" />
              </label>
            </div>

            <template v-if="config.textFillMode === 'gradient'">
              <div class="slider-row mt-12">
                <span class="label">起始色 / 结束色</span>
                <div style="display:flex; gap:8px; align-items:center;">
                  <label class="color-swatch-sm" :style="{ background: config.textGradStart }"><input type="color" v-model="config.textGradStart" class="hidden-color-input" /></label>
                  <label class="color-swatch-sm" :style="{ background: config.textGradEnd }"><input type="color" v-model="config.textGradEnd" class="hidden-color-input" /></label>
                </div>
              </div>
              <div class="slider-row mt-8">
                <span class="label">渐变角度</span>
                <span class="val">{{ config.textGradAngle }}°</span>
              </div>
              <div class="ruler-slider mt-4"><input type="range" v-model.number="config.textGradAngle" min="0" max="360" @input="updateSliderVar" /><div class="ticks"></div></div>
            </template>

            <div class="slider-row mt-16">
              <span class="label">文字大小</span>
              <span class="val">{{ config.fontSize }}px</span>
            </div>
            <div class="ruler-slider"><input type="range" v-model.number="config.fontSize" min="40" max="300" @input="updateSliderVar" /><div class="ticks"></div></div>

            <div class="slider-row mt-16">
              <span class="label">荧光强度</span>
              <span class="val">{{ config.glowIntensity }}%</span>
            </div>
            <div class="ruler-slider"><input type="range" v-model.number="config.glowIntensity" min="0" max="200" @input="updateSliderVar" /><div class="ticks"></div></div>

            <div class="font-grid mt-16">
              <button v-for="f in fonts" :key="f" class="font-btn" :class="{ active: config.font === f }" @click="config.font = f">{{ f }}</button>
            </div>
          </div>

          <div class="card glass-card">
            <div class="slider-row">
              <span class="label">背景填充模式</span>
              <div class="pill-group mini">
                <button class="pill sm" :class="{ active: config.bgFillMode === 'solid' }" @click="config.bgFillMode = 'solid'">纯色</button>
                <button class="pill sm" :class="{ active: config.bgFillMode === 'gradient' }" @click="config.bgFillMode = 'gradient'">渐变</button>
                <button class="pill sm" :class="{ active: config.bgFillMode === 'rainbow' }" @click="config.bgFillMode = 'rainbow'">彩虹</button>
              </div>
            </div>
            <div v-if="config.bgFillMode === 'solid'" class="slider-row mt-12">
              <span class="label">背景颜色</span>
              <label class="color-swatch-sm" :style="{ background: config.bgColor }"><input type="color" v-model="config.bgColor" class="hidden-color-input" /></label>
            </div>
            <template v-if="config.bgFillMode === 'gradient'">
              <div class="slider-row mt-12">
                <span class="label">起始色 / 结束色</span>
                <div style="display:flex; gap:8px; align-items:center;">
                  <label class="color-swatch-sm" :style="{ background: config.bgGradStart }"><input type="color" v-model="config.bgGradStart" class="hidden-color-input" /></label>
                  <label class="color-swatch-sm" :style="{ background: config.bgGradEnd }"><input type="color" v-model="config.bgGradEnd" class="hidden-color-input" /></label>
                </div>
              </div>
              <div class="slider-row mt-8"><span class="label">渐变角度</span><span class="val">{{ config.bgGradAngle }}°</span></div>
              <div class="ruler-slider mt-4"><input type="range" v-model.number="config.bgGradAngle" min="0" max="360" @input="updateSliderVar" /><div class="ticks"></div></div>
            </template>
            <div v-if="config.bgFillMode === 'solid'" class="bg-presets mt-12">
              <span class="sub-label">背景建议</span>
              <div class="color-swatch-grid small">
                <div v-for="(c, i) in bgPresets" :key="i" class="swatch-item sm" :class="{ active: config.bgColor === c }" :style="{ background: c }" @click="config.bgColor = c; config.bgFillMode = 'solid'"></div>
              </div>
            </div>
          </div>

          <div class="card glass-card">
            <span class="label">形变特效</span>
            <div class="fx-grid wide">
              <button
                v-for="fx in animateFx" :key="fx.val"
                class="fx-btn"
                :class="{ active: config.animateEffect === fx.val }"
                @click="config.animateEffect = fx.val"
              >{{ fx.label }}</button>
            </div>
          </div>

          <div class="card glass-card">
            <span class="label">光影律动</span>
            <div class="fx-grid wide">
              <button
                v-for="fx in gsapFx" :key="fx.val"
                class="fx-btn glow-fx"
                :class="{ active: config.gsapEffect === fx.val }"
                @click="config.gsapEffect = fx.val"
              >{{ fx.label }}</button>
            </div>
          </div>

          <div class="card glass-card">
            <div class="slider-row"><span class="label">滚动方向</span></div>
            <div class="dir-pills">
              <button class="pill dir" :class="{ active: config.scrollDir === 'static' }" @click="config.scrollDir = 'static'">◉ 静态</button>
              <button class="pill dir" :class="{ active: config.scrollDir === 'rtl' }" @click="config.scrollDir = 'rtl'">← 向左</button>
              <button class="pill dir" :class="{ active: config.scrollDir === 'ltr' }" @click="config.scrollDir = 'ltr'">向右 →</button>
              <button class="pill dir" :class="{ active: config.scrollDir === 'ttb' }" @click="config.scrollDir = 'ttb'">↓ 向下</button>
              <button class="pill dir" :class="{ active: config.scrollDir === 'btt' }" @click="config.scrollDir = 'btt'">↑ 向上</button>
            </div>
            <div class="mt-16">
              <div class="slider-row"><span class="label">滚动速度</span><span class="val">{{ config.speed }}</span></div>
              <div class="ruler-slider"><input type="range" v-model.number="config.speed" min="1" max="1000" @input="updateSliderVar" /><div class="ticks"></div></div>
            </div>
          </div>

          <div class="card glass-card">
            <span class="label">镜像模式</span>
            <div class="mirror-grid">
              <button class="pill mirror" :class="{ active: config.mirror === 'none' }" @click="config.mirror = 'none'">保持一致</button>
              <button class="pill mirror" :class="{ active: config.mirror === 'horizontal' }" @click="config.mirror = 'horizontal'">↔ 左右镜像</button>
              <button class="pill mirror" :class="{ active: config.mirror === 'vertical' }" @click="config.mirror = 'vertical'">↕ 上下镜像</button>
              <button class="pill mirror" :class="{ active: config.mirror === 'both' }" @click="config.mirror = 'both'">⇄⇅ 双向镜像</button>
            </div>
          </div>

          <div class="card glass-card row-card">
            <span class="label">全屏方向</span>
            <div class="pill-group mini">
              <button class="pill sm" :class="{ active: config.orientation === 'landscape' }" @click="config.orientation = 'landscape'">横屏</button>
              <button class="pill sm" :class="{ active: config.orientation === 'portrait' }" @click="config.orientation = 'portrait'">竖屏</button>
            </div>
          </div>

          <div class="card glass-card history-card">
            <span class="label">历史配置</span>
            <div v-if="historyList.length" class="history-list">
              <div v-for="(h, i) in historyList" :key="i" class="history-item" @click.stop="loadHistory(h)">
                <span class="h-name">{{ h.name }}</span>
                <span class="h-preview">{{ h.text }}</span>
              </div>
            </div>
            <div v-else class="history-empty">暂无保存的配置，点击底部“保存配置”即可添加</div>
          </div>
          <div class="spacer"></div>
        </div>

        <div class="bottom-bar">
          <button class="action-btn secondary" @click="savePreset">保存配置</button>
          <button class="action-btn primary" @click="startPlay">开始播放</button>
        </div>
      </div>
    </Transition>
  </div>
</template>

<script setup>
import { ref, reactive, computed, watch, onMounted, onUnmounted, nextTick } from 'vue';
import gsap from 'gsap';
import { CSSPlugin } from 'gsap/CSSPlugin';
gsap.registerPlugin(CSSPlugin);

const isPlaying = ref(false);
const currentThemeIdx = ref(3);
const presetName = ref('');
const historyList = ref([]);
const fullRibbonRef = ref(null);
const prevRibbonRef = ref(null);
const particleCanvasRef = ref(null);
let fullTween = null;
let prevTween = null;
let longPressTimer = null;
let touchStartTime = 0;
let particleAnimationId = null;
const LONG_PRESS_DURATION = 800;

const config = reactive({
  text: '',
  orientation: 'landscape',
  font: '系统默认',
  fontEffect: 'normal',
  scrollDir: 'rtl',
  speed: 80,
  animateEffect: '',
  gsapEffect: '',
  glowIntensity: 60,
  fontSize: 140,
  mirror: 'none',
  textFillMode: 'solid',
  textColor: '#b066ff',
  textGradStart: '#ff007f',
  textGradEnd: '#00f0ff',
  textGradAngle: 90,
  bgFillMode: 'solid',
  bgColor: '#0a0a0a',
  bgGradStart: '#0f0c29',
  bgGradEnd: '#302b63',
  bgGradAngle: 135
});

const fonts = ['系统默认', '黑体', '加油体', '可爱体', '卡通体', '像素体', '快乐体', '男神体', '潮流体'];
const themes = [
  { name: '经典白', bg: '#000', color: '#fff', glow: '#fff' },
  { name: '高级灰', bg: '#1c1c1e', color: '#e5e5ea', glow: '#8e8e93' },
  { name: '警示黄', bg: '#000', color: '#ffd60a', glow: '#ffcc00' },
  { name: '赛博紫', bg: '#0a0a0a', color: '#b066ff', glow: '#b066ff' },
  { name: '科技蓝', bg: '#050a14', color: '#0a84ff', glow: '#0a84ff' },
  { name: '暗夜绿', bg: '#05140a', color: '#30d158', glow: '#30d158' },
  { name: '热烈红', bg: '#140505', color: '#ff453a', glow: '#ff453a' },
  { name: '清新绿', bg: '#05140a', color: '#a8e063', glow: '#56ab2f' },
  { name: '少女粉', bg: '#14050a', color: '#ff2d55', glow: '#ff2d55' },
  { name: '极光青', bg: '#051414', color: '#64d2ff', glow: '#5ac8fa' }
];
const bgPresets = ['#000000', '#1a1a1a', '#2d2d2d', '#0a0a0a', '#050505', '#1c1c1e', '#0f0c29', '#050a14', '#140505', '#05140a'];
const animateFx = [
  { label: '无', val: '' },
  { label: '弹跳', val: 'bounce' }, { label: '闪烁', val: 'flash' }, { label: '脉冲', val: 'pulse' },
  { label: '橡皮筋', val: 'rubberBand' }, { label: '抖动X', val: 'shakeX' }, { label: '抖动Y', val: 'shakeY' },
  { label: '摇头', val: 'headShake' }, { label: '摇摆', val: 'swing' }, { label: '铃铛', val: 'tada' },
  { label: '晃动', val: 'wobble' }, { label: '果冻', val: 'jello' }, { label: '心跳', val: 'heartBeat' },
  { label: '淡入', val: 'fadeIn' }, { label: '淡入下', val: 'fadeInDown' }, { label: '淡入左', val: 'fadeInLeft' },
  { label: '淡入右', val: 'fadeInRight' }, { label: '淡入上', val: 'fadeInUp' }, { label: '淡出', val: 'fadeOut' },
  { label: '淡出下', val: 'fadeOutDown' }, { label: '淡出左', val: 'fadeOutLeft' }, { label: '淡出右', val: 'fadeOutRight' },
  { label: '淡出上', val: 'fadeOutUp' }, { label: '翻转', val: 'flip' }, { label: '翻转X', val: 'flipInX' },
  { label: '翻转Y', val: 'flipInY' }, { label: '翻转出X', val: 'flipOutX' }, { label: '翻转出Y', val: 'flipOutY' },
  { label: '光速入', val: 'lightSpeedInRight' }, { label: '光速出', val: 'lightSpeedOutRight' },
  { label: '旋转入', val: 'rotateIn' }, { label: '旋转入下左', val: 'rotateInDownLeft' }, { label: '旋转入下右', val: 'rotateInDownRight' },
  { label: '旋转入上左', val: 'rotateInUpLeft' }, { label: '旋转入上右', val: 'rotateInUpRight' }, { label: '旋转出', val: 'rotateOut' },
  { label: '旋转出下左', val: 'rotateOutDownLeft' }, { label: '旋转出下右', val: 'rotateOutDownRight' },
  { label: '旋转出上左', val: 'rotateOutUpLeft' }, { label: '旋转出上右', val: 'rotateOutUpRight' },
  { label: '铰链', val: 'hinge' }, { label: '盒子入', val: 'jackInTheBox' }, { label: '滚入', val: 'rollIn' },
  { label: '滚出', val: 'rollOut' }, { label: '缩放入', val: 'zoomIn' }, { label: '缩放入下', val: 'zoomInDown' },
  { label: '缩放入左', val: 'zoomInLeft' }, { label: '缩放入右', val: 'zoomInRight' }, { label: '缩放入上', val: 'zoomInUp' },
  { label: '缩放退', val: 'zoomOut' }, { label: '缩放退下', val: 'zoomOutDown' }, { label: '缩放退左', val: 'zoomOutLeft' },
  { label: '缩放退右', val: 'zoomOutRight' }, { label: '缩放退上', val: 'zoomOutUp' }, { label: '滑入下', val: 'slideInDown' },
  { label: '滑入左', val: 'slideInLeft' }, { label: '滑入右', val: 'slideInRight' }, { label: '滑入上', val: 'slideInUp' },
  { label: '滑出下', val: 'slideOutDown' }, { label: '滑出左', val: 'slideOutLeft' }, { label: '滑出右', val: 'slideOutRight' },
  { label: '滑出上', val: 'slideOutUp' }
];
const gsapFx = [
  { label: '无', val: '' }, { label: '霓虹呼吸', val: 'breathe' }, { label: '律动缩放', val: 'scale' },
  { label: '赛博倾斜', val: 'tilt' }, { label: '色彩流转', val: 'hueShift' }, { label: '弹性震颤', val: 'elastic' },
  { label: '慢速呼吸', val: 'slowBreathe' }, { label: '快速脉冲', val: 'fastPulse' }, { label: '深度缩放', val: 'deepScale' },
  { label: '微颤', val: 'microShake' }, { label: '左右摆', val: 'sway' }, { label: '上下浮', val: 'float' },
  { label: '频闪', val: 'strobe' }, { label: '弹簧', val: 'spring' }, { label: '心跳模拟', val: 'heartbeat' },
  { label: '呼吸+色移', val: 'breatheHue' }, { label: '弹性缩放', val: 'elasticScale' }, { label: '渐隐脉冲', val: 'fadePulse' },
  { label: '扭曲呼吸', val: 'skewBreathe' }, { label: '随机微动', val: 'randomMicro' }, { label: '交错缩放', val: 'staggerScale' },
  { label: '频闪呼吸', val: 'strobeBreathe' }, { label: '波浪缩放', val: 'waveScale' }, { label: '螺旋', val: 'spiral' },
  { label: '缓入缩放', val: 'easeScale' }, { label: '抖动呼吸', val: 'jitterBreathe' }, { label: '渐进倾斜', val: 'gradualTilt' },
  { label: '双脉冲', val: 'doublePulse' }, { label: '反向呼吸', val: 'reverseBreathe' }, { label: '旋转呼吸', val: 'spinBreathe' },
  { label: '渐强呼吸', val: 'crescendo' }, { label: '漂浮+旋转', val: 'floatSpin' }
];

const previewFontSize = computed(() => Math.round(config.fontSize * 0.3));

const hexToRgb = (hex) => {
  const result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex);
  return result ? `${parseInt(result[1], 16)}, ${parseInt(result[2], 16)}, ${parseInt(result[3], 16)}` : '255, 255, 255';
};

const textStyle = computed(() => {
  const intensity = config.glowIntensity / 100;
  const glowColor = config.textFillMode === 'solid' ? config.textColor : '#ffffff';
  const shadowLayers = [];
  for (let i = 1; i <= 4; i++) {
    const blur = intensity * i * 12;
    const alpha = Math.max(0.2, 1 - i * 0.25);
    shadowLayers.push(`0 0 ${blur}px rgba(${hexToRgb(glowColor)}, ${alpha})`);
  }
  const base = {
    fontSize: `${isPlaying.value ? config.fontSize : previewFontSize.value}px`,
    fontFamily: config.font === '系统默认' ? 'system-ui, sans-serif' : config.font,
    textShadow: shadowLayers.join(', '),
    fontWeight: config.font.includes('粗') || config.font === '加油体' ? '900' : 'normal'
  };
  let transform = '';
  if (config.mirror === 'horizontal') transform = 'scaleX(-1)';
  else if (config.mirror === 'vertical') transform = 'scaleY(-1)';
  else if (config.mirror === 'both') transform = 'scale(-1, -1)';
  if (transform) base.transform = transform;
  if (config.textFillMode === 'solid') {
    base.color = config.textColor;
  } else if (config.textFillMode === 'gradient') {
    base.background = `linear-gradient(${config.textGradAngle}deg, ${config.textGradStart}, ${config.textGradEnd})`;
    base.webkitBackgroundClip = 'text';
    base.webkitTextFillColor = 'transparent';
    base.color = 'transparent';
  } else if (config.textFillMode === 'rainbow') {
    base.background = 'linear-gradient(90deg, #ff0000, #ff7f00, #ffff00, #00ff00, #0000ff, #4b0082, #9400d3)';
    base.webkitBackgroundClip = 'text';
    base.webkitTextFillColor = 'transparent';
    base.color = 'transparent';
  }
  if (config.fontEffect === 'pixel' || config.fontEffect === 'particle') {
    base.opacity = '0';
  }
  return base;
});

const stageStyle = computed(() => {
  let bg = '';
  if (config.bgFillMode === 'solid') bg = config.bgColor;
  else if (config.bgFillMode === 'gradient') bg = `linear-gradient(${config.bgGradAngle}deg, ${config.bgGradStart}, ${config.bgGradEnd})`;
  else if (config.bgFillMode === 'rainbow') bg = 'linear-gradient(90deg, #ff0000, #ff7f00, #ffff00, #00ff00, #0000ff, #4b0082, #9400d3)';
  return { background: bg };
});

const runMarquee = (el) => {
  if (!el || config.scrollDir === 'static') return null;
  const children = el.children;
  if (!children || children.length === 0) return null;
  const singleWidth = children[0].offsetWidth;
  if (singleWidth === 0) return null;
  const duration = Math.max(0.5, singleWidth / config.speed);
  const axis = (config.scrollDir === 'rtl' || config.scrollDir === 'ltr') ? 'x' : 'y';
  const reverse = (config.scrollDir === 'rtl' || config.scrollDir === 'ttb');
  const start = reverse ? 0 : -singleWidth;
  const end = reverse ? -singleWidth : 0;
  gsap.set(el, { [axis]: start });
  return gsap.to(el, {
    [axis]: end,
    duration,
    ease: 'none',
    repeat: -1
  });
};

const updateMarquee = () => {
  nextTick(() => {
    if (fullTween) { fullTween.kill(); fullTween = null; }
    if (prevTween) { prevTween.kill(); prevTween = null; }
    if (isPlaying.value && fullRibbonRef.value) {
      fullTween = runMarquee(fullRibbonRef.value);
    } else if (!isPlaying.value && prevRibbonRef.value) {
      prevTween = runMarquee(prevRibbonRef.value);
    }
  });
};

watch(() => [config.text, config.fontSize, config.font, config.scrollDir, config.speed, isPlaying.value], updateMarquee, { flush: 'post' });

const applyEffectsToElements = (containerRef, className) => {
  if (!containerRef.value) return;
  const els = containerRef.value.querySelectorAll(className);
  els.forEach(el => applySingleEffect(el));
};

const applySingleEffect = (targetEl) => {
  if (!targetEl) return;
  gsap.killTweensOf(targetEl);
  if (config.gsapEffect) {
    const type = config.gsapEffect;
    const map = {
      breathe: () => gsap.to(targetEl, { scale: 1.08, repeat: -1, yoyo: true, duration: 1.2, ease: 'sine.inOut' }),
      scale: () => gsap.to(targetEl, { scale: 1.15, repeat: -1, yoyo: true, duration: 0.8, ease: 'power1.inOut' }),
      tilt: () => gsap.to(targetEl, { rotate: 6, repeat: -1, yoyo: true, duration: 0.5, ease: 'power1.inOut' }),
      hueShift: () => gsap.to(targetEl, { hueRotate: 360, repeat: -1, duration: 3, ease: 'none' }),
      elastic: () => gsap.to(targetEl, { scale: 1.1, repeat: -1, yoyo: true, duration: 0.6, ease: 'elastic.out(1,0.3)' }),
      slowBreathe: () => gsap.to(targetEl, { scale: 1.05, repeat: -1, yoyo: true, duration: 2.5, ease: 'sine.inOut' }),
      fastPulse: () => gsap.to(targetEl, { scale: 1.12, repeat: -1, yoyo: true, duration: 0.4, ease: 'power2.inOut' }),
      deepScale: () => gsap.to(targetEl, { scale: 1.25, repeat: -1, yoyo: true, duration: 1.5, ease: 'power2.inOut' }),
      microShake: () => gsap.to(targetEl, { x: 2, repeat: -1, yoyo: true, duration: 0.1, ease: 'none' }),
      sway: () => gsap.to(targetEl, { rotate: 3, repeat: -1, yoyo: true, duration: 1, ease: 'sine.inOut' }),
      float: () => gsap.to(targetEl, { y: -8, repeat: -1, yoyo: true, duration: 1.5, ease: 'sine.inOut' }),
      strobe: () => gsap.to(targetEl, { opacity: 0.3, repeat: -1, yoyo: true, duration: 0.15, ease: 'none' }),
      spring: () => gsap.to(targetEl, { scale: 1.15, repeat: -1, yoyo: true, duration: 0.5, ease: 'elastic.out(1,0.5)' }),
      heartbeat: () => gsap.to(targetEl, { scale: 1.15, repeat: -1, yoyo: true, duration: 0.15, ease: 'power2.out', repeatDelay: 0.5 }),
      breatheHue: () => gsap.to(targetEl, { scale: 1.08, hueRotate: 60, repeat: -1, yoyo: true, duration: 1.5, ease: 'sine.inOut' }),
      elasticScale: () => gsap.to(targetEl, { scale: 1.2, repeat: -1, yoyo: true, duration: 0.8, ease: 'elastic.out(1,0.4)' }),
      fadePulse: () => gsap.to(targetEl, { opacity: 0.5, scale: 1.05, repeat: -1, yoyo: true, duration: 1, ease: 'sine.inOut' }),
      skewBreathe: () => gsap.to(targetEl, { skewX: 5, scale: 1.05, repeat: -1, yoyo: true, duration: 1.2, ease: 'sine.inOut' }),
      randomMicro: () => gsap.to(targetEl, { x: 'random(-3,3)', y: 'random(-3,3)', repeat: -1, yoyo: true, duration: 0.3, ease: 'none' }),
      staggerScale: () => gsap.to(targetEl, { scale: 1.1, repeat: -1, yoyo: true, duration: 0.7, ease: 'back.inOut(1.7)' }),
      strobeBreathe: () => gsap.to(targetEl, { opacity: 0.4, scale: 1.06, repeat: -1, yoyo: true, duration: 0.8, ease: 'sine.inOut' }),
      waveScale: () => gsap.to(targetEl, { scale: 1.1, repeat: -1, yoyo: true, duration: 1, ease: 'sine.inOut', delay: 0.1 }),
      spiral: () => gsap.to(targetEl, { rotate: 360, scale: 1.05, repeat: -1, duration: 3, ease: 'none' }),
      easeScale: () => gsap.to(targetEl, { scale: 1.08, repeat: -1, yoyo: true, duration: 1.8, ease: 'power4.inOut' }),
      jitterBreathe: () => gsap.to(targetEl, { scale: 1.06, x: 1, repeat: -1, yoyo: true, duration: 0.8, ease: 'none' }),
      gradualTilt: () => gsap.to(targetEl, { rotate: 10, repeat: -1, yoyo: true, duration: 2, ease: 'power1.inOut' }),
      doublePulse: () => gsap.to(targetEl, { scale: 1.1, repeat: -1, yoyo: true, duration: 0.3, ease: 'power2.inOut', repeatDelay: 0.3 }),
      reverseBreathe: () => gsap.to(targetEl, { scale: 0.92, repeat: -1, yoyo: true, duration: 1.2, ease: 'sine.inOut' }),
      spinBreathe: () => gsap.to(targetEl, { rotate: 5, scale: 1.05, repeat: -1, yoyo: true, duration: 1.2, ease: 'sine.inOut' }),
      crescendo: () => gsap.to(targetEl, { scale: 1.2, repeat: -1, yoyo: true, duration: 2, ease: 'power3.inOut' }),
      floatSpin: () => gsap.to(targetEl, { y: -10, rotate: 8, repeat: -1, yoyo: true, duration: 1.8, ease: 'sine.inOut' })
    };
    if (map[type]) map[type]();
  }
};

watch(() => [config.gsapEffect], () => {
  nextTick(() => {
    if (isPlaying.value) {
      applyEffectsToElements(fullRibbonRef, '.stage-glyph');
    } else {
      applyEffectsToElements(prevRibbonRef, '.preview-glyph');
    }
  });
});

const stopCanvasEffects = () => {
  if (particleAnimationId) {
    cancelAnimationFrame(particleAnimationId);
    particleAnimationId = null;
  }
  if (particleCanvasRef.value) {
    const ctx = particleCanvasRef.value.getContext('2d');
    ctx.clearRect(0, 0, particleCanvasRef.value.width, particleCanvasRef.value.height);
  }
};

const startPixelText = () => {
  stopCanvasEffects();
  if (!particleCanvasRef.value || !config.text) return;
  const canvas = particleCanvasRef.value;
  const ctx = canvas.getContext('2d');
  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight;
  const fontSize = config.fontSize;
  const fontFamily = config.font === '系统默认' ? 'system-ui, sans-serif' : config.font;
  const text = config.text;
  ctx.font = `900 ${fontSize}px ${fontFamily}`;
  const metrics = ctx.measureText(text);
  const textWidth = metrics.width;
  const textHeight = fontSize;
  const offCanvas = document.createElement('canvas');
  offCanvas.width = textWidth;
  offCanvas.height = textHeight;
  const offCtx = offCanvas.getContext('2d');
  offCtx.font = `900 ${fontSize}px ${fontFamily}`;
  offCtx.fillStyle = '#ffffff';
  offCtx.fillText(text, 0, textHeight * 0.8);
  const imageData = offCtx.getImageData(0, 0, textWidth, textHeight).data;
  const particles = [];
  const step = 4;
  const startX = (canvas.width - textWidth) / 2;
  const startY = (canvas.height - textHeight) / 2;
  for (let y = 0; y < textHeight; y += step) {
    for (let x = 0; x < textWidth; x += step) {
      const index = (y * textWidth + x) * 4;
      if (imageData[index + 3] > 128) {
        particles.push({
          x: x + startX,
          y: y + startY,
          originX: x + startX,
          originY: y + startY
        });
      }
    }
  }
  const draw = () => {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    ctx.fillStyle = config.textColor;
    for (const p of particles) {
      ctx.fillRect(Math.round(p.x), Math.round(p.y), step, step);
    }
  };
  draw();
};

const startParticleText = () => {
  stopCanvasEffects();
  if (!particleCanvasRef.value || !config.text) return;
  const canvas = particleCanvasRef.value;
  const ctx = canvas.getContext('2d');
  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight;
  const fontSize = config.fontSize;
  const fontFamily = config.font === '系统默认' ? 'system-ui, sans-serif' : config.font;
  const text = config.text;
  ctx.font = `900 ${fontSize}px ${fontFamily}`;
  const metrics = ctx.measureText(text);
  const textWidth = metrics.width;
  const textHeight = fontSize;
  const offCanvas = document.createElement('canvas');
  offCanvas.width = textWidth;
  offCanvas.height = textHeight;
  const offCtx = offCanvas.getContext('2d');
  offCtx.font = `900 ${fontSize}px ${fontFamily}`;
  offCtx.fillStyle = '#ffffff';
  offCtx.fillText(text, 0, textHeight * 0.8);
  const imageData = offCtx.getImageData(0, 0, textWidth, textHeight).data;
  const particles = [];
  const step = 4;
  const startX = (canvas.width - textWidth) / 2;
  const startY = (canvas.height - textHeight) / 2;
  for (let y = 0; y < textHeight; y += step) {
    for (let x = 0; x < textWidth; x += step) {
      const index = (y * textWidth + x) * 4;
      if (imageData[index + 3] > 128) {
        particles.push({
          x: x + startX,
          y: y + startY,
          originX: x + startX,
          originY: y + startY,
          vx: 0,
          vy: 0
        });
      }
    }
  }
  const animate = () => {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    for (const p of particles) {
      p.vx += (Math.random() - 0.5) * 1;
      p.vy += (Math.random() - 0.5) * 1;
      p.x += p.vx;
      p.y += p.vy;
      p.vx *= 0.98;
      p.vy *= 0.98;
      p.x += (p.originX - p.x) * 0.05;
      p.y += (p.originY - p.y) * 0.05;
      ctx.fillStyle = config.textColor;
      ctx.fillRect(p.x, p.y, 3, 3);
    }
    particleAnimationId = requestAnimationFrame(animate);
  };
  animate();
};

const setFontEffect = (mode) => {
  config.fontEffect = mode;
  stopCanvasEffects();
  if (mode === 'pixel') {
    if (isPlaying.value) nextTick(() => startPixelText());
  } else if (mode === 'particle') {
    if (isPlaying.value) nextTick(() => startParticleText());
  } else {
    if (isPlaying.value) {
      nextTick(() => applyEffectsToElements(fullRibbonRef, '.stage-glyph'));
    } else {
      nextTick(() => applyEffectsToElements(prevRibbonRef, '.preview-glyph'));
    }
  }
};

const applyTheme = (idx) => {
  currentThemeIdx.value = idx;
  const t = themes[idx];
  config.textFillMode = 'solid';
  config.textColor = t.color;
  config.bgFillMode = 'solid';
  config.bgColor = t.bg;
};

const updateSliderVar = (e) => {
  const input = e.target || e;
  if (!input || !input.min) return;
  const min = parseFloat(input.min) || 0;
  const max = parseFloat(input.max) || 100;
  const val = ((input.value - min) / (max - min)) * 100;
  if (input.parentElement) input.parentElement.style.setProperty('--val', val);
};

const startPlay = async () => {
  isPlaying.value = true;
  await document.documentElement.requestFullscreen?.().catch(() => {});
  const targetOrientation = config.orientation || 'landscape';
  if (screen.orientation && screen.orientation.lock) {
    screen.orientation.lock(targetOrientation).catch(() => {});
  }
  window.history.pushState({ page: 'play' }, '');
  nextTick(() => {
    updateMarquee();
    applyEffectsToElements(fullRibbonRef, '.stage-glyph');
    if (config.fontEffect === 'pixel') startPixelText();
    else if (config.fontEffect === 'particle') startParticleText();
  });
};

const exitFullscreenLogic = () => {
  isPlaying.value = false;
  stopCanvasEffects();
  if (document.exitFullscreen && document.fullscreenElement) {
    document.exitFullscreen().catch(() => {});
  }
  if (screen.orientation && screen.orientation.unlock) {
    screen.orientation.unlock();
  }
  gsap.killTweensOf(['.stage-glyph', '.preview-glyph']);
  if (window.history.state?.page === 'play') window.history.back();
};

const onTouchStart = () => {
  touchStartTime = Date.now();
  longPressTimer = setTimeout(() => exitFullscreenLogic(), LONG_PRESS_DURATION);
};
const onTouchEnd = () => {
  clearTimeout(longPressTimer);
  if (Date.now() - touchStartTime < 300) exitFullscreenLogic();
};
const onTouchMove = () => clearTimeout(longPressTimer);

const handleFullscreenChange = () => {
  if (!document.fullscreenElement && isPlaying.value) exitFullscreenLogic();
};
const handlePopState = () => { if (isPlaying.value) exitFullscreenLogic(); };
const handleVisibilityChange = () => { if (document.hidden && isPlaying.value) exitFullscreenLogic(); };

const savePreset = () => {
  const name = presetName.value.trim() || new Date().toISOString().slice(0,19).replace(/[-T:]/g,'') + '配置';
  const preset = { ...config, name, time: Date.now() };
  historyList.value = [preset, ...historyList.value.filter(h => h.name !== name)].slice(0, 20);
  try { localStorage.setItem('cheer_history_v12', JSON.stringify(historyList.value)); } catch(e){}
  presetName.value = '';
};

const loadHistory = (h) => {
  Object.assign(config, h);
  presetName.value = h.name;
  nextTick(() => {
    applyEffectsToElements(prevRibbonRef, '.preview-glyph');
    updateMarquee();
  });
};

onMounted(() => {
  try { const saved = localStorage.getItem('cheer_history_v12'); if (saved) historyList.value = JSON.parse(saved); } catch(e){}
  updateMarquee();
  document.querySelectorAll('.ruler-slider input').forEach(updateSliderVar);
  document.addEventListener('fullscreenchange', handleFullscreenChange);
  window.addEventListener('popstate', handlePopState);
  document.addEventListener('visibilitychange', handleVisibilityChange);
});

onUnmounted(() => {
  stopCanvasEffects();
  document.removeEventListener('fullscreenchange', handleFullscreenChange);
  window.removeEventListener('popstate', handlePopState);
  document.removeEventListener('visibilitychange', handleVisibilityChange);
  clearTimeout(longPressTimer);
});
</script>

<style scoped>
:root {
  --bg: #000;
  --card-bg: rgba(60, 60, 70, 0.75);
  --card-border: rgba(255, 255, 255, 0.15);
  --card-shadow: 0 16px 48px rgba(0, 0, 0, 0.6), inset 0 1px 0 rgba(255, 255, 255, 0.1);
  --primary: #b066ff;
  --primary-glow: rgba(176, 102, 255, 0.4);
  --t1: #fff;
  --t2: #8e8e93;
  --r-lg: 20px;
  --r-md: 14px;
  --r-sm: 10px;
}
.app-root {
  width: 100%;
  height: 100vh;
  overflow: hidden;
  background: var(--bg);
  color: var(--t1);
  font-family: system-ui, -apple-system, sans-serif;
  position: relative;
}
.fullscreen-stage {
  position: fixed;
  inset: 0;
  z-index: 1000;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  cursor: pointer;
}
.particle-canvas {
  position: absolute;
  inset: 0;
  z-index: 0;
  pointer-events: none;
}
.marquee-wrapper {
  width: 100%;
  overflow: hidden;
  display: flex;
  align-items: center;
  position: relative;
  z-index: 1;
}
.marquee-wrapper.is-static {
  justify-content: center;
}
.marquee-ribbon {
  display: inline-flex;
  gap: 0;
  will-change: transform;
  white-space: nowrap;
  align-items: center;
}
.stage-glyph {
  font-weight: 900;
  line-height: 1.2;
  user-select: none;
  -webkit-user-select: none;
  padding: 0 10vw;
}
.pixel-mode {
  text-shadow: none !important;
}
.editor-layout {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  background: linear-gradient(180deg, #0a0a0a 0%, #000 100%);
}
.preview-theater {
  height: 22vh;
  min-height: 140px;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 12px;
  border-bottom: 1px solid var(--card-border);
  flex-shrink: 0;
}
.phone-frame {
  width: 100%;
  max-width: 320px;
  height: 100%;
  border-radius: var(--r-lg);
  border: 2px solid rgba(255, 255, 255, 0.1);
  overflow: hidden;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.6);
  background: #000;
  position: relative;
}
.frame-screen {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
}
.mini-marquee {
  width: 100%;
  overflow: hidden;
  white-space: nowrap;
}
.mini-marquee.is-static {
  display: flex;
  justify-content: center;
}
.mini-ribbon {
  display: inline-flex;
  gap: 0;
  will-change: transform;
  white-space: nowrap;
  align-items: center;
}
.preview-glyph {
  font-weight: 900;
  line-height: 1.2;
  padding: 0 5vw;
}
.config-area {
  flex: 1;
  overflow-y: auto;
  padding: 16px 16px 140px;
}
.glass-card {
  background: var(--card-bg);
  backdrop-filter: blur(30px);
  -webkit-backdrop-filter: blur(30px);
  border: 1px solid var(--card-border);
  border-radius: var(--r-lg);
  padding: 18px;
  margin-bottom: 16px;
  display: flex;
  flex-direction: column;
  gap: 14px;
  box-shadow: var(--card-shadow);
}
.row-card {
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
}
.label {
  font-size: 12px;
  color: var(--t2);
  font-weight: 600;
  letter-spacing: 0.05em;
}
.sub-label {
  font-size: 11px;
  color: var(--t2);
  margin-bottom: 6px;
  display: block;
}
.mt-4 { margin-top: 4px; }
.mt-8 { margin-top: 8px; }
.mt-12 { margin-top: 12px; }
.mt-16 { margin-top: 16px; }
.input-row {
  position: relative;
  display: flex;
  align-items: center;
}
.neo-input {
  width: 100%;
  padding: 14px 16px;
  border-radius: var(--r-md);
  background: rgba(0, 0, 0, 0.3);
  border: 1px solid var(--card-border);
  color: #fff;
  font-size: 16px;
  outline: none;
  box-sizing: border-box;
  transition: all 0.2s;
}
.neo-input.sm {
  padding: 10px 14px;
  font-size: 14px;
}
.neo-input:focus {
  border-color: var(--primary);
  background: rgba(0, 0, 0, 0.5);
}
.clear-inline {
  position: absolute;
  right: 12px;
  background: none;
  border: none;
  color: var(--t2);
  font-size: 14px;
  cursor: pointer;
  padding: 4px;
}
.pill-group {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}
.pill-group.mini {
  gap: 6px;
}
.pill {
  padding: 10px 16px;
  border-radius: 10px;
  border: 1px solid var(--card-border);
  background: rgba(255, 255, 255, 0.03);
  color: var(--t2);
  font-size: 13px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
}
.pill.sm {
  padding: 8px 12px;
  font-size: 12px;
}
.pill.active {
  background: #fff;
  color: #000;
  border-color: #fff;
  box-shadow: 0 2px 10px rgba(255, 255, 255, 0.2);
}
.dir-pills {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 8px;
}
.pill.dir {
  text-align: center;
}
.mirror-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 8px;
}
.pill.mirror {
  text-align: center;
}
.color-swatch-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  justify-content: flex-start;
}
.color-swatch-grid.small {
  gap: 6px;
}
.swatch-item {
  width: 42px;
  height: 42px;
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 2px solid transparent;
  cursor: pointer;
  transition: all 0.2s;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);
  font-size: 15px;
  font-weight: 900;
  position: relative;
}
.swatch-item.sm {
  width: 32px;
  height: 32px;
  font-size: 0;
}
.swatch-item.active {
  transform: scale(1.1);
  border-color: #fff !important;
}
.swatch-text {
  pointer-events: none;
}
.color-swatch-sm {
  display: inline-block;
  width: 36px;
  height: 36px;
  border-radius: 8px;
  border: 2px solid var(--card-border);
  cursor: pointer;
  position: relative;
  overflow: hidden;
  box-sizing: border-box;
}
.hidden-color-input {
  position: absolute;
  top: -10px;
  left: -10px;
  width: calc(100% + 20px);
  height: calc(100% + 20px);
  opacity: 0;
  cursor: pointer;
}
.font-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 8px;
}
.font-btn {
  padding: 12px;
  border-radius: 10px;
  border: 1px solid var(--card-border);
  background: rgba(255, 255, 255, 0.03);
  color: var(--t2);
  font-size: 12px;
  cursor: pointer;
}
.font-btn.active {
  background: #fff;
  color: #000;
  border-color: #fff;
}
.slider-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.val {
  font-size: 13px;
  color: var(--primary);
  font-weight: 700;
}
.ruler-slider {
  position: relative;
  height: 30px;
  display: flex;
  align-items: center;
  margin-top: 4px;
}
.ruler-slider input[type="range"] {
  position: absolute;
  width: 100%;
  height: 100%;
  opacity: 0;
  cursor: pointer;
  z-index: 10;
  margin: 0;
  top: 0;
  left: 0;
}
.ticks {
  position: absolute;
  width: 100%;
  height: 4px;
  background: rgba(255, 255, 255, 0.08);
  border-radius: 2px;
  pointer-events: none;
}
.ruler-slider::before {
  content: '';
  position: absolute;
  height: 18px;
  width: 18px;
  background: #fff;
  border-radius: 50%;
  z-index: 5;
  pointer-events: none;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.5);
  left: calc(var(--val, 50) * 1%);
  transform: translateX(-50%);
}
.fx-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 6px;
}
.fx-btn {
  padding: 8px 4px;
  border-radius: 10px;
  border: 1px solid var(--card-border);
  background: rgba(255, 255, 255, 0.03);
  color: var(--t2);
  font-size: 10px;
  cursor: pointer;
  text-align: center;
  line-height: 1.3;
  transition: all 0.2s;
}
.fx-btn.active {
  background: #fff;
  color: #000;
  border-color: #fff;
}
.fx-btn.glow-fx.active {
  background: linear-gradient(135deg, #fff, #e0e0e0);
  box-shadow: 0 2px 12px rgba(255, 255, 255, 0.3);
}
.history-card {
  background: rgba(40, 40, 48, 0.8);
}
.history-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
  max-height: 200px;
  overflow-y: auto;
}
.history-item {
  padding: 12px 14px;
  border-radius: var(--r-md);
  background: rgba(255, 255, 255, 0.03);
  border: 1px solid var(--card-border);
  cursor: pointer;
  transition: all 0.2s;
  display: flex;
  flex-direction: column;
  gap: 4px;
}
.history-item:active {
  background: rgba(255, 255, 255, 0.1);
  border-color: #fff;
}
.h-name {
  font-size: 12px;
  color: var(--primary);
  font-weight: 600;
}
.h-preview {
  font-size: 13px;
  color: var(--t1);
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
.history-empty {
  padding: 20px;
  text-align: center;
  color: var(--t2);
  font-size: 12px;
  border: 1px dashed var(--card-border);
  border-radius: var(--r-md);
}
.bottom-bar {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  padding: 16px 20px calc(16px + env(safe-area-inset-bottom));
  background: rgba(10, 10, 10, 0.9);
  backdrop-filter: blur(24px);
  border-top: 1px solid var(--card-border);
  display: flex;
  gap: 12px;
  z-index: 10000;
  pointer-events: auto;
}
.action-btn {
  flex: 1;
  padding: 16px;
  border-radius: var(--r-lg);
  border: none;
  font-size: 15px;
  font-weight: 700;
  cursor: pointer;
  transition: transform 0.1s;
  letter-spacing: 0.02em;
}
.action-btn:active {
  transform: scale(0.96);
}
.action-btn.secondary {
  background: rgba(255, 255, 255, 0.1);
  color: var(--t1);
  border: 1px solid var(--card-border);
  backdrop-filter: blur(10px);
}
.action-btn.primary {
  background: linear-gradient(135deg, #F0EEE9, #E8D6D0);
  color: #3A2825;
  border: 0;
  border-radius: 9999px;
  padding: 14px 36px;
  font-size: 16px;
  font-weight: 500;
  box-shadow: 0 3px 12px rgba(228, 208, 202, 0.3);
  transition: all 0.2s ease;
}
.action-btn.primary:active {
  transform: scale(0.96);
  opacity: 0.9;
}
.fade-stage-enter-active,
.fade-stage-leave-active {
  transition: opacity 0.3s;
}
.fade-stage-enter-from,
.fade-stage-leave-to {
  opacity: 0;
}
.slide-editor-enter-active,
.slide-editor-leave-active {
  transition: transform 0.35s cubic-bezier(0.34, 1.56, 0.64, 1);
}
.slide-editor-enter-from,
.slide-editor-leave-to {
  transform: translateY(100%);
}
.spacer {
  height: 20px;
}
</style>
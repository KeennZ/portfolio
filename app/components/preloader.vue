<template>
  <Transition name="fade">
    <div v-if="isLoading" class="preloader-overlay">
      <div class="grid-background"></div>
      <div class="ambient-glow"></div>

      <div class="web-container">
        <div class="web-rotation">
          <div class="web-scale">
            <svg
              class="web-svg"
              viewBox="0 0 1000 1000"
              preserveAspectRatio="xMidYMid slice"
              fill="none"
              xmlns="http://www.w3.org/2000/svg"
            >
              <g class="radiating-lines">
                <line
                  v-for="(line, index) in spokeLines"
                  :key="'spoke-' + index"
                  :x1="line.x1"
                  :y1="line.y1"
                  :x2="line.x2"
                  :y2="line.y2"
                  class="web-line"
                  :style="{
                    '--delay': `${index * 0.045}s`
                  }"
                />
              </g>

              <g class="web-rings">
                <path
                  v-for="(dPath, index) in ringPaths"
                  :key="'ring-' + index"
                  :d="dPath"
                  class="ring-path"
                  :class="{
                    active: progress > (index + 1) * 10
                  }"
                />
              </g>
            </svg>

            <div class="core-ring">
              <div class="core-inner">
                <div class="core-dot"></div>
              </div>
            </div>
          </div>
        </div>

        <div
          class="mask-eyes"
          :class="{
            active: progress > 45
          }"
        >
          <img
            src="/img/eyes.png"
            class="eyes-image"
            alt=""
          />
        </div>
      </div>


    </div>
  </Transition>
</template>

<script setup>
import {
  ref,
  computed,
  onMounted,
  onBeforeUnmount
} from 'vue'

const progress = ref(0)
const isLoading = ref(true)
const emit = defineEmits(['loaded'])

const spokesCount = 12
const ringsCount = 5
const center = 500
const duration = 3400

let animationFrame = null
let startTime = null

const spokeLines = computed(() => {
  const lines = []
  const maxRadius = 900

  for (let i = 0; i < spokesCount; i++) {
    const angle =
      (
        i * 360 / spokesCount
      ) *
      (
        Math.PI / 180
      )

    lines.push({
      x1: center,
      y1: center,
      x2:
        center +
        maxRadius *
        Math.cos(angle),
      y2:
        center +
        maxRadius *
        Math.sin(angle)
    })
  }

  return lines
})

const ringPaths = computed(() => {
  const paths = []
  const ringSpacing = 105
  const startingRadius = 90

  for (
    let rIdx = 1;
    rIdx <= ringsCount;
    rIdx++
  ) {
    const radius =
      startingRadius +
      rIdx * ringSpacing

    let d = ''

    for (
      let i = 0;
      i < spokesCount;
      i++
    ) {
      const angle1 =
        (
          i * 360 / spokesCount
        ) *
        (
          Math.PI / 180
        )

      const angle2 =
        (
          (i + 1) *
          360 /
          spokesCount
        ) *
        (
          Math.PI / 180
        )

      const midAngle =
        (
          (i + 0.5) *
          360 /
          spokesCount
        ) *
        (
          Math.PI / 180
        )

      const x1 =
        center +
        radius *
        Math.cos(angle1)

      const y1 =
        center +
        radius *
        Math.sin(angle1)

      const x2 =
        center +
        radius *
        Math.cos(angle2)

      const y2 =
        center +
        radius *
        Math.sin(angle2)

      const ctrlRadius =
        radius * 0.86

      const cx =
        center +
        ctrlRadius *
        Math.cos(midAngle)

      const cy =
        center +
        ctrlRadius *
        Math.sin(midAngle)

      if (i === 0) {
        d += `
          M
          ${x1.toFixed(1)}
          ${y1.toFixed(1)}
        `
      }

      d += `
        Q
        ${cx.toFixed(1)}
        ${cy.toFixed(1)}
        ${x2.toFixed(1)}
        ${y2.toFixed(1)}
      `
    }

    d += ' Z'
    paths.push(d)
  }

  return paths
})

const easeOutCubic = (t) => {
  return 1 - Math.pow(1 - t, 3)
}

const animateLoader = (timestamp) => {
  if (!startTime) {
    startTime = timestamp
  }

  const elapsed =
    timestamp - startTime

  const rawProgress =
    Math.min(
      elapsed / duration,
      1
    )

  const easedProgress =
    easeOutCubic(
      rawProgress
    )

  progress.value =
    easedProgress * 100

  if (rawProgress < 1) {
    animationFrame =
      requestAnimationFrame(
        animateLoader
      )

    return
  }

  progress.value = 100

  setTimeout(() => {
    isLoading.value = false
    emit('loaded')
  }, 900)
}

onMounted(() => {
  animationFrame =
    requestAnimationFrame(
      animateLoader
    )
})

onBeforeUnmount(() => {
  if (animationFrame) {
    cancelAnimationFrame(
      animationFrame
    )
  }
})
</script>

<style scoped>

@import url('https://googleapis.com');

.preloader-overlay {
  position: fixed;
  inset: 0;
  width: 100vw;
  height: 100vh;
  z-index: 99999;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  background:
    radial-gradient(
      circle at center,
      rgba(
        255,
        35,
        60,
        0.045
      ),
      transparent 45%
    ),
    #0c0204;
  font-family:
    'Space Grotesk',
    sans-serif;
}

.grid-background {
  position: absolute;
  inset: -20%;
  width: 140%;
  height: 140%;
  background-image:
    linear-gradient(
      to right,
      rgba(
        255,
        35,
        60,
        0.045
      ) 1px,
      transparent 1px
    ),
    linear-gradient(
      to bottom,
      rgba(
        255,
        35,
        60,
        0.045
      ) 1px,
      transparent 1px
    );
  background-size: 80px 80px;
  transform:
    perspective(900px)
    rotateX(60deg)
    scale(1.8);
  transform-origin:
    center bottom;
  opacity: 0.4;
  animation:
    gridMove
    12s
    linear
    infinite;
}

@keyframes gridMove {
  from {
    background-position:
      0 0,
      0 0;
  }

  to {
    background-position:
      0 80px,
      0 80px;
  }
}

.ambient-glow {
  position: absolute;
  width: 70vw;
  height: 70vw;
  max-width: 950px;
  max-height: 950px;
  background:
    radial-gradient(
      circle,
      rgba(
        255,
        35,
        60,
        0.12
      ),
      transparent 70%
    );
  filter:
    blur(55px);
  animation:
    ambientPulse
    3s
    ease-in-out
    infinite;
}

@keyframes ambientPulse {
  0%,
  100% {
    transform:
      scale(0.96);
    opacity: 0.45;
  }

  50% {
    transform:
      scale(1.04);
    opacity: 0.75;
  }
}

.web-container {
  position: absolute;
  inset: 0;
  width: 100vw;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  pointer-events: none;
}

.web-rotation {
  position: absolute;
  inset: -10%;
  width: 120%;
  height: 120%;
  display: flex;
  align-items: center;
  justify-content: center;
  transform-origin: center;
  will-change:
    transform,
    opacity;
  animation:
    webRotate
    3.4s
    cubic-bezier(
      0.16,
      1,
      0.3,
      1
    )
    forwards;
}

.web-scale {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  transform-origin: center;
  will-change:
    transform,
    filter;
  animation:
    webScale
    3.4s
    cubic-bezier(
      0.16,
      1,
      0.3,
      1
    )
    forwards;
}

@keyframes webRotate {
  0% {
    transform:
      rotate(-120deg);
    opacity: 0;
  }

  20% {
    opacity: 0.25;
  }

  45% {
    opacity: 0.5;
  }

  70% {
    opacity: 0.8;
  }

  100% {
    transform:
      rotate(0deg);
    opacity: 1;
  }
}

@keyframes webScale {
  0% {
    transform:
      scale(0.72);
    filter:
      blur(10px);
  }

  30% {
    transform:
      scale(0.8);
    filter:
      blur(5px);
  }

  65% {
    transform:
      scale(0.92);
    filter:
      blur(1.5px);
  }

  100% {
    transform:
      scale(1);
    filter:
      blur(0);
  }
}

.web-svg {
  width: 100%;
  height: 100%;
  display: block;
  overflow: visible;
}

.web-line {
  stroke: #ff233c;
  stroke-width: 1.15;
  opacity: 0;
  stroke-dasharray: 1400;
  stroke-dashoffset: 1400;
  filter:
    drop-shadow(
      0 0 5px
      rgba(
        255,
        35,
        60,
        0.7
      )
    );
  animation:
    drawSpoke
    1.15s
    cubic-bezier(
      0.16,
      1,
      0.3,
      1
    )
    forwards;
  animation-delay:
    calc(
      0.35s +
      var(--delay)
    );
}

@keyframes drawSpoke {
  from {
    stroke-dashoffset: 1400;
    opacity: 0;
  }

  30% {
    opacity: 0.75;
  }

  to {
    stroke-dashoffset: 0;
    opacity: 0.32;
  }
}

.ring-path {
  stroke: #ff233c;
  stroke-width: 1.7;
  fill: none;
  opacity: 0;
  transform: scale(0.68);
  transform-origin: 500px 500px;
  filter:
    drop-shadow(
      0 0 6px
      rgba(
        255,
        35,
        60,
        0.7
      )
    );
  transition:
    opacity
    0.55s
    ease,
    transform
    0.95s
    cubic-bezier(
      0.16,
      1,
      0.3,
      1
    );
}

.ring-path.active {
  opacity: 1;
  transform: scale(1);
}

.core-ring {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 105px;
  height: 105px;
  transform:
    translate(
      -50%,
      -50%
    );
  border:
    1px solid
    rgba(
      255,
      35,
      60,
      0.45
    );
  border-radius: 50%;
  box-shadow:
    0 0 25px
    rgba(
      255,
      35,
      60,
      0.23
    ),
    inset
    0 0 25px
    rgba(
      255,
      35,
      60,
      0.1
    );
  animation:
    coreRotate
    8s
    linear
    infinite;
}

@keyframes coreRotate {
  from {
    transform:
      translate(
        -50%,
        -50%
      )
      rotate(0deg);
  }

  to {
    transform:
      translate(
        -50%,
        -50%
      )
      rotate(360deg);
  }
}

.core-inner {
  position: absolute;
  inset: 15px;
  border:
    1px dashed
    rgba(
      255,
      35,
      60,
      0.3
    );
  border-radius: 50%;
  animation:
    coreReverse
    6s
    linear
    infinite;
}

@keyframes coreReverse {
  from {
    transform:
      rotate(0deg);
  }

  to {
    transform:
      rotate(-360deg);
  }
}

.core-dot {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 7px;
  height: 7px;
  transform:
    translate(
      -50%,
      -50%
    );
  border-radius: 50%;
  background: #ffffff;
  box-shadow:
    0 0 8px #ffffff,
    0 0 20px #ff233c,
    0 0 45px #ff233c;
}

.mask-eyes {
  position: absolute;
  top: 50%;
  left: 50%;
  z-index: 5;
  transform:
    translate(
      -50%,
      -50%
    )
    scale(0.55);
  opacity: 0;
  filter: blur(8px);
  transition:
    opacity
    1s
    ease,
    transform
    1.15s
    cubic-bezier(
      0.16,
      1,
      0.3,
      1
    ),
    filter
    1s
    ease;
}

.mask-eyes.active {
  opacity: 0.9;
  transform:
    translate(
      -50%,
      -50%
    )
    scale(1);
  filter: blur(0);
}

.eyes-image {
  width:
    clamp(
      180px,
      28vw,
      420px
    );
  height: auto;
  display: block;
  object-fit: contain;
  opacity: 0.9;
  will-change:
    filter,
    opacity;
  filter:
    drop-shadow(
      0 0 5px
      rgba(
        255,
        255,
        255,
        0.25
      )
    )
    drop-shadow(
      0 0 25px
      rgba(
        255,
        35,
        60,
        0.8
      )
    );
  animation:
    eyesGlow
    3s
    ease-in-out
    infinite;
}

@keyframes eyesGlow {
  0%,
  100% {
    opacity: 0.72;
    filter:
      drop-shadow(
        0 0 4px
        rgba(
          255,
          255,
          255,
          0.2
        )
      )
      drop-shadow(
        0 0 20px
        rgba(
          255,
          35,
          60,
          0.55
        )
      );
  }

  50% {
    opacity: 0.95;
    filter:
      drop-shadow(
        0 0 7px
        rgba(
          255,
          255,
          255,
          0.35
        )
      )
      drop-shadow(
        0 0 28px
        rgba(
          255,
          35,
          60,
          0.8
        )
      );
  }
}

.text-content {
  position: relative;
  z-index: 10;
  text-align: center;
  opacity: 0;
  transform:
    translateY(28px)
    scale(0.96);
  filter: blur(10px);
  transition:
    opacity
    1s
    cubic-bezier(
      0.16,
      1,
      0.3,
      1
    ),
    transform
    1.15s
    cubic-bezier(
      0.16,
      1,
      0.3,
      1
    ),
    filter
    1s
    ease;
}

.brand-name {
  position: absolute;
  z-index: 10;

  top: 70%;
  left: 51%;

  transform:
    translate(-50%, -50%)
    scale(0.92);

  opacity: 0;

  font-family: 'Rajdhani', sans-serif;

  font-size:
    clamp(
      4rem,
      10vw,
      8rem
    );

  letter-spacing:
    0.18em;

  color:
    #ffffff;

  text-shadow:
    0 0 12px
    rgba(
      255,
      255,
      255,
      0.25
    ),

    0 0 30px
    rgba(
      255,
      35,
      60,
      0.45
    );

  transition:
    opacity
    1s
    ease,

    transform
    1.2s
    cubic-bezier(
      0.16,
      1,
      0.3,
      1
    );
}

.brand-name.visible {
  opacity: 1;

  transform:
    translate(-50%, -50%)
    scale(1);
}

.text-content.visible {
  opacity: 1;
  transform:
    translateY(0)
    scale(1);
  filter: blur(0);
}

.heading-text {
  font-family:
    'Bangers',
    cursive;
  font-size:
    clamp(
      2rem,
      5.2vw,
      5rem
    );
  font-weight: 400;
  line-height: 1.08;
  letter-spacing: 0.08em;
  color: transparent;
  -webkit-text-stroke:
    1px
    rgba(
      255,
      255,
      255,
      0.9
    );
}

.heading-text.bold {
  font-weight: 600;
  -webkit-text-stroke:
    1.2px
    #ffffff;
  text-shadow:
    0 0 15px
    rgba(
      255,
      255,
      255,
      0.12
    ),
    0 0 30px
    rgba(
      255,
      35,
      60,
      0.15
    );
}

.fade-leave-active {
  transition:
    opacity
    0.9s
    cubic-bezier(
      0.16,
      1,
      0.3,
      1
    );
}

.fade-leave-to {
  opacity: 0;
}

@media (max-width: 1024px) {
  .heading-text {
    font-size:
      clamp(
        2rem,
        7vw,
        4rem
      );
  }
}

@media (max-width: 768px) {
  .web-rotation {
    inset: -20%;
    width: 140%;
    height: 140%;
  }

  .grid-background {
    background-size:
      55px
      55px;
  }

  .heading-text {
    font-size:
      clamp(
        1.7rem,
        8vw,
        3rem
      );
    letter-spacing:
      0.04em;
  }

  .eyes-image {
    width:
      clamp(
        150px,
        42vw,
        280px
      );
  }

  .hud-status {
    right: 16px;
    bottom: 16px;
    font-size: 0.54rem;
    letter-spacing: 0.1em;
  }
}

@media (max-width: 480px) {
  .heading-text {
    font-size: 1.65rem;
  }

  .eyes-image {
    width: 170px;
  }

  .hud-status {
    max-width: 90vw;
  }
}

</style> 
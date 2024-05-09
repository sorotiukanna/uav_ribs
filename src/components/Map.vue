<script setup lang="ts">
import flight_data from '@/assets/flight_data.json'
import {computed, onMounted, ref} from "vue";
import {useDraggable} from "@vueuse/core";
const f_data = flight_data

const directions = ref([])
const speeds = ref([])
const timestamps = ref([])

const grabbing = ref(false)
const isAnimated = ref(false)
const counter = ref(0)
const wrapper = computed(() => document.getElementById('wrapper'))
const plane = computed(() => document.getElementById('airplane'))
const map = computed(() => document.getElementById('map'))
const top = computed(() => (map.value.offsetHeight / 2) - (wrapper.value.offsetHeight / 2))
const left = computed(() => (map.value.offsetWidth / 2) - (wrapper.value.offsetWidth / 2))
const map_top = computed(() => window.innerHeight / 2)
const map_left = computed(() => (window.innerWidth / 2) - (map.value.offsetWidth / 2))
const temp_top = ref(0)
const temp_left = ref(0)

const intervalId = ref(0)

const el = ref<HTMLElement | null>(null)
const {x, y, style} = useDraggable(el, {
  initialValue: {x: 0, y: 0},
})
onMounted(() => {
  directions.value = f_data.map((f) => f.direction)
  speeds.value = f_data.map((f) => f.speed)
  timestamps.value = f_data.map((f) => new Date(f.timestamp))
  toCenter()
})

function toCenter() {
  isAnimated.value = false
  counter.value = 0
  x.value = map_left.value
  y.value = map_top.value
  temp_top.value = top.value
  temp_left.value = left.value
  wrapper.value.style.top = top.value + 'px'
  wrapper.value.style.left = left.value + 'px'
  plane.value.setAttributeNS(null, "transform", "rotate(" + 0 + ")")
  clearInterval(intervalId.value)
}

function moveElement() {
  if (counter.value <= Object.keys(f_data).length - 1) {
    const rotate_angle = directions.value[counter.value]
    const calcAngle = directions.value[counter.value] * Math.PI / 180
    const setScale = speeds.value[counter.value] / 10 // setting the scale
    const getTimeDifference = (Math.abs(timestamps.value[counter.value] - timestamps.value[counter.value + 1]) / 60)
    const shiftDistance = setScale * getTimeDifference
    const deltaX = Math.cos(calcAngle) * shiftDistance
    const deltaY = Math.sin(calcAngle) * shiftDistance
    const shiftTop = ref(0)
    const shiftLeft = ref(0)
    switch (true) {
      case calcAngle >= 0 && calcAngle <= 90:
        shiftTop.value = temp_top.value - deltaX
        shiftLeft.value = temp_left.value + deltaY
        break;
      case calcAngle >= 90 && calcAngle <= 180:
        shiftTop.value = temp_top.value + deltaX
        shiftLeft.value = temp_left.value + deltaY
        break;
      case calcAngle >= 180 && calcAngle <= 270:
        shiftTop.value = temp_top.value + deltaX
        shiftLeft.value = temp_left.value - deltaY
        break;
      case calcAngle >= 270 && calcAngle <= 360:
        shiftTop.value = temp_top.value - deltaX
        shiftLeft.value = temp_left.value - deltaY
        break;
      default:
        shiftTop.value = temp_top.value
        shiftLeft.value = temp_left.value
    }
    plane.value.setAttributeNS(null, "transform", "rotate(" + rotate_angle + ")");
    setTimeout(() => {
      wrapper.value.style.top = shiftTop.value + 'px'
      wrapper.value.style.left = shiftLeft.value + 'px'
      temp_top.value = shiftTop.value
      temp_left.value = shiftLeft.value
      counter.value++
    }, 50)
  }
}

function startAnimation() {
  isAnimated.value = true
  intervalId.value = setInterval(moveElement, 200)
}
</script>

<template>
  <div ref="el" :style="style" class="position-fixed draggable" :class="grabbing ? 'cursor-grabbing' : 'cursor-grab' "
       @mousedown="grabbing = true" @mouseup="grabbing = false">
    <v-img src="@/assets/bg.jpg" width="5000" height="3500" id="map" class="map">
      <div id="wrapper" class="position-absolute wrapper">
        <svg xmlns="http://www.w3.org/2000/svg" width="3em" height="3em" viewBox="0 0 64 64" id="airplane" class="position-relative pa-2 airplane">
          <g>
            <path fill="#3b5156"
                  d="M33.627 21.527c0 1.321-.729 2.398-1.624 2.398c-.9 0-1.626-1.077-1.626-2.398l.793-19.13c0-1.327-.067-2.4.833-2.4c.894 0 .831 1.073.831 2.4z"/>
            <path fill="#243438"
                  d="M47.31 40.534c0 .899 1.969 1.624 4.395 1.624c2.424 0 4.393-.725 4.393-1.624l-2.145-12.941c0-.896.176-1.626-2.248-1.626c-2.426 0-2.246.73-2.246 1.626z"/>
            <path fill="#3b5156"
                  d="M43.522 37.892c0 .892.612 1.62 1.375 1.62c.754 0 1.366-.729 1.366-1.62l-.672-12.949c0-.894.06-1.623-.694-1.623c-.763 0-.707.729-.707 1.623zm-25.72 0c0 .896.612 1.62 1.369 1.62c.754 0 1.372-.725 1.372-1.62l-.667-12.949c0-.894.049-1.623-.705-1.623c-.756 0-.701.729-.701 1.623z"/>
            <path fill="#49a59c"
                  d="M38.29 24.47c0 1.286-2.817 2.325-6.287 2.325c-3.469 0-6.287-1.04-6.287-2.325l3.07-18.526c0-1.282-.252-2.323 3.217-2.323c3.469 0 3.217 1.042 3.217 2.323z"/>
            <path fill="#3b5156"
                  d="M29.389 44.558c0-1.326 1.186-2.399 2.646-2.399c1.457 0 2.643 1.073 2.643 2.399l-1.29 16.867c0 1.327.104 2.399-1.353 2.399c-1.461 0-1.357-1.072-1.357-2.399z"/>
            <path fill="#243438"
                  d="M16.698 40.534c0 .896-1.969 1.624-4.392 1.624c-2.424 0-4.393-.729-4.393-1.624l2.142-12.945c0-.896-.173-1.622 2.25-1.622s2.248.726 2.248 1.622z"/>
            <path fill="#49a59c"
                  d="M38.532 51.17c0 1.503-.67 2.72-4.735 2.72H30.21c-4.068 0-4.731-1.217-4.731-2.72l-2.626-24.529c0-1.502 3.29-2.721 7.357-2.721h3.587c4.065 0 7.365 1.219 7.365 2.721z"/>
            <path fill="#3fa094"
                  d="m31.57 51.17l-2.625-24.529c0-1.345 2.644-2.467 6.113-2.68a17.842 17.842 0 0 0-1.24-.042h-3.591c-4.063 0-7.359 1.219-7.359 2.721l2.628 24.529c0 1.503.667 2.72 4.731 2.72h3.591c.457 0 .867-.017 1.24-.046c-2.958-.232-3.488-1.338-3.488-2.673"/>
            <path fill="#4bb2a5"
                  d="M60.705 42.587a102798.51 102798.51 0 0 1-26.691-22.952c-.879-.746-3.13-.746-4 0C21.12 27.29 12.219 34.933 3.328 42.587c-.484.424-.384.823.006 1.139c.341.332.999.582 1.987.582c8.903 0 17.808-5.895 26.712-5.895c8.885 0 17.779 5.895 26.667 5.895c1.773 0 2.873-.975 2.01-1.721"/>
            <path fill="#3fa094"
                  d="M11.25 42.587c8.265-7.109 16.53-14.21 24.791-21.316c-.631-.547-1.266-1.087-1.896-1.632c-.878-.75-3.134-.75-4 0c-8.893 7.654-17.794 15.294-26.687 22.948c-.484.43-.378.825.006 1.141c.345.332.997.58 1.989.58c1.896 0 3.798-.266 5.696-.688c-.285-.295-.326-.653.103-1.033m28.701-4.177c-1.299 0-2.597.132-3.897.347c7.591 1.254 15.183 5.548 22.774 5.548c.98 0 1.751-.295 2.104-.688c-6.996-1.557-13.984-5.207-20.981-5.207"/>
            <path fill="#4bb2a5"
                  d="M46.11 62.27c-4.363-3.753-8.727-7.501-13.09-11.26c-.426-.366-1.534-.366-1.961 0c-4.363 3.759-8.726 7.507-13.09 11.26c-.236.209-.187.405.004.562c.167.16.49.286.977.286c4.369 0 8.731-2.896 13.1-2.896c4.36 0 8.72 2.896 13.09 2.896c.867.001 1.41-.479.977-.848"/>
            <path fill="#fff"
                  d="M35.04 26.752c0 1.126-1.35 2.036-3.02 2.036c-1.666 0-3.01-.91-3.01-2.036v-6.03c0-1.124 1.349-2.034 3.01-2.034c1.668 0 3.02.91 3.02 2.034z"/>
            <path fill="#e6e7e8"
                  d="M31.08 26.896v-6.03c0-.941.953-1.727 2.236-1.961a4.18 4.18 0 0 0-1.342-.218c-1.662 0-3.01.91-3.01 2.034v6.03c0 1.126 1.351 2.036 3.01 2.036c.281 0 .54-.034.79-.077c-.991-.335-1.684-1.018-1.684-1.815"/>
            <path fill="#26a9e0"
                  d="M32.05 18.465c-1.764 0-3.193.964-3.193 2.155v1.854c0-1.193 1.429-2.161 3.193-2.161c1.766 0 3.197.968 3.197 2.161V20.62c0-1.191-1.432-2.155-3.197-2.155"/>
          </g>
        </svg>
      </div>
    </v-img>
  </div>
  <div class="position-fixed control">
    <v-btn v-if="isAnimated" color="blue-grey-darken-1" width="150" rounded="pill" @click="toCenter()">Stop</v-btn>
    <v-btn v-else color="yellow-lighten-1" width="150" rounded="pill" @click="startAnimation()">Start</v-btn>
  </div>
  <div class="coordinates">
    <div class="position-fixed font-weight-black coordinates__top">
      {{ 'N 360 &deg;' }}
    </div>
    <div class="position-fixed font-weight-black coordinates__bottom">
      {{ 'S 180 &deg;' }}
    </div>
    <div class="position-fixed font-weight-black coordinates__left">
      {{ 'W 270 &deg;' }}
    </div>
    <div class="position-fixed font-weight-black coordinates__right">
      {{ 'E 90 &deg;' }}
    </div>
  </div>
</template>

<style scoped lang="scss">
.map {
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  .wrapper {
    transition: all 0.2s linear;
    .airplane {
      background: rgba(52, 52, 52, 0.7);
      border: 1px solid rgb(229, 219, 84);
      border-radius: 16px;
      box-shadow: 0 4px 30px rgba(0, 0, 0, 0.1);
      backdrop-filter: blur(5.3px);
      -webkit-backdrop-filter: blur(5.3px);
      transition: all 0.2s linear;
    }
  }
}

.control {
  bottom: 150px;
  left: 50%;
  transform: translate(-50%, 0);
}

.coordinates {
  &__top {
    top: 40px;
    left: 50%;
    transform: translate(-50%, 0);
  }

  &__bottom {
    bottom: 40px;
    left: 50%;
    transform: translate(-50%, 0);
  }

  &__left {
    top: 50%;
    left: 40px;
    transform: translate(0, -50%) rotate(180deg);
    writing-mode: vertical-rl;
  }

  &__right {
    top: 50%;
    right: 40px;
    transform: translate(0, -50%);
    writing-mode: vertical-rl;
  }
}
</style>

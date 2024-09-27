<script setup lang="ts">
import { ref, reactive, watch } from 'vue';
import { useMouse, useMousePressed } from '@vueuse/core';

const count = ref(0);
const turnPoint = reactive({ x: 0, y: 0 });
const boundary = reactive({
  left: 0,
  right: 0,
  top: 0,
  bottom: 0,
})

const direction = ref(1 as 1 | -1); // 1: down, -1: up
const mouse = useMouse();
const pressed = useMousePressed().pressed;

const BOUNCE_Y_THRESHOLD = 3;
const DY_THRESHOLD = 6;
const DX_THRESHOLD = -5;

function update_boundary() {
  boundary.left = Math.min(boundary.left, mouse.x.value);
  boundary.right = Math.max(boundary.right, mouse.x.value);
  boundary.top = Math.min(boundary.top, mouse.y.value);
  boundary.bottom = Math.max(boundary.bottom, mouse.y.value);
}

function set_turn_point() {
  turnPoint.x = mouse.x.value;
  turnPoint.y = mouse.y.value;
  boundary.left = mouse.x.value;
  boundary.right = mouse.x.value;
  boundary.top = mouse.y.value;
  boundary.bottom = mouse.y.value;
}

watch(pressed, newVal => {
  if (newVal) {
    direction.value = 1;
    count.value += 1;
    set_turn_point();
  } else {
    if (Math.abs(mouse.y.value - turnPoint.y) > DY_THRESHOLD && mouse.x.value - turnPoint.x < DX_THRESHOLD) {
      count.value -= 2;
    }
  }
});

watch([mouse.x, mouse.y], ([x, y]) => {
  if (!pressed.value) {
    return;
  }
  update_boundary();
  const dy = y - turnPoint.y;
  const dx = x - turnPoint.x;

  const bounceY = Math.max((boundary.bottom - y) * direction.value, (boundary.top - y) * direction.value)
  if (direction.value * dy > 0 && Math.abs(dy) >= DY_THRESHOLD && bounceY > BOUNCE_Y_THRESHOLD) {
    console.log(direction.value, dx, dy, bounceY, boundary)
    count.value += dx < DX_THRESHOLD ? -1 : 1;
    direction.value *= -1;
    set_turn_point();
  }
})

function reset() {
  count.value = 0;
}
</script>

<template>
  <div class="root">
    <div class="container">
      {{ count }}
    </div>
    <div class="legend"></div>
    <div class="button-container"><button @click="reset">Reset</button></div>
  </div>
</template>

<style scoped>
.root {
  display: flex;
  flex-direction: column;
  height: 100dvh;
  justify-content: center;
  gap: 1rem;
  padding-left: 2rem;
  padding-right: 2rem;
}

.container {
  text-align: center;
  align-items: center;
  border-collapse: collapse;
  user-select: none;
  font-size: 6rem;
}

.container span {
  width: 4rem;
  border: 1px solid black;
  text-align: center;
}

.button-container {
  text-align: center;
  font-size: 2rem;
  user-select: none;
}

.legend {
  height: 32px;
  border-top: 2px dashed black;
  border-bottom: 2px dashed black;
}

svg {
  width: 100%;
}
</style>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';

const petContainer = ref<HTMLElement | null>(null);
const petX = ref(50); // Initial X position
const petY = ref(50); // Initial Y position
const dragging = ref(false);
const offsetX = ref(0);
const offsetY = ref(0);

let autoMoveInterval: NodeJS.Timeout | null = null;
const windowWidth = 150; // Match BrowserWindow width
const windowHeight = 150; // Match BrowserWindow height
const petWidth = 100; // SVG width
const petHeight = 100; // SVG height

const startAutoMove = () => {
  if (autoMoveInterval) clearInterval(autoMoveInterval);

  autoMoveInterval = setInterval(() => {
    if (dragging.value) return;

    const moveX = (Math.random() - 0.5) * 10;
    const moveY = (Math.random() - 0.5) * 10;

    let nextX = petX.value + moveX;
    let nextY = petY.value + moveY;

    nextX = Math.max(0, Math.min(nextX, windowWidth - petWidth));
    nextY = Math.max(0, Math.min(nextY, windowHeight - petHeight));

    petX.value = nextX;
    petY.value = nextY;
  }, 1000);
};

const stopAutoMove = () => {
  if (autoMoveInterval) {
    clearInterval(autoMoveInterval);
    autoMoveInterval = null;
  }
};

const handleMouseDown = (event: MouseEvent) => {
  if (!petContainer.value) return;
  stopAutoMove();
  dragging.value = true;
  const rect = petContainer.value.getBoundingClientRect();
  offsetX.value = event.clientX - rect.left;
  offsetY.value = event.clientY - rect.top;
  petContainer.value.style.cursor = 'grabbing';
  window.addEventListener('mousemove', handleMouseMove);
  window.addEventListener('mouseup', handleMouseUp);
};

const handleMouseMove = (event: MouseEvent) => {
  if (!dragging.value) return;
  let newX = event.clientX - offsetX.value;
  let newY = event.clientY - offsetY.value;
  petX.value = newX;
  petY.value = newY;
};

const handleMouseUp = () => {
  if (!dragging.value) return;
  dragging.value = false;
  if (petContainer.value) {
    petContainer.value.style.cursor = 'grab';
    let currentX = petX.value;
    let currentY = petY.value;
    currentX = Math.max(0, Math.min(currentX, windowWidth - petWidth));
    currentY = Math.max(0, Math.min(currentY, windowHeight - petHeight));
    petX.value = currentX;
    petY.value = currentY;
  }
  window.removeEventListener('mousemove', handleMouseMove);
  window.removeEventListener('mouseup', handleMouseUp);
  startAutoMove();
};

onMounted(() => {
  startAutoMove();
});

onUnmounted(() => {
  stopAutoMove();
  window.removeEventListener('mousemove', handleMouseMove);
  window.removeEventListener('mouseup', handleMouseUp);
});

</script>

<template>
  <div id="pet-container" ref="petContainer" :style="{ left: `${petX}px`, top: `${petY}px` }"
    @mousedown="handleMouseDown">
    <svg width="100" height="100" viewBox="0 0 100 100" class="desktop-pet">
      <circle cx="50" cy="50" r="45" fill="yellow" stroke="black" stroke-width="2" />
      <circle cx="35" cy="40" r="5" fill="black" />
      <circle cx="65" cy="40" r="5" fill="black" />
      <path d="M 30 65 Q 50 80 70 65" stroke="black" stroke-width="3" fill="transparent" />
    </svg>
  </div>
</template>

<style scoped>
#pet-container {
  position: absolute;
  cursor: grab;
  user-select: none;
  /* Prevent text selection during drag */
}

.desktop-pet {
  display: block;
  /* Prevents potential small space below SVG */
}
</style>

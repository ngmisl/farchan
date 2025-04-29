<script setup lang="ts">
defineProps<{
  mood: string;
}>();
</script>

<template>
  <div class="character">
    <img 
      :src="`/farchan-${mood}-transparent.png`" 
      :alt="`Far-chan (${mood})`" 
      :class="`mood-${mood}`"
    />
  </div>
</template>

<style scoped>
.character {
  position: absolute;
  bottom: 25%;
  left: 50%;
  transform: translateX(-50%);
  width: 55%;
  height: 65%;
  display: flex;
  justify-content: center;
}

.character img {
  height: 100%;
  object-fit: contain;
  transition: transform 0.3s ease;
}

.mood-angry {
  transform: scale(1.05);
}

.mood-shy {
  transform: scale(0.95) translateY(5px);
}

.mood-inlove {
  animation: heartbeat 1.5s infinite;
}

@keyframes heartbeat {
  0% { transform: scale(1); }
  15% { transform: scale(1.03); }
  30% { transform: scale(1); }
  45% { transform: scale(1.03); }
  60% { transform: scale(1); }
}

@media (max-width: 768px) {
  .character {
    bottom: 32%;
    width: 90%;
  }
}

@media (min-width: 1200px) {
  .character {
    width: 45%;
  }
}

@media (min-width: 1600px) {
  .character {
    width: 40%;
  }
}
</style>

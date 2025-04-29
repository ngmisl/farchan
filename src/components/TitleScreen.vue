<script setup lang="ts">
defineProps({
  title: {
    type: String,
    required: true
  },
  farcasterConnected: {
    type: Boolean,
    default: false
  },
  playerName: {
    type: String,
    default: 'You'
  }
});

defineEmits(['startGame', 'connectFarcaster']);
</script>

<template>
  <div class="title-screen" :style="{ backgroundImage: 'url(/farchan-bg.png)' }">
    <div class="title-content">
      <h1 class="game-title">{{ title }}</h1>
      
      <div class="title-character">
        <img src="/farchan-neutral-transparent.png" alt="Far-chan" />
      </div>
      
      <div class="title-buttons">
        <button @click="$emit('startGame')" class="start-button">Start Game</button>
        <button 
          v-if="!farcasterConnected" 
          @click="$emit('connectFarcaster')" 
          class="farcaster-button"
        >
          Connect Farcaster
        </button>
        <div v-else class="farcaster-connected">
          Connected as: {{ playerName }}
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.title-screen {
  position: relative;
  width: 100%;
  height: 100%;
  min-height: 100%;
  background-size: cover;
  background-position: center;
  overflow: hidden;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  display: flex;
  justify-content: center;
  align-items: center;
}

.title-content {
  width: 90%;
  max-width: 800px;
  background-color: rgba(255, 255, 255, 0.9);
  border-radius: 16px;
  padding: 2rem;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.2);
  text-align: center;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  max-height: 90%;
  overflow: visible;
}

.game-title {
  font-size: clamp(1rem, 2.5vw, 1.7rem);
  font-weight: bold;
  margin-bottom: 1.5rem;
  color: #ff6bae;
  line-height: 1.4;
  text-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.title-character {
  width: 60%;
  max-width: 300px;
  height: auto;
  margin: 0 auto;
  display: flex;
  justify-content: center;
}

.title-character img {
  width: 100%;
  object-fit: contain;
  filter: drop-shadow(0 4px 8px rgba(0, 0, 0, 0.2));
}

.title-buttons {
  margin-top: 2rem;
  display: flex;
  flex-direction: column;
  gap: 1rem;
  max-width: 300px;
  margin-left: auto;
  margin-right: auto;
}

.start-button {
  padding: 1rem;
  background-color: #ff8dc7;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: bold;
  font-size: 1.1rem;
  transition: all 0.2s;
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.2);
}

.start-button:hover {
  background-color: #ff6bae;
  transform: translateY(-2px);
  box-shadow: 0 5px 10px rgba(0, 0, 0, 0.25);
}

.farcaster-button {
  padding: 1rem;
  background-color: #6E91F6;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 1.1rem;
  transition: all 0.2s;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.farcaster-button:hover {
  background-color: #5D7FDE;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
}

.farcaster-connected {
  font-size: 1.1rem;
  color: #666;
  background-color: #f0f0f0;
  padding: 0.75rem;
  border-radius: 8px;
  text-align: center;
}

@media (max-width: 768px) {
  .title-content {
    width: 95%;
    padding: 1.5rem;
  }
  
  .game-title {
    font-size: clamp(0.9rem, 4vw, 1.4rem);
  }
}
</style>

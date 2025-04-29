<script setup lang="ts">
import CharacterDisplay from './CharacterDisplay.vue';
import DialogueBox from './DialogueBox.vue';

defineProps({
  // Character state
  farChanMood: String,
  
  // Dialogue display
  currentDialogue: Object,
  dialogOptions: Array,
  isFinalScene: Boolean,
  finalEnding: String,
  
  // Farcaster and game state
  farcasterConnected: Boolean,
  turnCount: Number,
  gameOutcome: [String, null]
});

defineEmits(['selectOption', 'resetGame', 'shareToFarcaster']);
</script>

<template>
  <div class="game-screen" :style="{ backgroundImage: 'url(/farchan-bg.png)' }">
    <CharacterDisplay :mood="farChanMood" />
    
    <DialogueBox 
      :currentDialogue="currentDialogue"
      :options="dialogOptions"
      :isFinalScene="isFinalScene"
      :finalEnding="finalEnding"
      :farcasterConnected="farcasterConnected"
      :turnCount="turnCount"
      :gameOutcome="gameOutcome"
      @selectOption="$emit('selectOption', $event)"
      @resetGame="$emit('resetGame')"
      @shareToFarcaster="$emit('shareToFarcaster')"
    />
  </div>
</template>

<style scoped>
.game-screen {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  background-size: cover;
  background-position: center;
  overflow: hidden;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

@media (max-width: 768px) {
  .game-screen {
    height: 80vh;
    border-radius: 0;
  }
}
</style>

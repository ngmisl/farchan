<script setup lang="ts">
import { ref } from 'vue';

// Define interface for option objects
export interface DialogOption {
  text: string;
  outcome?: string;
}

defineProps({
  currentDialogue: {
    type: Object,
    default: () => ({ speaker: '', text: '' })
  },
  options: {
    type: Array as () => DialogOption[],
    default: () => []
  },
  isFinalScene: {
    type: Boolean,
    default: false
  },
  finalEnding: {
    type: String,
    default: ''
  },
  farcasterConnected: {
    type: Boolean,
    default: false
  },
  turnCount: {
    type: Number,
    default: 0
  },
  gameOutcome: {
    type: String,
    default: ''
  }
});

const emit = defineEmits(['selectOption', 'resetGame', 'shareToFarcaster']);

// For handling choice popup
const showChoicePopup = ref(false);

function toggleChoicePopup() {
  showChoicePopup.value = !showChoicePopup.value;
}

function handleOptionClick(index: number) {
  emit('selectOption', index);
  showChoicePopup.value = false;
}
</script>

<template>
  <div class="dialogue-box">
    <!-- Single Dialogue Entry (only shown when not in final scene) -->
    <div class="single-dialogue-display" v-if="!isFinalScene">
      <div v-if="currentDialogue" 
           class="dialogue-entry"
           :class="{ 
             'narrator': currentDialogue.speaker === 'Narrator', 
             'player': currentDialogue.speaker !== 'Narrator' && currentDialogue.speaker !== 'Far-chan', 
             'Far-chan': currentDialogue.speaker === 'Far-chan' 
           }">
        <strong>{{ currentDialogue.speaker }}:</strong> {{ currentDialogue.text }}
      </div>
    </div>
    
    <!-- Final Scene -->
    <div v-if="isFinalScene" class="ending">
      <p class="ending-text">{{ finalEnding }}</p>
      <div class="ending-buttons">
        <button @click="$emit('resetGame')" class="reset-button">Play Again</button>
        
        <!-- Share to Farcaster button (only appears for good ending and when connected) -->
        <button 
          v-if="gameOutcome === 'good_end' && farcasterConnected" 
          @click="$emit('shareToFarcaster')" 
          class="share-button"
        >
          Share to Farcaster
        </button>
      </div>
    </div>
    
    <!-- Choice Button for Non-Final Scenes -->
    <div v-else class="choice-button-container">
      <button v-if="options && options.length > 0" @click="toggleChoicePopup" class="choice-button">
        Choose Your Response
      </button>
    </div>
    
    <!-- Choice Popup -->
    <div v-if="showChoicePopup" class="choice-popup-overlay" @click="toggleChoicePopup">
      <div class="choice-popup" @click.stop>
        <div class="choice-popup-header">
          <h2>Choose Your Response</h2>
          <button @click="toggleChoicePopup" class="close-button">×</button>
        </div>
        <div class="choice-options">
          <button 
            v-for="(option, index) in options" 
            :key="index"
            @click="handleOptionClick(index)"
            class="option-button"
          >
            {{ option.text }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.dialogue-box {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  min-height: 25%;
  background-color: rgba(255, 255, 255, 0.92);
  padding: 1.25rem 1.25rem 1rem;
  border-radius: 12px 12px 0 0;
  box-shadow: 0 -3px 10px rgba(0, 0, 0, 0.15);
  display: flex;
  flex-direction: column;
  margin-bottom: 0;
}

.single-dialogue-display {
  min-height: 60px;
  max-height: 110px;
  margin-bottom: 1rem;
  font-size: 1rem;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.dialogue-entry {
  margin-bottom: 0.6rem;
  line-height: 1.5;
  padding: 0.35rem 0;
  text-shadow: 0 0 1px rgba(255, 255, 255, 0.7);
}

.dialogue-entry.narrator {
  font-style: italic;
  color: #555;
  background-color: rgba(240, 240, 240, 0.7);
  padding: 0.5rem;
  border-radius: 6px;
  margin-bottom: 0.8rem;
}

.dialogue-entry.Far-chan {
  color: #cc3366;
  font-weight: 500;
}

.dialogue-entry.player {
  color: #3355aa;
  font-weight: 500;
}

.dialogue-entry strong {
  font-weight: 700;
  margin-right: 0.4rem;
}

.ending {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.2rem;
  padding: 0.5rem;
}

.ending-text {
  font-style: italic;
  text-align: center;
  color: #ff4d94;
  font-size: 1.1rem;
  line-height: 1.6;
  text-shadow: 0 0 1px rgba(255, 255, 255, 0.9);
}

.reset-button {
  padding: 0.85rem 1.75rem;
  background-color: #ff8dc7;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: bold;
  transition: all 0.2s;
  font-size: 1.05rem;
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.2);
}

.reset-button:hover {
  background-color: #ff6bae;
  transform: translateY(-2px);
  box-shadow: 0 5px 10px rgba(0, 0, 0, 0.25);
}

.share-button {
  padding: 0.85rem 1.75rem;
  background-color: #ff8dc7;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: bold;
  transition: all 0.2s;
  font-size: 1.05rem;
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.2);
}

.share-button:hover {
  background-color: #ff6bae;
  transform: translateY(-2px);
  box-shadow: 0 5px 10px rgba(0, 0, 0, 0.25);
}

.ending-buttons {
  display: flex;
  gap: 1rem;
  justify-content: center;
}

.choice-button-container {
  display: flex;
  justify-content: center;
  margin-top: 0.25rem;
  margin-bottom: 0;
}

.choice-button {
  padding: 0.8rem 1.5rem;
  background-color: #ff8dc7;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: bold;
  transition: all 0.2s;
  font-size: 1rem;
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.2);
}

.choice-button:hover {
  background-color: #ff6bae;
  transform: translateY(-2px);
  box-shadow: 0 5px 10px rgba(0, 0, 0, 0.25);
}

.choice-popup-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.choice-popup {
  width: 90%;
  max-width: 500px;
  background-color: white;
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
  padding: 1.5rem;
  max-height: 80vh;
  overflow-y: auto;
  position: relative;
}

.choice-popup-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  border-bottom: 1px solid #eee;
  padding-bottom: 0.5rem;
}

.choice-popup-header h2 {
  font-size: 1.3rem;
  color: #ff6bae;
  margin: 0;
}

.close-button {
  background: none;
  border: none;
  font-size: 1.5rem;
  cursor: pointer;
  color: #999;
  transition: color 0.2s;
}

.close-button:hover {
  color: #333;
}

.choice-options {
  display: flex;
  flex-direction: column;
  gap: 0.8rem;
}

.option-button {
  padding: 1rem;
  background-color: #f5f5f5;
  border: 2px solid #e0e0e0;
  border-radius: 8px;
  cursor: pointer;
  text-align: left;
  font-size: 1rem;
  transition: all 0.2s;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  font-weight: 500;
  color: #333;
}

.option-button:hover {
  background-color: #ffe6f2;
  border-color: #ffb6e1;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
}

@media (max-width: 768px) {
  .dialogue-box {
    padding: 1rem;
    min-height: 35%;
  }
  
  .single-dialogue-display {
    max-height: 120px;
    font-size: 0.95rem;
  }
  
  .choice-popup {
    width: 95%;
    padding: 1rem;
  }
  
  .option-button {
    padding: 0.8rem;
    font-size: 0.95rem;
  }
}

@media (min-width: 1200px) {
  .single-dialogue-display {
    max-height: 150px;
    font-size: 1.1rem;
  }
  
  .dialogue-box {
    padding: 1.5rem;
  }
}

@media (min-width: 1600px) {
  .single-dialogue-display {
    max-height: 180px;
  }
}
</style>

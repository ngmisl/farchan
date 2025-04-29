<script setup lang="ts">
import { ref, onMounted, computed } from 'vue'
import sdk from '@farcaster/frame-sdk'
import { z } from 'zod'

// Components
import TitleScreen from './components/TitleScreen.vue'
import GameScreen from './components/GameScreen.vue'

// Define type for the Farcaster SDK context
type FrameContext = Record<string, unknown>

// Game state and types
interface GameState {
  phase: number
  farChanMood: string
  dialogHistory: Array<{ speaker: string; text: string; mood?: string }>
  selectedOption: number | null
  showTitleScreen: boolean
  playerName: string
  farcasterConnected: boolean
  lastOutcome: string | null
  turnCount: number
}

// Validation schema for saving game state
const gameStateSchema = z.object({
  phase: z.number().int().min(0),
  farChanMood: z.string(),
  dialogHistory: z.array(z.object({
    speaker: z.string(),
    text: z.string(),
    mood: z.string().optional()
  })),
  selectedOption: z.number().nullable(),
  showTitleScreen: z.boolean(),
  playerName: z.string(),
  farcasterConnected: z.boolean(),
  lastOutcome: z.string().nullable(),
  turnCount: z.number().int().min(0)
})

// SDK state
const isSDKLoaded = ref(false)
const context = ref<FrameContext | undefined>()

// Game state
const gameState = ref<GameState>({
  phase: 0,
  farChanMood: 'neutral',
  dialogHistory: [],
  selectedOption: null,
  showTitleScreen: true,
  playerName: 'You',
  farcasterConnected: false,
  lastOutcome: null,
  turnCount: 0
})

// Game scenes and dialogue content - simplified to a single scene
const scenes = [
  // Main Scene
  {
    id: 0,
    location: 'classroom',
    intro: "I can't believe I accidentally confessed to Far-chan yesterday! Now I have to face her at school...",
    text: "You spot Far-chan reading alone by the window. The morning sunlight catches in her hair as she looks up, noticing you. Her expression is unreadable as she closes her book.",
    speaker: "Narrator",
    options: [
      { 
        text: "Hey, about yesterday... I meant what I said. You're really special to me.", 
        outcome: "sincere"
      },
      { 
        text: "Good morning! Nice weather we're having, right? Haha...", 
        outcome: "awkward"
      },
      { 
        text: "I was wondering if you thought about what I said yesterday?", 
        outcome: "direct"
      }
    ],
    responses: {
      sincere: {
        text: "Hmph. You can't just say things like that so easily! But... I guess I don't hate hearing it.",
        mood: "shy",
        nextOptions: [
          {
            text: "Would you like to have lunch together today? Just the two of us?",
            outcome: "lunch_invite"
          },
          {
            text: "I've liked you for a long time. I was just too nervous to say anything until now.",
            outcome: "confession_deeper"
          },
          {
            text: "I understand if you need time to think about it. I'll wait.",
            outcome: "patient"
          }
        ]
      },
      awkward: {
        text: "Are you seriously talking about the weather right now? After what you said yesterday?",
        mood: "angry",
        nextOptions: [
          {
            text: "Sorry, I'm just nervous. I really do like you, Far-chan.",
            outcome: "recover"
          },
          {
            text: "Yeah, I guess that was pretty lame. But talking about feelings is hard!",
            outcome: "honest_awkward"
          },
          {
            text: "Maybe we should just forget about yesterday...",
            outcome: "retreat"
          }
        ]
      },
      direct: {
        text: "You can't just ask me that out of nowhere! I... I haven't decided how I feel yet.",
        mood: "shy",
        nextOptions: [
          {
            text: "That's okay. I just want you to know my feelings are genuine.",
            outcome: "reassure"
          },
          {
            text: "Maybe we could hang out sometime? No pressure.",
            outcome: "casual"
          },
          {
            text: "I understand. Take all the time you need.",
            outcome: "space"
          }
        ]
      },
      lunch_invite: {
        text: "I... I suppose I could do that. But don't get any weird ideas! I just happen to have made extra food today.",
        mood: "shy",
        nextOptions: [
          {
            text: "Your cooking looks amazing! Did you make this yourself?",
            outcome: "good_end"
          }
        ]
      },
      confession_deeper: {
        text: "You've liked me for that long? Why didn't you say something sooner, you dummy...",
        mood: "shy",
        nextOptions: [
          {
            text: "The sunsets are always beautiful here, but they're better when I'm with you.",
            outcome: "good_end"
          }
        ]
      },
      patient: {
        text: "You'd... wait for me? Nobody's ever been that patient with me before...",
        mood: "shy",
        nextOptions: [
          {
            text: "You're worth waiting for. I just want you to be comfortable.",
            outcome: "good_end"
          }
        ]
      },
      recover: {
        text: "Well... at least you're being honest now.",
        mood: "neutral",
        nextOptions: [
          {
            text: "I promise to always be honest with you about my feelings.",
            outcome: "good_end"
          }
        ]
      },
      honest_awkward: {
        text: "At least you're being honest now. Maybe you're not completely hopeless.",
        mood: "neutral",
        nextOptions: [
          {
            text: "Does that mean I have a chance?",
            outcome: "good_end"
          }
        ]
      },
      retreat: {
        text: "Fine. If that's what you want, then forget it ever happened.",
        mood: "angry",
        nextOptions: [
          {
            text: "Wait, Far-chan! That's not what I meant. I do care about you!",
            outcome: "bad_end"
          }
        ]
      },
      reassure: {
        text: "Genuine, huh? It's not like I care that much... but I'm listening.",
        mood: "shy",
        nextOptions: [
          {
            text: "I want to know everything about you. What makes you happy? What are your dreams?",
            outcome: "good_end"
          }
        ]
      },
      casual: {
        text: "Hmm... I guess hanging out would be fine. Just don't make it weird, okay?",
        mood: "neutral",
        nextOptions: [
          {
            text: "I'd really like that. I know a great cafe we could try.",
            outcome: "good_end"
          }
        ]
      },
      space: {
        text: "...thank you for understanding. Not many people do.",
        mood: "shy",
        nextOptions: [
          {
            text: "Of course. Your feelings matter to me.",
            outcome: "good_end"
          }
        ]
      },
      good_end: {
        text: "You know, I never thought someone would understand me the way you do. It's... nice.",
        mood: "inlove",
        ending: "Far-chan smiles at you, a genuine smile that lights up her entire face. As cherry blossoms drift down around you both, you realize that sometimes the most beautiful love stories begin with an accidental confession."
      },
      bad_end: {
        text: "It's too late for that. I think we should just be classmates. Nothing more.",
        mood: "neutral",
        ending: "You nod, accepting her decision with as much dignity as you can muster. The cherry blossoms continue to fall, indifferent to the quiet heartbreak beneath them."
      }
    }
  }
]

// Type assertion helper for responses
function assertStringIndex<T>(obj: Record<string, T>, key: string): T | undefined {
  return obj[key as keyof typeof obj];
}

// Game mechanics and helper functions
const currentScene = computed(() => {
  return scenes[0]; // Always the same scene for this simple game
})

const currentOptions = computed(() => {
  const scene = currentScene.value;
  if (gameState.value.lastOutcome === null) {
    return scene.options || [];
  }
  
  // If we have a response, get the next options based on the last outcome
  const response = gameState.value.lastOutcome ? 
    assertStringIndex(scene.responses, gameState.value.lastOutcome) : undefined;
  return response?.nextOptions || [];
})

const isFinalScene = computed(() => {
  if (gameState.value.lastOutcome === null) return false;
  
  const isEndingOutcome = gameState.value.lastOutcome === 'good_end' || 
                          gameState.value.lastOutcome === 'bad_end';
  
  return isEndingOutcome;
})

const getFinalEnding = computed(() => {
  if (!isFinalScene.value) return '';
  
  const scene = currentScene.value;
  const lastOutcome = gameState.value.lastOutcome;
  
  if (lastOutcome && lastOutcome in scene.responses) {
    const response = assertStringIndex(scene.responses, lastOutcome);
    return response?.ending || '';
  }
  
  return '';
})

// Handle starting the game
const startGame = () => {
  gameState.value.showTitleScreen = false;
  resetGame();
}

// Connect to Farcaster
const connectFarcaster = async () => {
  try {
    // If SDK isn't loaded yet, wait for it
    if (!isSDKLoaded.value) {
      console.log('Waiting for SDK to load...');
      return;
    }
    
    // If we have a valid context with user information
    if (context.value && 'user' in context.value) {
      const user = context.value.user as Record<string, unknown>;
      if (typeof user.username === 'string') {
        gameState.value.playerName = user.username;
        gameState.value.farcasterConnected = true;
        console.log(`Connected as: ${gameState.value.playerName}`);
      }
    } else {
      console.log('No Farcaster user found in context');
    }
  } catch (error) {
    console.error('Failed to connect to Farcaster:', error);
  }
}

// Handle selecting a dialogue option
const handleOptionSelect = (index: number) => {
  if (isFinalScene.value) return;
  
  gameState.value.selectedOption = index;
  const option = currentOptions.value[index];
  if (!option) return;
  
  console.log("Selected option:", option); // Debug
  
  // Increment turn counter
  gameState.value.turnCount++;
  
  // Add player's choice to dialogue history
  gameState.value.dialogHistory.push({ 
    speaker: gameState.value.playerName, 
    text: option.text 
  });
  
  // Store the outcome for next options and ending determination
  gameState.value.lastOutcome = option.outcome;
  
  // Get current scene response
  const scene = currentScene.value;
  if (scene.responses && option.outcome in scene.responses) {
    const responseObj = assertStringIndex(scene.responses, option.outcome);
    
    if (responseObj) {
      // Update Far-chan's mood based on the response
      if (responseObj.mood) {
        gameState.value.farChanMood = responseObj.mood;
      }
      
      // Add Far-chan's response to dialogue history
      setTimeout(() => {
        gameState.value.dialogHistory.push({ 
          speaker: 'Far-chan', 
          text: responseObj.text,
          mood: responseObj.mood 
        });
        
        // If this is an ending, add the ending text after a delay
        if (responseObj.ending) {
          setTimeout(() => {
            // Use player's name in the ending text
            let endingText = responseObj.ending;
            if (gameState.value.playerName !== 'You') {
              endingText = endingText.replace(/You/g, gameState.value.playerName);
            }
            
            gameState.value.dialogHistory.push({ 
              speaker: 'Narrator', 
              text: endingText
            });
          }, 1500);
        }
      }, 1000);
    }
  }
  
  console.log("Updated dialogue history:", gameState.value.dialogHistory); // Debug
  
  // Save game state
  saveGameState();
}

// Save game state
const saveGameState = () => {
  try {
    const validated = gameStateSchema.parse(gameState.value);
    localStorage.setItem('romcom-game-state', JSON.stringify(validated));
    console.log("Game state saved"); // Debug
  } catch (error) {
    console.error('Failed to save game state:', error);
  }
}

// Reset game state
const resetGame = () => {
  gameState.value = {
    ...gameState.value,
    phase: 0,
    farChanMood: 'neutral',
    dialogHistory: [],
    selectedOption: null,
    showTitleScreen: false,
    lastOutcome: null,
    turnCount: 0
  };
  
  // Add the initial scene text
  const scene = scenes[0];
  
  // First add intro
  if (scene.intro) {
    let introText = scene.intro;
    if (gameState.value.playerName !== 'You') {
      introText = introText.replace(/I/g, gameState.value.playerName);
    }
    
    gameState.value.dialogHistory.push({ 
      speaker: 'Narrator', 
      text: introText
    });
  }
  
  // Then add the scene text
  let sceneText = scene.text;
  if (gameState.value.playerName !== 'You') {
    sceneText = sceneText.replace(/You/g, gameState.value.playerName);
  }
  
  gameState.value.dialogHistory.push({ 
    speaker: scene.speaker ?? 'Narrator', 
    text: sceneText
  });
  
  // Save the reset state
  saveGameState();
}

// Add function to share to Farcaster
function shareToFarcaster() {
  if (!gameState.value.farcasterConnected) {
    console.error('Cannot share: Farcaster not connected');
    return;
  }
  
  // Create the share text with turn count
  const shareText = `I just beat "Even Though I'm Just a Regular High Schooler Who Accidentally Confessed My Feelings to the Cool and Mysterious Girl in Class, She Turned Out to Be a Tsundere, and Now Every Day Feels Like a Battle Between Love, Embarrassment, and Unexpected Heartwarming Moments Under the Cherry Blossoms!" in ${gameState.value.turnCount} turns!`;
  
  try {
    // Use the Farcaster SDK to create a cast
    sdk.actions.composeCast({
      text: shareText,
      embeds: ["https://farchan.orbiter.website"]
    }).then(result => {
      console.log('Shared to Farcaster successfully', result);
    }).catch(error => {
      console.error('Error posting to Farcaster:', error);
    });
  } catch (error) {
    console.error('Failed to share to Farcaster:', error);
  }
}

// Farcaster SDK initialization
onMounted(async () => {
  try {
    context.value = await sdk.context;
    isSDKLoaded.value = true;
    // Signal to Farcaster that the frame is ready
    if (sdk.actions && typeof sdk.actions.ready === 'function') {
      sdk.actions.ready();
    }
    
    // Check if there's a Farcaster user already in the context
    if (context.value && 'user' in context.value) {
      const user = context.value.user as Record<string, unknown>;
      if (typeof user.username === 'string') {
        gameState.value.playerName = user.username;
        gameState.value.farcasterConnected = true;
        console.log(`Connected as: ${gameState.value.playerName}`);
      }
    }
  } catch (error) {
    console.error('Failed to initialize Farcaster SDK:', error);
  }
  
  // Start with a clean state
  resetGame();
  gameState.value.showTitleScreen = true; // Force title screen to show initially
});

// Game title for display
const gameTitle = "Even Though I'm Just a Regular High Schooler Who Accidentally Confessed My Feelings to the Cool and Mysterious Girl in Class, She Turned Out to Be a Tsundere, and Now Every Day Feels Like a Battle Between Love, Embarrassment, and Unexpected Heartwarming Moments Under the Cherry Blossoms!";
</script>

<template>
  <div class="game-container">
    <div v-if="!isSDKLoaded" class="loading">Loading...</div>
    
    <template v-else>
      <!-- Title Screen -->
      <TitleScreen 
        v-if="gameState.showTitleScreen"
        :title="gameTitle"
        :farcasterConnected="gameState.farcasterConnected"
        :playerName="gameState.playerName"
        @startGame="startGame"
        @connectFarcaster="connectFarcaster"
      />
      
      <!-- Game Screen -->
      <GameScreen 
        v-else
        :farChanMood="gameState.farChanMood"
        :currentDialogue="gameState.dialogHistory[gameState.dialogHistory.length - 1]"
        :dialogOptions="currentOptions"
        :isFinalScene="isFinalScene"
        :finalEnding="getFinalEnding"
        @selectOption="handleOptionSelect"
        @resetGame="resetGame"
        @shareToFarcaster="shareToFarcaster"
        :farcasterConnected="gameState.farcasterConnected"
        :turnCount="gameState.turnCount"
        :gameOutcome="gameState.lastOutcome"
      />
    </template>
  </div>
</template>

<style>
/* Apply a reset to ensure consistent sizing */
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html, body {
  height: 100%;
  width: 100%;
  overflow-x: hidden;
}

body {
  display: flex;
  justify-content: center;
}

#app {
  width: 100%;
  display: flex;
  justify-content: center;
}

.game-container {
  width: 100%;
  max-width: 95vw;
  margin: 0 auto;
  height: 100vh;
  display: flex;
  flex-direction: column;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

.loading {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
  font-size: 1.5rem;
}

/* Responsive styles */
@media (min-width: 1200px) {
  .game-container {
    max-width: 90vw;
  }
}

@media (min-width: 1600px) {
  .game-container {
    max-width: 85vw;
  }
}

@media (max-width: 768px) {
  .game-container {
    height: 100vh;
    width: 100%;
    max-width: 100%;
  }
}
</style>

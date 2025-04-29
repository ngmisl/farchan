[![CodeQL](https://github.com/ngmisl/farchan/actions/workflows/github-code-scanning/codeql/badge.svg)](https://github.com/ngmisl/farchan/actions/workflows/github-code-scanning/codeql)

# Far-chan RomCom Game 💘

A visual novel-style romance game built as a Farcaster Frame, featuring Far-chan, a cute tsundere high school girl who you accidentally confessed your feelings to!

## 🌟 Features

- **Visual Novel Gameplay**: Experience a short, sweet romcom adventure with multiple endings
- **Character Expression System**: Watch Far-chan's mood change based on your dialogue choices
- **Turn Counting**: The game tracks how many turns it takes you to reach an ending
- **Farcaster Integration**: Connect your Farcaster account and share your good ending
- **Responsive Design**: Fully optimized for both mobile and desktop play
- **Static Architecture**: No backend required, deploy anywhere

## 🚀 Tech Stack

- **Framework**: [Vue.js](https://vuejs.org/) 
- **Build Tool**: [Vite](https://vitejs.dev/)
- **Package Manager**: [Bun](https://bun.sh/)
- **TypeScript**: For type safety
- **Zod**: For schema validation
- **Farcaster SDK**: For Farcaster Frame integration

## 📋 Prerequisites

- [Bun](https://bun.sh/) installed on your machine
- A Farcaster account (optional, only for sharing feature)

## 🔧 Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/yourusername/farchan.git
   cd farchan
   ```

2. Install dependencies:
   ```bash
   bun install
   ```

3. Start the development server:
   ```bash
   bun run dev
   ```

4. Build for production:
   ```bash
   bun run build
   ```

## 🎮 How to Play

1. Start the game by clicking "Start Game" on the title screen
2. Read through the dialogue and select your responses
3. Your choices affect Far-chan's mood and potentially lead to different endings
4. Try to get the "good ending" in as few turns as possible
5. If you connect your Farcaster account, you can share your achievement

## 🖼️ Farcaster Frame

This game is designed to run as a Farcaster Frame. When deployed, users can:

- Play directly within the Farcaster client
- Share their in-game achievements to their Farcaster feed
- Create a fun, interactive experience within the Farcaster ecosystem

## 📱 Mobile Optimization

The game is designed with a mobile-first approach:
- Responsive text sizing that adapts to screen size
- Touch-friendly UI elements
- Compact dialogue interface optimized for smaller screens
- Vertical layout that works well on mobile devices

## 🧑‍💻 Development

### Project Structure

- `src/components/` - Vue components for game parts
- `src/App.vue` - Main game logic and state management
- `public/` - Static assets including character expressions

### Game State

The game uses Zod for validation of the game state, ensuring type safety throughout the gameplay. The state includes:

- Current game phase
- Far-chan's mood
- Dialogue history
- Player choices
- Turn count

## 📄 License

[MIT](LICENSE)

## 🙏 Acknowledgements

- Character design inspired by anime visual novels
- Built with the awesome Farcaster Frames SDK

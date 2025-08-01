<template>
  <q-page class="snake-game-page">
    <div class="snake-game-container">
      <div class="game-header">
        <h1 class="game-title">🐍 Snake Game</h1>
        <div class="game-info">
          <div class="score">Score: {{ score }}</div>
          <div class="high-score">High Score: {{ highScore }}</div>
        </div>
      </div>

      <div class="game-container">
        <canvas
          ref="gameCanvas"
          :width="canvasWidth"
          :height="canvasHeight"
          class="game-canvas"
          @keydown="handleKeyPress"
          tabindex="0"
        ></canvas>
      </div>

      <!-- Mobile Controls -->
      <div class="mobile-controls">
        <div class="control-row">
          <q-btn
            class="control-btn up-btn"
            @click="handleMobileControl('up')"
            @touchstart="handleMobileControl('up')"
            @touchend="preventDefault"
            @mousedown="handleMobileControl('up')"
            @mouseup="preventDefault"
            round
            color="primary"
            size="lg"
            icon="keyboard_arrow_up"
          />
        </div>
        <div class="control-row">
          <q-btn
            class="control-btn left-btn"
            @click="handleMobileControl('left')"
            @touchstart="handleMobileControl('left')"
            @touchend="preventDefault"
            @mousedown="handleMobileControl('left')"
            @mouseup="preventDefault"
            round
            color="primary"
            size="lg"
            icon="keyboard_arrow_left"
          />
          <q-btn
            class="control-btn down-btn"
            @click="handleMobileControl('down')"
            @touchstart="handleMobileControl('down')"
            @touchend="preventDefault"
            @mousedown="handleMobileControl('down')"
            @mouseup="preventDefault"
            round
            color="primary"
            size="lg"
            icon="keyboard_arrow_down"
          />
          <q-btn
            class="control-btn right-btn"
            @click="handleMobileControl('right')"
            @touchstart="handleMobileControl('right')"
            @touchend="preventDefault"
            @mousedown="handleMobileControl('right')"
            @mouseup="preventDefault"
            round
            color="primary"
            size="lg"
            icon="keyboard_arrow_right"
          />
        </div>
      </div>

      <div class="game-controls">
        <q-btn
          v-if="!gameStarted"
          color="primary"
          size="lg"
          @click="startGame"
          class="start-btn"
        >
          Start Game
        </q-btn>

        <q-btn
          v-if="gameOver"
          color="positive"
          size="lg"
          @click="restartGame"
          class="restart-btn"
        >
          Play Again
        </q-btn>

        <div class="instructions">
          <p>Use arrow keys, WASD, or touch controls to control the snake</p>
          <p>Eat the food to grow and increase your score!</p>
        </div>
      </div>

      <div v-if="gameOver" class="game-over-overlay">
        <div class="game-over-modal">
          <h2>Game Over!</h2>
          <p>Final Score: {{ score }}</p>
          <p v-if="newHighScore" class="new-high-score">🎉 New High Score! 🎉</p>
          <div class="modal-buttons">
            <q-btn
              color="primary"
              size="md"
              @click="restartGame"
              class="restart-modal-btn"
            >
              Play Again
            </q-btn>
            <q-btn
              color="grey"
              size="md"
              @click="closeGameOver"
              class="close-modal-btn"
            >
              Close
            </q-btn>
          </div>
        </div>
      </div>
    </div>
  </q-page>
</template>

<script setup>
import { ref, onMounted, onUnmounted, nextTick } from 'vue'
import { useQuasar } from 'quasar'

const $q = useQuasar()

// Game state
const gameCanvas = ref(null)
const gameStarted = ref(false)
const gameOver = ref(false)
const score = ref(0)
const highScore = ref(0)
const newHighScore = ref(false)

// Game settings
const canvasWidth = 400
const canvasHeight = 400
const gridSize = 20
const gameSpeed = 100 // Faster for smoother movement

// Game variables
let gameLoop = null
let appleTimer = null
let snake = []
let foods = []
let direction = 'right'
let nextDirection = 'right'
let animationFrame = null

// Initialize game
const initGame = () => {
  // Initialize snake
  snake = [
    { x: 6, y: 10 },
    { x: 5, y: 10 },
    { x: 4, y: 10 }
  ]

  // Initialize foods array
  foods = []

  // Generate initial foods
  generateFood()
  generateFood()
  generateFood()

  // Start apple generation timer
  appleTimer = setInterval(() => {
    if (foods.length < 5) { // Keep max 5 apples on screen
      generateFood()
    }
  }, 3000) // Generate new apple every 3 seconds

  // Reset game state
  direction = 'right'
  nextDirection = 'right'
  score.value = 0
  gameOver.value = false
  newHighScore.value = false
}

// Generate food at random position
const generateFood = () => {
  const maxX = Math.floor(canvasWidth / gridSize)
  const maxY = Math.floor(canvasHeight / gridSize)

  let newFood
  do {
    newFood = {
      x: Math.floor(Math.random() * maxX),
      y: Math.floor(Math.random() * maxY),
      createdAt: Date.now()
    }
  } while (
    snake.some(segment => segment.x === newFood.x && segment.y === newFood.y) ||
    foods.some(food => food.x === newFood.x && food.y === newFood.y)
  )

  foods.push(newFood)

  // Remove food after 7 seconds
  setTimeout(() => {
    const index = foods.findIndex(food => food === newFood)
    if (index > -1) {
      foods.splice(index, 1)
    }
  }, 7000)
}

// Handle mobile control input
const handleMobileControl = (newDirection) => {
  if (!gameStarted.value || gameOver.value) return

  switch (newDirection) {
    case 'up':
      if (direction !== 'down') nextDirection = 'up'
      break
    case 'down':
      if (direction !== 'up') nextDirection = 'down'
      break
    case 'left':
      if (direction !== 'right') nextDirection = 'left'
      break
    case 'right':
      if (direction !== 'left') nextDirection = 'right'
      break
  }
}

// Prevent default touch/mouse behavior
const preventDefault = (event) => {
  event.preventDefault()
}

// Handle keyboard input
const handleKeyPress = (event) => {
  const key = event.key.toLowerCase()

  switch (key) {
    case 'arrowup':
    case 'w':
      if (direction !== 'down') nextDirection = 'up'
      break
    case 'arrowdown':
    case 's':
      if (direction !== 'up') nextDirection = 'down'
      break
    case 'arrowleft':
    case 'a':
      if (direction !== 'right') nextDirection = 'left'
      break
    case 'arrowright':
    case 'd':
      if (direction !== 'left') nextDirection = 'right'
      break
    case ' ':
      if (!gameStarted.value) startGame()
      else if (gameOver.value) restartGame()
      break
  }
}

// Start the game
const startGame = () => {
  if (gameStarted.value) return

  initGame()
  gameStarted.value = true
  gameOver.value = false

  // Focus canvas for keyboard input
  nextTick(() => {
    gameCanvas.value.focus()
  })

  // Draw initial game state
  drawGame()

  gameLoop = setInterval(updateGame, gameSpeed)

  // Start smooth animation loop
  const animate = () => {
    drawGame()
    animationFrame = requestAnimationFrame(animate)
  }
  animate()
}

// Restart the game
const restartGame = () => {
  gameOver.value = false
  gameStarted.value = false
  clearInterval(gameLoop)
  clearInterval(appleTimer)
  if (animationFrame) {
    cancelAnimationFrame(animationFrame)
  }
  startGame()
}

// Close game over modal
const closeGameOver = () => {
  gameOver.value = false
}

// Update game state
const updateGame = () => {
  direction = nextDirection

  // Move snake
  const head = { ...snake[0] }

  switch (direction) {
    case 'up':
      head.y--
      break
    case 'down':
      head.y++
      break
    case 'left':
      head.x--
      break
    case 'right':
      head.x++
      break
  }

  // Wrap around the playground (no wall boundaries)
  const maxX = Math.floor(canvasWidth / gridSize)
  const maxY = Math.floor(canvasHeight / gridSize)

  if (head.x < 0) head.x = maxX - 1
  if (head.x >= maxX) head.x = 0
  if (head.y < 0) head.y = maxY - 1
  if (head.y >= maxY) head.y = 0

  // Check collision with self
  if (snake.some(segment => segment.x === head.x && segment.y === head.y)) {
    endGame()
    return
  }

  // Add new head
  snake.unshift(head)

    // Check if any food is eaten
  const eatenFoodIndex = foods.findIndex(food => food.x === head.x && food.y === head.y)
  if (eatenFoodIndex !== -1) {
    score.value += 10
    foods.splice(eatenFoodIndex, 1)
    generateFood() // Generate new food to maintain count

    // Increase speed slightly
    if (score.value % 50 === 0) {
      clearInterval(gameLoop)
      gameLoop = setInterval(updateGame, Math.max(50, gameSpeed - (score.value / 50) * 10))
    }
  } else {
    // Remove tail if no food eaten
    snake.pop()
  }
}

// End the game
const endGame = () => {
  gameOver.value = true
  gameStarted.value = false
  clearInterval(gameLoop)
  clearInterval(appleTimer)
  if (animationFrame) {
    cancelAnimationFrame(animationFrame)
  }

  // Update high score
  if (score.value > highScore.value) {
    highScore.value = score.value
    newHighScore.value = true
    $q.notify({
      type: 'positive',
      message: 'New High Score! 🎉',
      position: 'top'
    })
  }
}

// Draw the game
const drawGame = () => {
  if (!gameCanvas.value) return

  const ctx = gameCanvas.value.getContext('2d')

  // Clear canvas
  ctx.fillStyle = '#2c3e50'
  ctx.fillRect(0, 0, canvasWidth, canvasHeight)

  // Draw grid
  ctx.strokeStyle = '#34495e'
  ctx.lineWidth = 1
  for (let x = 0; x < canvasWidth; x += gridSize) {
    ctx.beginPath()
    ctx.moveTo(x, 0)
    ctx.lineTo(x, canvasHeight)
    ctx.stroke()
  }
  for (let y = 0; y < canvasHeight; y += gridSize) {
    ctx.beginPath()
    ctx.moveTo(0, y)
    ctx.lineTo(canvasWidth, y)
    ctx.stroke()
  }

  // Draw snake with enhanced graphics
  snake.forEach((segment, index) => {
    const color = index === 0 ? '#2ecc71' : '#27ae60'
    const isHead = index === 0
    drawPixelatedSnakeSegment(ctx, segment.x * gridSize + gridSize/2, segment.y * gridSize + gridSize/2, gridSize - 2, color, isHead)
  })

  // Draw all apple foods
  foods.forEach(food => {
    const timeLeft = 7000 - (Date.now() - food.createdAt)
    const alpha = Math.max(0.3, timeLeft / 7000) // Fade out effect
    drawApple(ctx, food.x * gridSize + gridSize/2, food.y * gridSize + gridSize/2, gridSize - 4, alpha)
  })
}

// Draw enhanced apple with smooth graphics
const drawApple = (ctx, x, y, size, alpha = 1) => {
  ctx.save()
  ctx.globalAlpha = alpha

  // Create gradient for 3D apple effect
  const gradient = ctx.createRadialGradient(x, y, 0, x, y, size/2)
  gradient.addColorStop(0, '#ff6b6b') // Bright red center
  gradient.addColorStop(0.7, '#e74c3c') // Main red
  gradient.addColorStop(1, '#c0392b') // Dark red edge

  // Draw apple body with gradient
  ctx.fillStyle = gradient
  ctx.beginPath()
  ctx.arc(x, y, size/2, 0, 2 * Math.PI)
  ctx.fill()

  // Add highlight for 3D effect
  ctx.fillStyle = 'rgba(255, 255, 255, 0.4)'
  ctx.beginPath()
  ctx.arc(x - size/6, y - size/6, size/6, 0, 2 * Math.PI)
  ctx.fill()

  // Add shadow for depth
  ctx.fillStyle = 'rgba(0, 0, 0, 0.2)'
  ctx.beginPath()
  ctx.arc(x + size/8, y + size/8, size/8, 0, 2 * Math.PI)
  ctx.fill()

  // Apple stem (brown)
  ctx.fillStyle = '#8B4513'
  ctx.fillRect(x - 1, y - size/2 - 3, 2, 5)

  // Apple leaf (green with gradient)
  const leafGradient = ctx.createLinearGradient(x + size/4, y - size/2, x + size/4 + 6, y - size/2 + 4)
  leafGradient.addColorStop(0, '#27ae60')
  leafGradient.addColorStop(1, '#2ecc71')

  ctx.fillStyle = leafGradient
  ctx.beginPath()
  ctx.ellipse(x + size/4, y - size/2 - 1, 4, 2, Math.PI/4, 0, 2 * Math.PI)
  ctx.fill()

  ctx.restore()
}

// Draw enhanced snake segment with smooth graphics
const drawPixelatedSnakeSegment = (ctx, x, y, size, color, isHead = false) => {
  ctx.save()

  // Create gradient for 3D effect
  const gradient = ctx.createRadialGradient(x, y, 0, x, y, size/2)
  if (isHead) {
    gradient.addColorStop(0, '#4CAF50') // Bright green center
    gradient.addColorStop(0.7, color)   // Main color
    gradient.addColorStop(1, '#1B5E20') // Dark green edge
  } else {
    gradient.addColorStop(0, '#66BB6A') // Light green center
    gradient.addColorStop(0.7, color)   // Main color
    gradient.addColorStop(1, '#2E7D32') // Dark green edge
  }

  // Draw main body with gradient
  ctx.fillStyle = gradient
  ctx.beginPath()
  ctx.arc(x, y, size/2, 0, 2 * Math.PI)
  ctx.fill()

  // Add highlight for 3D effect
  ctx.fillStyle = 'rgba(255, 255, 255, 0.3)'
  ctx.beginPath()
  ctx.arc(x - size/6, y - size/6, size/6, 0, 2 * Math.PI)
  ctx.fill()

  // Add eyes for head
  if (isHead) {
    ctx.fillStyle = '#000'
    ctx.beginPath()
    ctx.arc(x - size/8, y - size/8, 2, 0, 2 * Math.PI)
    ctx.fill()
    ctx.beginPath()
    ctx.arc(x + size/8, y - size/8, 2, 0, 2 * Math.PI)
    ctx.fill()
  }

  // Add scale pattern for body segments
  if (!isHead) {
    ctx.fillStyle = 'rgba(255, 255, 255, 0.1)'
    ctx.beginPath()
    ctx.arc(x, y, size/4, 0, 2 * Math.PI)
    ctx.fill()
  }

  ctx.restore()
}

// Load high score from localStorage
const loadHighScore = () => {
  const saved = localStorage.getItem('snakeHighScore')
  if (saved) {
    highScore.value = parseInt(saved)
  }
}

// Save high score to localStorage
const saveHighScore = () => {
  localStorage.setItem('snakeHighScore', highScore.value.toString())
}

// Lifecycle hooks
onMounted(() => {
  loadHighScore()
  initGame()
  drawGame()
})

onUnmounted(() => {
  if (gameLoop) {
    clearInterval(gameLoop)
  }
  if (appleTimer) {
    clearInterval(appleTimer)
  }
  if (animationFrame) {
    cancelAnimationFrame(animationFrame)
  }
  saveHighScore()
})
</script>

<style scoped>
.snake-game-page {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100vh;
  width: 100vw;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  overflow: hidden;
  position: fixed;
  top: 0;
  left: 0;
}

.snake-game-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 10px;
  box-sizing: border-box;
  max-height: 100vh;
  max-width: 100vw;
}

.game-header {
  margin-bottom: 20px;
  text-align: center;
}

.game-title {
  font-size: 3rem;
  margin: 0 0 10px 0;
  color: white;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
}

.game-info {
  display: flex;
  gap: 30px;
  justify-content: center;
  margin-bottom: 20px;
}

.score, .high-score {
  font-size: 1.2rem;
  font-weight: bold;
  color: white;
  background: rgba(255, 255, 255, 0.1);
  padding: 10px 20px;
  border-radius: 10px;
  backdrop-filter: blur(10px);
}

.game-container {
  margin-bottom: 20px;
}

.game-canvas {
  border: 3px solid #34495e;
  border-radius: 10px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
  background: #2c3e50;
  cursor: pointer;
}

  .mobile-controls {
    display: none; /* Hide by default on larger screens */
    flex-direction: column;
    gap: 15px;
    margin-top: 20px;
    align-items: center;
  }

  .control-row {
    display: flex;
    gap: 15px;
    justify-content: center;
    align-items: center;
  }

  .control-btn {
    width: 70px;
    height: 70px;
    display: flex;
    align-items: center;
    justify-content: center;
    background: rgba(52, 152, 219, 0.8);
    border: 3px solid #2980b9;
    border-radius: 50%;
    transition: all 0.2s ease;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
    color: white;
    font-weight: bold;
  }

  .control-btn:hover {
    background: rgba(52, 152, 219, 1);
    transform: scale(1.05);
    box-shadow: 0 6px 12px rgba(0, 0, 0, 0.4);
  }

  .control-btn:active {
    background: rgba(41, 128, 185, 1);
    transform: scale(0.95);
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
  }

  .control-btn.up-btn {
    margin-bottom: 10px;
  }

  .control-btn.left-btn {
    margin-right: 10px;
  }

  .control-btn.right-btn {
    margin-left: 10px;
  }

.game-controls {
  text-align: center;
}

.start-btn, .restart-btn {
  margin: 10px;
  font-weight: bold;
  font-size: 1.1rem;
}

.instructions {
  margin-top: 20px;
  color: white;
  font-size: 0.9rem;
  opacity: 0.8;
}

.instructions p {
  margin: 5px 0;
}

.game-over-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.8);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.game-over-modal {
  background: white;
  padding: 40px;
  border-radius: 15px;
  text-align: center;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.3);
}

.game-over-modal h2 {
  color: #e74c3c;
  margin: 0 0 20px 0;
  font-size: 2rem;
}

.game-over-modal p {
  margin: 10px 0;
  font-size: 1.2rem;
}

.new-high-score {
  color: #27ae60;
  font-weight: bold;
  font-size: 1.3rem;
}

.modal-buttons {
  display: flex;
  gap: 10px;
  justify-content: center;
  margin-top: 20px;
}

.restart-modal-btn, .close-modal-btn {
  min-width: 100px;
}

  @media (max-width: 600px) {
    .game-title {
      font-size: 1.5rem;
      margin: 0 0 5px 0;
    }

    .game-info {
      flex-direction: column;
      gap: 5px;
      margin-bottom: 10px;
    }

    .score, .high-score {
      font-size: 1rem;
      padding: 5px 10px;
    }

    .game-canvas {
      width: 280px;
      height: 280px;
    }

    .instructions {
      font-size: 0.8rem;
      margin-top: 10px;
    }

    .instructions p {
      margin: 2px 0;
    }

    .mobile-controls {
      display: flex; /* Show on mobile screens */
      margin-top: 15px;
    }

    .control-btn {
      width: 60px;
      height: 60px;
      font-size: 0.9rem;
    }

    .control-row {
      gap: 10px;
    }
  }
</style>

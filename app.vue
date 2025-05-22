<template>
  <div class="main-container">
    <div class="roulette-container">
      <div class="game-header">
        <h1>Segundo <span>Premio</span></h1>
      </div>
      
      <div class="roulette-table">
        <div class="roulette-wheel-container">
          <div class="roulette-wheel">
            <div class="outer-ring"></div>
            <div class="spinning-wheel" :style="{ transform: `rotate(${rotation}deg)` }">
              <div 
                v-for="(number, index) in numbers" 
                :key="index" 
                class="number-slot"
                :class="{ 'even': index % 2 === 0 }"
                :style="{ transform: `rotate(${index * 3.6}deg)` }"
              >
                <div class="number-content" :style="{ transform: `rotate(${-index * 3.6}deg)` }">
                  {{ number }}
                </div>
              </div>
            </div>
            <div class="center-circle">
              <div class="inner-circle"></div>
            </div>
            <div class="pointer-container">
              <div class="pointer"></div>
            </div>
          </div>
        </div>
        
        <div class="game-info">
          <div v-if="result !== null" class="result">
            <div class="result-display">
              <span>{{ result }}</span>
            </div>
            <h2>Last Number</h2>
          </div>
          <div v-else class="result empty">
            <div class="result-display empty">
              <span>--</span>
            </div>
            <h2>Waiting...</h2>
          </div>
        </div>
      </div>
      
      <div class="controls">
        <button @click="spinWheel" :disabled="spinning" class="spin-button">
          <span class="button-text">{{ spinning ? 'SPINNING...' : 'SPIN' }}</span>
          <span class="button-shine"></span>
        </button>
      </div>
    </div>
    
    <div class="divider"></div>
    
    <!-- Second Roulette Component -->
    <RuletteV2 />
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import RuletteV2 from './components/RuletteV2.vue';

// Generate numbers from 00 to 99
const numbers = computed(() => {
  const nums = [];
  for (let i = 0; i < 100; i++) {
    nums.push(i < 10 ? `0${i}` : `${i}`);
  }
  return nums;
});

const spinning = ref(false);
const rotation = ref(0);
const result = ref(null);

// Sound effects
let spinSound, winSound;

onMounted(() => {
  // Initialize sound effects if browser supports Audio API
  try {
    spinSound = new Audio('https://assets.mixkit.co/active_storage/sfx/2006/2006-preview.mp3');
    spinSound.volume = 0.5;
    
    winSound = new Audio('https://assets.mixkit.co/active_storage/sfx/2000/2000-preview.mp3');
    winSound.volume = 0.7;
  } catch (e) {
    console.log('Audio not supported');
  }
});

const spinWheel = () => {
  if (spinning.value) return;
  
  spinning.value = true;
  result.value = null;
  
  // Play spin sound
  if (spinSound) {
    spinSound.currentTime = 0;
    spinSound.play().catch(e => console.log('Could not play sound'));
  }
  
  // Generate a random number of rotations (between 5 and 10 full rotations)
  const rotations = 5 + Math.random() * 5;
  
  // Calculate the position for number 19
  // Find the index of number 19 in the numbers array
  const targetIndex = numbers.value.findIndex(num => num === '19');
  
  // Calculate the angle needed to land on 19
  // We need to calculate the opposite position because the result is determined
  // by the number at the top of the wheel (opposite to the pointer)
  const oppositeIndex = (targetIndex + 50) % 100; // 50 positions away (180 degrees)
  
  // Calculate the final rotation to make it land on 19
  // We add a small random offset to make it look more natural
  const randomOffset = (Math.random() * 0.5) - 0.25; // Small random offset between -0.25 and 0.25 degrees
  const finalRotation = rotation.value + (rotations * 360) + (oppositeIndex * 3.6) + randomOffset;
  
  // Animate the rotation
  const startTime = Date.now();
  const duration = 7000; // 7 seconds for a longer, more dramatic spin
  const startRotation = rotation.value;
  const rotationDiff = finalRotation - startRotation;
  
  const animate = () => {
    const currentTime = Date.now();
    const elapsed = currentTime - startTime;
    
    if (elapsed < duration) {
      // Easing function for a more realistic spin
      const progress = 1 - Math.pow(1 - elapsed / duration, 4); // More dramatic easing
      rotation.value = startRotation + (rotationDiff * progress);
      requestAnimationFrame(animate);
    } else {
      // Animation complete
      rotation.value = finalRotation % 360;
      spinning.value = false;
      
      // Force the result to be 19
      result.value = '19';
      
      // Play win sound
      if (winSound) {
        winSound.currentTime = 0;
        winSound.play().catch(e => console.log('Could not play sound'));
      }
    }
  };
  
  requestAnimationFrame(animate);
};
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Montserrat:wght@400;700;900&display=swap');

:root {
  --primary-color: #e63946;
  --secondary-color: #1d3557;
  --accent-color: #f1faee;
  --dark-color: #1d3557;
  --light-color: #a8dadc;
  --gold-color: #ffd700;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  background-color: #f5f5f5;
  background-image: linear-gradient(315deg, #2a2a72 0%, #009ffd 74%);
  min-height: 100vh;
}

.main-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
  max-width: 1400px;
  margin: 0 auto;
  padding: 2rem;
}

.divider {
  width: 80%;
  height: 3px;
  background: linear-gradient(to right, transparent, var(--gold-color), transparent);
  margin: 3rem 0;
  border-radius: 50%;
}

.roulette-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 2rem;
  font-family: 'Montserrat', sans-serif;
  max-width: 1200px;
  width: 100%;
  background-color: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  border-radius: 20px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
}

.game-header {
  width: 100%;
  text-align: center;
  margin-bottom: 2rem;
  padding: 1rem;
  background: var(--secondary-color);
  border-radius: 10px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
}

.game-header h1 {
  color: white;
  font-size: 3rem;
  font-weight: 900;
  text-transform: uppercase;
  letter-spacing: 2px;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
}

.game-header h1 span {
  color: var(--gold-color);
}

.roulette-table {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
  margin-bottom: 2rem;
  background: rgba(29, 53, 87, 0.8);
  border-radius: 20px;
  padding: 2rem;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
}

@media (min-width: 768px) {
  .roulette-table {
    flex-direction: row;
    justify-content: space-around;
  }
}

.roulette-wheel-container {
  position: relative;
  padding: 20px;
}

.roulette-wheel {
  position: relative;
  width: 350px;
  height: 350px;
  border-radius: 50%;
  background: #333;
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: hidden;
  box-shadow: 
    0 0 0 15px #222,
    0 0 0 30px var(--dark-color),
    0 5px 20px rgba(0, 0, 0, 0.5),
    0 0 40px rgba(0, 0, 0, 0.3) inset;
}

.outer-ring {
  position: absolute;
  width: 100%;
  height: 100%;
  border-radius: 50%;
  background: linear-gradient(135deg, #333 0%, #111 100%);
  z-index: 1;
}

.spinning-wheel {
  position: absolute;
  width: 90%;
  height: 90%;
  border-radius: 50%;
  background: radial-gradient(circle, #222, #444);
  z-index: 2;
  transition: transform 0.05s linear;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.5) inset;
}

.number-slot {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  transform-origin: center;
  clip-path: polygon(50% 0%, 52% 50%, 48% 50%);
  background-color: #e63946;
}

.number-slot.even {
  background-color: #1d3557;
}

.number-content {
  position: absolute;
  width: 30px;
  top: 15px;
  left: calc(50% - 15px);
  color: white;
  font-weight: bold;
  font-size: 12px;
  text-align: center;
  text-shadow: 1px 1px 1px rgba(0, 0, 0, 0.5);
}

.center-circle {
  position: absolute;
  width: 80px;
  height: 80px;
  border-radius: 50%;
  background: radial-gradient(circle, #666, #333);
  z-index: 4;
  display: flex;
  justify-content: center;
  align-items: center;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
}

.inner-circle {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  background: radial-gradient(circle, #222, #111);
  border: 2px solid #gold;
}

.pointer-container {
  position: absolute;
  top: -15px;
  left: calc(50% - 15px);
  width: 30px;
  height: 50px;
  z-index: 10;
}

.pointer {
  width: 30px;
  height: 30px;
  background-color: var(--gold-color);
  clip-path: polygon(50% 0%, 0% 100%, 100% 100%);
  filter: drop-shadow(0 3px 5px rgba(0, 0, 0, 0.5));
}

.game-info {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 2rem;
}

.result {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 1rem;
}

.result-display {
  width: 100px;
  height: 100px;
  border-radius: 50%;
  background: linear-gradient(145deg, #1d3557, #0f1a2b);
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 1rem;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
  border: 3px solid var(--gold-color);
}

.result-display span {
  font-size: 2.5rem;
  font-weight: bold;
  color: white;
  text-shadow: 0 0 10px rgba(255, 215, 0, 0.7);
}

.result-display.empty {
  background: linear-gradient(145deg, #333, #222);
  border: 3px solid #666;
}

.result-display.empty span {
  color: #666;
  text-shadow: none;
}

.result h2 {
  color: white;
  font-size: 1.2rem;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.controls {
  margin-top: 2rem;
  width: 100%;
  display: flex;
  justify-content: center;
}

.spin-button {
  position: relative;
  padding: 1rem 3rem;
  font-size: 1.5rem;
  font-weight: bold;
  text-transform: uppercase;
  letter-spacing: 2px;
  background: linear-gradient(to right, #e63946, #d00000);
  color: white;
  border: none;
  border-radius: 50px;
  cursor: pointer;
  transition: all 0.3s;
  overflow: hidden;
  box-shadow: 0 5px 15px rgba(230, 57, 70, 0.4);
}

.spin-button:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 25px rgba(230, 57, 70, 0.6);
}

.spin-button:active {
  transform: translateY(1px);
  box-shadow: 0 3px 10px rgba(230, 57, 70, 0.4);
}

.spin-button:disabled {
  background: linear-gradient(to right, #888, #666);
  cursor: not-allowed;
  transform: none;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
}

.button-text {
  position: relative;
  z-index: 2;
}

.button-shine {
  position: absolute;
  top: 0;
  left: -100%;
  width: 50%;
  height: 100%;
  background: linear-gradient(
    to right,
    rgba(255, 255, 255, 0) 0%,
    rgba(255, 255, 255, 0.3) 50%,
    rgba(255, 255, 255, 0) 100%
  );
  transform: skewX(-25deg);
  animation: shine 3s infinite;
}

@keyframes shine {
  0% {
    left: -100%;
  }
  20% {
    left: 100%;
  }
  100% {
    left: 100%;
  }
}

/* Add some animations */
@keyframes pulse {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.05);
  }
  100% {
    transform: scale(1);
  }
}

@keyframes glow {
  0% {
    box-shadow: 0 0 5px rgba(255, 215, 0, 0.5);
  }
  50% {
    box-shadow: 0 0 20px rgba(255, 215, 0, 0.8);
  }
  100% {
    box-shadow: 0 0 5px rgba(255, 215, 0, 0.5);
  }
}

.result-display {
  animation: glow 2s infinite;
}

.spin-button:not(:disabled) {
  animation: pulse 2s infinite;
}
</style>

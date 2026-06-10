<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import NavBar from '../components/NavBar.vue'
import { useSRS } from '../composables/useSRS.js'
import { useProgress } from '../composables/useProgress.js'

const router = useRouter()
const { getDueCards, reviewCard } = useSRS()
const { completeLesson, getStreak } = useProgress()

const allVocab = ref([])
const cards = ref([])
const currentIndex = ref(0)
const isFlipped = ref(false)
const sessionComplete = ref(false)
const hasTTS = ref('speechSynthesis' in window)
const cardsStudied = ref(0)
const xpEarned = ref(0)
const dueCount = ref(0)

const currentCard = computed(() => cards.value[currentIndex.value] || null)
const progress = computed(() => `Card ${currentIndex.value + 1} of ${cards.value.length}`)

onMounted(async () => {
  allVocab.value = await fetch('/content/topic1-vocab.json').then(r => r.json())

  const due = getDueCards(null, allVocab.value)
  dueCount.value = due.length
  cards.value = due

  if ('speechSynthesis' in window) {
    speechSynthesis.onvoiceschanged = () => {
      hasTTS.value = 'speechSynthesis' in window
    }
  }
})

function flipCard() {
  isFlipped.value = true
}

function rate(quality) {
  reviewCard(currentCard.value.id, quality)
  cardsStudied.value++

  if (currentIndex.value < cards.value.length - 1) {
    currentIndex.value++
    isFlipped.value = false
  } else {
    const sessions = parseInt(localStorage.getItem('korean-review-sessions') || '0', 10) + 1
    localStorage.setItem('korean-review-sessions', sessions.toString())
    completeLesson(0)
    xpEarned.value += 20
    sessionComplete.value = true
  }
}

function speak(text) {
  if ('speechSynthesis' in window) {
    speechSynthesis.cancel()
    const utter = new SpeechSynthesisUtterance(text)
    utter.lang = 'ko-KR'
    const voices = speechSynthesis.getVoices()
    const koVoice = voices.find(v => v.lang.startsWith('ko')) || voices[0]
    if (koVoice) utter.voice = koVoice
    utter.rate = 0.85
    speechSynthesis.speak(utter)
  }
}

function restart() {
  const due = getDueCards(null, allVocab.value)
  cards.value = due
  currentIndex.value = 0
  isFlipped.value = false
  sessionComplete.value = false
  cardsStudied.value = 0
  xpEarned.value = 0
}
</script>

<template>
  <div class="review-page">
    <NavBar :showBack="true" />
    <div class="container">
      <!-- Session complete -->
      <div v-if="sessionComplete" class="complete-screen">
        <h2>Review Complete!</h2>
        <p>You reviewed {{ cardsStudied }} cards. +{{ xpEarned }} XP</p>
        <div class="complete-actions">
          <button class="btn-gold" @click="restart">Review Again</button>
          <button class="btn-outline" @click="router.push('/lessons')">Back to Lessons</button>
        </div>
      </div>

      <!-- No cards due -->
      <div v-else-if="cards.length === 0" class="empty-screen">
        <div class="empty-icon">🎉</div>
        <h2>All caught up!</h2>
        <p>No cards due for review right now. Come back later or start a new lesson!</p>
        <div class="empty-actions">
          <button class="btn-gold" @click="router.push('/lessons')">Go to Lessons</button>
        </div>
      </div>

      <!-- Review flashcard -->
      <div v-else-if="currentCard" class="study-area">
        <div class="study-header">
          <span class="progress-text">{{ progress }}</span>
          <span class="bridge-hint">한자 Bridge</span>
        </div>

        <div class="progress-bar">
          <div
            class="progress-fill"
            :style="{ width: ((currentIndex + 1) / cards.length) * 100 + '%' }"
          ></div>
        </div>

        <div class="card-scene" @click="!isFlipped && flipCard()">
          <div class="card" :class="{ 'is-flipped': isFlipped }">
            <div class="card-face card-front">
              <span class="card-hanja">{{ currentCard.hanja }}</span>
              <span class="card-korean">{{ currentCard.korean }}</span>
              <span class="card-pronunciation">{{ currentCard.pronunciation }}</span>
              <span class="card-hint">Tap to flip</span>
            </div>
            <div class="card-face card-back">
              <button v-if="hasTTS" class="speak-btn" @click.stop="speak(currentCard.korean)">🔊</button>
              <span class="card-hanja-large">{{ currentCard.hanja }}</span>
              <span class="card-chinese">{{ currentCard.chinese }} ({{ currentCard.pinyin }})</span>
              <span class="card-meaning">{{ currentCard.meaning }}</span>
              <div class="bridge-note">{{ currentCard.bridgeNote }}</div>
            </div>
          </div>
        </div>

        <div v-if="!isFlipped" class="action-row">
          <button class="btn-gold" @click="flipCard">Show Answer</button>
        </div>

        <div v-else class="rating-row">
          <button class="rate-btn rate-hard" @click="rate(1)">Hard</button>
          <button class="rate-btn rate-medium" @click="rate(3)">Good</button>
          <button class="rate-btn rate-easy" @click="rate(5)">Easy</button>
        </div>

        <div class="session-stats">
          <div class="stat-item">
            <span class="stat-value">{{ dueCount }}</span>
            <span class="stat-label">Due Today</span>
          </div>
          <div class="stat-item">
            <span class="stat-value">{{ cardsStudied }}</span>
            <span class="stat-label">Reviewed</span>
          </div>
          <div class="stat-item">
            <span class="stat-value">🔥 {{ getStreak() }}</span>
            <span class="stat-label">Day Streak</span>
          </div>
        </div>
      </div>

      <!-- Loading -->
      <div v-else class="loading">Loading cards...</div>
    </div>
  </div>
</template>

<style scoped>
.review-page {
  padding-top: 0;
}

.study-area {
  padding-top: 16px;
}

.study-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
}

.progress-text {
  font-size: 0.85rem;
  color: var(--color-text-light);
  font-weight: 500;
}

.bridge-hint {
  font-size: 0.75rem;
  font-weight: 600;
  background: rgba(52, 152, 219, 0.1);
  color: var(--color-blue);
  padding: 4px 10px;
  border-radius: 20px;
}

.progress-bar {
  height: 4px;
  background: var(--color-border);
  border-radius: 2px;
  margin-bottom: 24px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: var(--color-gold);
  border-radius: 2px;
  transition: width 0.3s ease;
}

.card-scene {
  perspective: 800px;
  margin-bottom: 24px;
  cursor: pointer;
}

.card {
  position: relative;
  width: 100%;
  min-height: 300px;
  transform-style: preserve-3d;
  transition: transform 0.5s ease;
}

.card.is-flipped {
  transform: rotateY(180deg);
}

.card-face {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  min-height: 300px;
  backface-visibility: hidden;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: var(--color-card);
  border: 1px solid var(--color-border);
  border-radius: var(--radius);
  box-shadow: var(--shadow);
  padding: 32px 20px;
  gap: 8px;
}

.card-back {
  transform: rotateY(180deg);
  position: relative;
}

.speak-btn {
  position: absolute;
  top: 12px;
  right: 12px;
  background: rgba(243, 156, 18, 0.15);
  border: none;
  border-radius: 50%;
  width: 44px;
  height: 44px;
  font-size: 1.2rem;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background 0.2s;
}

.speak-btn:hover {
  background: rgba(243, 156, 18, 0.3);
}

.card-hanja {
  font-size: 3rem;
  font-weight: 700;
  color: var(--color-primary);
  line-height: 1.2;
}

.card-hanja-large {
  font-size: 2.5rem;
  font-weight: 700;
  color: var(--color-primary);
}

.card-korean {
  font-size: 1.8rem;
  font-weight: 600;
  color: var(--color-text);
}

.card-pronunciation {
  font-size: 1rem;
  color: var(--color-text-light);
}

.card-chinese {
  font-size: 1.2rem;
  font-weight: 500;
  color: #c0392b;
}

.card-meaning {
  font-size: 1rem;
  color: var(--color-text);
}

.card-hint {
  font-size: 0.85rem;
  color: var(--color-text-light);
}

.bridge-note {
  margin-top: 8px;
  background: rgba(52, 152, 219, 0.08);
  border: 1px solid rgba(52, 152, 219, 0.15);
  border-radius: 8px;
  padding: 8px 14px;
  font-size: 0.8rem;
  color: var(--color-blue);
  text-align: center;
  max-width: 300px;
}

.action-row {
  display: flex;
  justify-content: center;
}

.btn-gold {
  background: var(--color-gold);
  color: #2c3e50;
  font-weight: 600;
  font-size: 1rem;
  padding: 12px 32px;
  border-radius: var(--radius);
  transition: background 0.2s;
}

.btn-gold:hover {
  background: var(--color-gold-light);
}

.rating-row {
  display: flex;
  gap: 12px;
  justify-content: center;
}

.rate-btn {
  flex: 1;
  max-width: 120px;
  padding: 12px 8px;
  border-radius: var(--radius);
  font-weight: 600;
  font-size: 0.95rem;
  transition: opacity 0.2s;
}

.rate-btn:hover {
  opacity: 0.85;
}

.rate-hard {
  background: #e74c3c;
  color: #fff;
}

.rate-medium {
  background: var(--color-gold);
  color: #2c3e50;
}

.rate-easy {
  background: #27ae60;
  color: #fff;
}

.complete-screen {
  text-align: center;
  padding: 64px 16px;
}

.complete-screen h2 {
  font-size: 1.6rem;
  color: var(--color-primary);
  margin-bottom: 8px;
}

.complete-screen p {
  color: var(--color-text-light);
  margin-bottom: 24px;
}

.complete-actions {
  display: flex;
  flex-direction: column;
  gap: 12px;
  align-items: center;
}

.btn-outline {
  background: transparent;
  color: var(--color-text);
  font-weight: 600;
  font-size: 1rem;
  padding: 12px 32px;
  border-radius: var(--radius);
  border: 1px solid var(--color-border);
}

.btn-outline:hover {
  background: var(--color-border);
}

.empty-screen {
  text-align: center;
  padding: 64px 16px;
}

.empty-icon {
  font-size: 4rem;
  margin-bottom: 16px;
}

.empty-screen h2 {
  font-size: 1.6rem;
  color: var(--color-text);
  margin-bottom: 8px;
}

.empty-screen p {
  color: var(--color-text-light);
  margin-bottom: 24px;
}

.empty-actions {
  display: flex;
  justify-content: center;
}

.session-stats {
  display: flex;
  justify-content: space-around;
  background: var(--color-card);
  border: 1px solid var(--color-border);
  border-radius: var(--radius);
  padding: 16px 12px;
  margin-top: 24px;
}

.stat-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
}

.stat-value {
  font-size: 1.1rem;
  font-weight: 700;
  color: var(--color-text);
}

.stat-label {
  font-size: 0.75rem;
  color: var(--color-text-light);
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.loading {
  text-align: center;
  padding: 64px 16px;
  color: var(--color-text-light);
}
</style>

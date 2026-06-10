<script setup>
import { ref, computed } from 'vue'
import { useRouter } from 'vue-router'
import NavBar from '../components/NavBar.vue'
import { useProgress } from '../composables/useProgress.js'

const router = useRouter()
const { state, getStreak, isLessonComplete } = useProgress()

const lessons = [
  { id: 1, title: 'Greetings & Basics', subtitle: '인사', wordCount: 10, desc: 'Hello, goodbye, thank you — built from Chinese characters you already know' },
  { id: 2, title: 'Numbers & Counting', subtitle: '숫자', wordCount: 10, desc: 'Chinese numbers unlock Korean Sino-Korean numbers instantly' },
  { id: 3, title: 'Days & Months', subtitle: '날짜', wordCount: 10, desc: 'Date/time words from Classical Chinese — same characters!' },
  { id: 4, title: 'Family & People', subtitle: '가족', wordCount: 10, desc: '자 family words — many derive from Chinese hanja' },
  { id: 5, title: 'Food & Drink', subtitle: '음식', wordCount: 10, desc: 'Eating vocabulary — Chinese characters reveal Korean meaning' },
  { id: 6, title: 'Body & Health', subtitle: '몸', wordCount: 10, desc: '몸 body words — Sino-Korean anatomical terms' },
  { id: 7, title: 'Shopping & Money', subtitle: '쇼핑', wordCount: 10, desc: 'Commerce vocabulary — 돈 money from Chinese 錢' },
  { id: 8, title: 'Travel & Places', subtitle: '여행', wordCount: 10, desc: 'Place words — 역 station, 공항 airport from Chinese' },
  { id: 9, title: 'Work & Business', subtitle: '일', wordCount: 10, desc: '일 work words — 사무실 office from Chinese 事務室' },
  { id: 10, title: 'Science & Technology', subtitle: '과학', wordCount: 10, desc: '科學 — Chinese speakers know this word already!' },
]

const topik2Lessons = [
  { id: 11, title: 'Education & Learning', subtitle: '교육', wordCount: 10 },
  { id: 12, title: 'Emotions & Psychology', subtitle: '감정', wordCount: 10 },
  { id: 13, title: 'Society & Culture', subtitle: '사회', wordCount: 10 },
  { id: 14, title: 'Nature & Environment', subtitle: '자연', wordCount: 10 },
  { id: 15, title: 'Communication & Media', subtitle: '통신', wordCount: 10 },
]

const topik2Expanded = ref(false)

const lessonsCompleted = computed(() =>
  [...lessons, ...topik2Lessons].filter(l => isLessonComplete(l.id)).length
)

const totalLessons = computed(() => lessons.length + topik2Lessons.length)

function openLesson(lesson) {
  router.push(`/study/${lesson.id}`)
}
</script>

<template>
  <div class="lessons-page">
    <NavBar :showBack="true" />
    <div class="container">
      <div class="level-header">
        <h1>TOPIK I</h1>
        <p>Beginner — First 100 words with hanja breakdowns</p>
      </div>
      <div class="stats-bar">
        <span class="stat-item">⭐ {{ state.xp }} XP</span>
        <span class="stat-item">🔥 {{ getStreak() }} streak</span>
        <span class="stat-item">{{ lessonsCompleted }} / {{ totalLessons }} lessons</span>
      </div>

      <div class="bridge-banner">
        <span class="bridge-icon">💡</span>
        <p>Each lesson shows how <strong>Korean words map to Chinese characters</strong> — your Chinese knowledge is your secret weapon!</p>
      </div>

      <div class="lesson-list">
        <div
          v-for="lesson in lessons"
          :key="lesson.id"
          class="lesson-card"
          @click="openLesson(lesson)"
        >
          <div class="lesson-info">
            <span class="lesson-number">Lesson {{ lesson.id }} · {{ lesson.subtitle }}</span>
            <h3 class="lesson-title">{{ lesson.title }}</h3>
            <span class="lesson-desc">{{ lesson.desc }}</span>
          </div>
          <span class="lesson-words">{{ lesson.wordCount }} words</span>
          <span v-if="isLessonComplete(lesson.id)" class="lesson-badge badge-complete">
            ✓
          </span>
          <span v-else class="lesson-badge badge-ready">
            Start
          </span>
        </div>
      </div>

      <div class="level-header topik2-header" @click="topik2Expanded = !topik2Expanded">
        <h1>TOPIK II <span class="expand-icon">{{ topik2Expanded ? '▾' : '▸' }}</span></h1>
        <p>Intermediate — More hanja vocabulary</p>
      </div>

      <div v-if="topik2Expanded" class="lesson-list">
        <div
          v-for="lesson in topik2Lessons"
          :key="lesson.id"
          class="lesson-card"
          @click="openLesson(lesson)"
        >
          <div class="lesson-info">
            <span class="lesson-number">Lesson {{ lesson.id }} · {{ lesson.subtitle }}</span>
            <h3 class="lesson-title">{{ lesson.title }}</h3>
            <span class="lesson-desc">{{ lesson.wordCount }} words</span>
          </div>
          <span v-if="isLessonComplete(lesson.id)" class="lesson-badge badge-complete">
            ✓
          </span>
          <span v-else class="lesson-badge badge-ready">
            Start
          </span>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.level-header {
  text-align: center;
  padding: 32px 0 16px;
}

.level-header h1 {
  font-size: 1.8rem;
  color: var(--color-primary);
}

.level-header p {
  color: var(--color-text-light);
  font-size: 0.95rem;
}

.stats-bar {
  display: flex;
  justify-content: center;
  gap: 16px;
  padding: 12px 16px;
  margin-bottom: 16px;
  background: rgba(243, 156, 18, 0.15);
  border-radius: var(--radius);
}

.stat-item {
  font-size: 0.85rem;
  font-weight: 600;
  color: var(--color-gold);
}

.bridge-banner {
  display: flex;
  align-items: center;
  gap: 12px;
  background: rgba(52, 152, 219, 0.1);
  border: 1px solid rgba(52, 152, 219, 0.2);
  border-radius: var(--radius);
  padding: 12px 16px;
  margin-bottom: 20px;
}

.bridge-icon {
  font-size: 1.4rem;
  flex-shrink: 0;
}

.bridge-banner p {
  font-size: 0.85rem;
  color: var(--color-text-light);
  line-height: 1.4;
}

.bridge-banner strong {
  color: var(--color-primary);
}

.lesson-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
  padding-bottom: 32px;
}

.lesson-card {
  display: flex;
  align-items: center;
  gap: 12px;
  background: var(--color-card);
  border: 1px solid var(--color-border);
  border-radius: var(--radius);
  padding: 16px 20px;
  box-shadow: var(--shadow);
  cursor: pointer;
  transition: transform 0.15s ease;
}

.lesson-card:hover {
  transform: translateY(-1px);
}

.lesson-info {
  flex: 1;
}

.lesson-number {
  font-size: 0.8rem;
  color: var(--color-text-light);
}

.lesson-title {
  font-size: 1.05rem;
  font-weight: 600;
  margin: 2px 0;
  color: var(--color-primary);
}

.lesson-desc {
  font-size: 0.82rem;
  color: var(--color-text-light);
  display: block;
  line-height: 1.4;
}

.lesson-words {
  font-size: 0.8rem;
  color: var(--color-text-light);
  white-space: nowrap;
}

.lesson-badge {
  font-size: 0.75rem;
  font-weight: 600;
  padding: 4px 10px;
  border-radius: 20px;
  white-space: nowrap;
  flex-shrink: 0;
}

.badge-ready {
  background: #e8f4fd;
  color: var(--color-blue);
}

.badge-complete {
  background: #e8f8f5;
  color: #27ae60;
}

.topik2-header {
  cursor: pointer;
  user-select: none;
  padding-top: 24px;
}

.topik2-header:hover {
  opacity: 0.85;
}

.expand-icon {
  font-size: 0.9rem;
  color: var(--color-text-light);
}
</style>

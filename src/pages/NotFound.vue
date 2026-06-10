<script setup>
import { useRouter } from 'vue-router'
const router = useRouter()
</script>
<template>
  <div class="not-found">
    <div class="content">
      <div class="icon">한</div>
      <h1>404</h1>
      <p>Page not found</p>
      <button class="btn" @click="router.push('/')">Go Home</button>
    </div>
  </div>
</template>
<style scoped>
.not-found {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--color-bg);
}
.content {
  text-align: center;
  padding: 40px;
}
.icon {
  font-size: 4rem;
  margin-bottom: 16px;
}
h1 {
  font-size: 2rem;
  color: var(--color-primary);
  margin-bottom: 8px;
}
p {
  color: var(--color-text-light);
  margin-bottom: 24px;
}
.btn {
  background: var(--color-primary);
  color: #fff;
  padding: 12px 32px;
  border-radius: var(--radius);
  font-weight: 600;
  font-size: 1rem;
}
</style>

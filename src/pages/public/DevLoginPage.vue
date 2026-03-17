<script setup>
import { useRouter } from 'vue-router'
import { useAuthStore } from '@/stores/auth'

const router = useRouter()
const auth = useAuthStore()

async function loginAs(role) {
  auth.setToken('dev-token')
  auth.setUser({
    id: `dev-${role.toLowerCase()}-1`,
    role,
    name: `Dev ${role}`,
    email: `${role.toLowerCase()}@example.com`
  })
  await router.replace(auth.defaultHomeRoute())
}
</script>

<template>
  <section class="card">
    <h1>开发快捷登录</h1>
    <p class="muted">不接后端时，用于直接进入各角色页面。</p>

    <div class="grid">
      <button class="btn" @click="loginAs('Customer')">以 Customer 进入</button>
      <button class="btn" @click="loginAs('Specialist')">以 Specialist 进入</button>
      <button class="btn" @click="loginAs('Admin')">以 Admin 进入</button>
    </div>

    <p class="hint">
      入口地址：
      <code>/dev-login</code>
    </p>
  </section>
</template>

<style scoped>
.card {
  max-width: 720px;
  margin: 26px auto 0;
  background: rgba(255, 255, 255, 0.06);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 16px;
  padding: 18px 18px 16px;
}
h1 {
  margin: 0 0 6px;
  font-size: 22px;
}
.muted {
  margin: 0 0 14px;
  opacity: 0.8;
}
.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
}
.btn {
  padding: 10px 12px;
  border-radius: 12px;
  border: 1px solid rgba(255, 255, 255, 0.16);
  background: rgba(169, 182, 255, 0.18);
  color: #eef0ff;
  cursor: pointer;
}
.hint {
  margin: 12px 0 0;
  opacity: 0.9;
}
code {
  background: rgba(0, 0, 0, 0.25);
  padding: 2px 6px;
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.14);
}
@media (max-width: 880px) {
  .grid {
    grid-template-columns: 1fr;
  }
}
</style>


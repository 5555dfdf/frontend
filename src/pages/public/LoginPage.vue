<script setup>
import { ref } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { useAuthStore } from '@/stores/auth'

const auth = useAuthStore()
const router = useRouter()
const route = useRoute()

const email = ref('')
const password = ref('')
const loading = ref(false)
const error = ref('')

async function onSubmit() {
  error.value = ''
  loading.value = true
  try {
    await auth.login({ email: email.value, password: password.value })
    const next = typeof route.query.next === 'string' ? route.query.next : ''
    if (next) await router.replace(next)
    else await router.replace(auth.defaultHomeRoute())
  } catch (e) {
    error.value = e?.message || '登录失败'
  } finally {
    loading.value = false
  }
}
</script>

<template>
  <section class="card">
    <h1>登录</h1>
    <p class="muted">使用你的账号登录（按角色进入对应工作台）。</p>

    <form class="form" @submit.prevent="onSubmit">
      <label>
        <div class="label">邮箱</div>
        <input v-model="email" type="email" autocomplete="email" required />
      </label>

      <label>
        <div class="label">密码</div>
        <input v-model="password" type="password" autocomplete="current-password" required />
      </label>

      <div v-if="error" class="error">{{ error }}</div>

      <button class="btn" type="submit" :disabled="loading">
        {{ loading ? '登录中...' : '登录' }}
      </button>
    </form>

    <p class="hint">
      没有账号？
      <router-link to="/register">去注册</router-link>
    </p>
  </section>
</template>

<style scoped>
.card {
  max-width: 520px;
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
.form {
  display: grid;
  gap: 12px;
}
.label {
  font-size: 13px;
  opacity: 0.85;
  margin-bottom: 6px;
}
input {
  width: 100%;
  padding: 10px 12px;
  border-radius: 12px;
  border: 1px solid rgba(255, 255, 255, 0.14);
  background: rgba(0, 0, 0, 0.25);
  color: #eef0ff;
  outline: none;
}
.btn {
  padding: 10px 12px;
  border-radius: 12px;
  border: 1px solid rgba(255, 255, 255, 0.16);
  background: rgba(169, 182, 255, 0.18);
  color: #eef0ff;
  cursor: pointer;
}
.btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}
.error {
  padding: 10px 12px;
  border-radius: 12px;
  border: 1px solid rgba(255, 120, 120, 0.35);
  background: rgba(255, 120, 120, 0.12);
}
.hint {
  margin: 12px 0 0;
  opacity: 0.9;
}
</style>


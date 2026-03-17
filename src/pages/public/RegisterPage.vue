<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { useAuthStore } from '@/stores/auth'
import { api } from '@/api/client'

const auth = useAuthStore()
const router = useRouter()

const name = ref('')
const email = ref('')
const password = ref('')
const verificationCode = ref('')
const loading = ref(false)
const error = ref('')
const sendingCode = ref(false)
const codeCountdown = ref(0)

async function onSendCode() {
  if (!email.value || sendingCode.value || codeCountdown.value > 0) return
  error.value = ''
  sendingCode.value = true
  try {
    await api.sendRegisterEmailCode({ email: email.value, scene: 'register' })
    codeCountdown.value = 60
    const timer = setInterval(() => {
      if (codeCountdown.value <= 1) {
        codeCountdown.value = 0
        clearInterval(timer)
      } else {
        codeCountdown.value -= 1
      }
    }, 1000)
  } catch (e) {
    error.value = e?.message || e?.data?.message || '验证码发送失败'
  } finally {
    sendingCode.value = false
  }
}

async function onSubmit() {
  error.value = ''
  loading.value = true
  try {
    await auth.register({
      name: name.value,
      email: email.value,
      password: password.value,
      verificationCode: verificationCode.value
    })
    await router.replace(auth.defaultHomeRoute())
  } catch (e) {
    error.value = e?.message || '注册失败'
  } finally {
    loading.value = false
  }
}
</script>

<template>
  <section class="card">
    <h1>注册</h1>
    <p class="muted">创建客户账号后即可浏览专家并预约。</p>

    <form class="form" @submit.prevent="onSubmit">
      <label>
        <div class="label">姓名</div>
        <input v-model="name" type="text" autocomplete="name" required />
      </label>

      <label>
        <div class="label">邮箱</div>
        <input v-model="email" type="email" autocomplete="email" required />
      </label>

      <label>
        <div class="label">邮箱验证码</div>
        <div class="code-row">
          <input
            v-model="verificationCode"
            type="text"
            inputmode="numeric"
            maxlength="6"
            placeholder="请输入邮箱收到的验证码"
            required
          />
          <button
            type="button"
            class="btn-secondary"
            :disabled="sendingCode || !email || codeCountdown > 0"
            @click="onSendCode"
          >
            {{ codeCountdown > 0 ? `重新发送 (${codeCountdown}s)` : sendingCode ? '发送中...' : '发送验证码' }}
          </button>
        </div>
      </label>

      <label>
        <div class="label">密码</div>
        <input v-model="password" type="password" autocomplete="new-password" required />
      </label>

      <div v-if="error" class="error">{{ error }}</div>

      <button class="btn" type="submit" :disabled="loading">
        {{ loading ? '注册中...' : '注册' }}
      </button>
    </form>

    <p class="hint">
      已有账号？
      <router-link to="/login">去登录</router-link>
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
.code-row {
  display: grid;
  grid-template-columns: 1fr auto;
  gap: 8px;
  align-items: center;
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
.btn-secondary {
  padding: 8px 10px;
  border-radius: 10px;
  border: 1px solid rgba(255, 255, 255, 0.3);
  background: transparent;
  color: #eef0ff;
  white-space: nowrap;
  cursor: pointer;
  font-size: 13px;
}
.btn-secondary:disabled {
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


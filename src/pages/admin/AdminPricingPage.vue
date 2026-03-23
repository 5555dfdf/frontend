<script setup>
import { ref } from 'vue'
import { api } from '@/api/client'

const specialistId = ref('')
const duration = ref(60)
const type = ref('online')
const loading = ref(false)
const error = ref('')
const quote = ref(null)

async function onQuote() {
  error.value = ''
  quote.value = null
  if (!specialistId.value.trim()) {
    error.value = '请填写 specialistId'
    return
  }
  loading.value = true
  try {
    quote.value = await api.quotePricing({
      specialistId: specialistId.value.trim(),
      duration: duration.value ? Number(duration.value) : undefined,
      type: type.value.trim() || undefined
    })
  } catch (e) {
    error.value = e?.message || '报价失败'
  } finally {
    loading.value = false
  }
}
</script>

<template>
  <section class="page">
    <header class="page__header">
      <h1>定价规则</h1>
      <p class="muted">POST /pricing/quote</p>
    </header>

    <div class="card">
      <label class="field">
        <span class="label">专家 ID</span>
        <input v-model="specialistId" class="input" placeholder="sp-1" />
      </label>
      <label class="field">
        <span class="label">时长（分钟）</span>
        <input v-model.number="duration" class="input" type="number" min="0" step="15" />
      </label>
      <label class="field">
        <span class="label">类型</span>
        <input v-model="type" class="input" placeholder="online" />
      </label>
      <button type="button" class="btn" :disabled="loading" @click="onQuote">
        {{ loading ? '计算中…' : '获取报价' }}
      </button>
    </div>

    <div v-if="error" class="banner banner--error" role="alert">{{ error }}</div>

    <div v-if="quote" class="card result">
      <div class="title">报价结果</div>
      <pre class="pre">{{ JSON.stringify(quote, null, 2) }}</pre>
    </div>
  </section>
</template>

<style scoped>
.page__header h1 {
  margin: 0 0 6px;
  font-size: 22px;
}
.muted {
  opacity: 0.8;
}
.card {
  margin-top: 14px;
  padding: 14px;
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 14px;
  background: rgba(255, 255, 255, 0.04);
}
.field {
  display: grid;
  gap: 6px;
  margin-bottom: 10px;
  max-width: 400px;
}
.label {
  font-size: 13px;
  opacity: 0.85;
}
.input {
  padding: 10px 12px;
  border-radius: 12px;
  border: 1px solid rgba(255, 255, 255, 0.14);
  background: #ffffff;
  color: #111827;
}
.btn {
  padding: 10px 18px;
  border-radius: 10px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  background: rgba(255, 255, 255, 0.12);
  color: inherit;
  cursor: pointer;
}
.btn:disabled {
  opacity: 0.5;
}
.title {
  font-weight: 700;
  margin-bottom: 8px;
}
.pre {
  margin: 0;
  font-size: 13px;
  overflow: auto;
  line-height: 1.45;
}
.banner {
  margin-top: 12px;
  padding: 10px 14px;
  border-radius: 12px;
}
.banner--error {
  border: 1px solid rgba(248, 113, 113, 0.45);
  background: rgba(248, 113, 113, 0.12);
}
</style>

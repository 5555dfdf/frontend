<script setup>
import { ref, watch } from 'vue'
import { useRouter } from 'vue-router'
import { api } from '@/api/client'

const props = defineProps({
  id: { type: String, required: true }
})

const router = useRouter()
const booking = ref(null)
const loading = ref(false)
const error = ref('')
const rejectReason = ref('')
const busy = ref('')

async function load() {
  error.value = ''
  loading.value = true
  booking.value = null
  try {
    booking.value = await api.getBooking(props.id)
  } catch (e) {
    error.value = e?.message || '加载失败'
  } finally {
    loading.value = false
  }
}

watch(
  () => props.id,
  () => load(),
  { immediate: true }
)

async function run(action) {
  busy.value = action
  try {
    if (action === 'confirm') booking.value = await api.confirmBooking(props.id)
    else if (action === 'reject')
      booking.value = await api.rejectBooking(props.id, {
        reason: rejectReason.value.trim() || undefined
      })
    else if (action === 'complete') booking.value = await api.completeBooking(props.id)
  } catch (e) {
    error.value = e?.message || '操作失败'
  } finally {
    busy.value = ''
  }
}
</script>

<template>
  <section class="page">
    <header class="page__header">
      <h1>预约详情（专家）</h1>
      <p class="muted mono">bookingId: {{ id }}</p>
    </header>

    <div v-if="error" class="banner banner--error" role="alert">{{ error }}</div>
    <div v-else-if="loading" class="card muted">加载中…</div>

    <template v-else-if="booking">
      <div class="card">
        <div class="title">预约信息</div>
        <dl class="kv">
          <dt>状态</dt>
          <dd>{{ booking.status ?? '—' }}</dd>
          <dt>时间</dt>
          <dd>{{ booking.time ?? booking.startTime ?? '—' }}</dd>
          <dt>客户</dt>
          <dd>{{ booking.customerName ?? booking.customerId ?? '—' }}</dd>
          <dt>时段</dt>
          <dd class="mono">{{ booking.slotId ?? '—' }}</dd>
        </dl>
      </div>

      <div class="card">
        <div class="title">操作</div>
        <label class="field">
          <span class="label">拒绝原因（可选）</span>
          <input v-model="rejectReason" class="input" />
        </label>
        <div class="btns">
          <button
            type="button"
            class="btn btn--ok"
            :disabled="!!busy"
            @click="run('confirm')"
          >
            {{ busy === 'confirm' ? '…' : '确认' }}
          </button>
          <button
            type="button"
            class="btn btn--danger"
            :disabled="!!busy"
            @click="run('reject')"
          >
            {{ busy === 'reject' ? '…' : '拒绝' }}
          </button>
          <button type="button" class="btn" :disabled="!!busy" @click="run('complete')">
            {{ busy === 'complete' ? '…' : '完成' }}
          </button>
        </div>
      </div>

      <p class="muted small">
        <button type="button" class="linkish" @click="router.push({ name: 'specialist.requests' })">
          返回请求列表
        </button>
      </p>
    </template>
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
.small {
  font-size: 12px;
  margin-top: 12px;
}
.mono {
  font-family: ui-monospace, monospace;
  font-size: 12px;
}
.card {
  margin-top: 14px;
  padding: 14px;
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 14px;
  background: rgba(255, 255, 255, 0.04);
}
.title {
  font-weight: 700;
  margin-bottom: 10px;
}
.kv {
  display: grid;
  grid-template-columns: 72px 1fr;
  gap: 8px 12px;
  margin: 0;
  font-size: 14px;
}
.kv dt {
  opacity: 0.75;
  margin: 0;
}
.kv dd {
  margin: 0;
}
.field {
  display: grid;
  gap: 6px;
  margin-bottom: 12px;
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
.btns {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}
.btn {
  padding: 10px 16px;
  border-radius: 10px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  background: rgba(255, 255, 255, 0.1);
  color: inherit;
  cursor: pointer;
}
.btn--ok {
  border-color: rgba(52, 211, 153, 0.45);
  background: rgba(52, 211, 153, 0.12);
}
.btn--danger {
  border-color: rgba(248, 113, 113, 0.45);
  background: rgba(248, 113, 113, 0.12);
}
.btn:disabled {
  opacity: 0.5;
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
.linkish {
  background: none;
  border: none;
  padding: 0;
  color: inherit;
  text-decoration: underline;
  cursor: pointer;
  font: inherit;
}
</style>

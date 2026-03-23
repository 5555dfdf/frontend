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
const cancelReason = ref('')
const newSlotId = ref('')
const busy = ref('')
const actionError = ref('')

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

async function onCancel() {
  actionError.value = ''
  busy.value = 'cancel'
  try {
    booking.value = await api.cancelBooking(props.id, {
      reason: cancelReason.value.trim() || undefined
    })
  } catch (e) {
    actionError.value = e?.message || '取消失败'
  } finally {
    busy.value = ''
  }
}

async function onReschedule() {
  actionError.value = ''
  if (!newSlotId.value.trim()) {
    actionError.value = '请填写新时段 slotId'
    return
  }
  busy.value = 'reschedule'
  try {
    booking.value = await api.rescheduleBooking(props.id, { slotId: newSlotId.value.trim() })
  } catch (e) {
    actionError.value = e?.message || '改期失败'
  } finally {
    busy.value = ''
  }
}
</script>

<template>
  <section class="page">
    <header class="page__header">
      <h1>预约详情</h1>
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
          <dt>专家</dt>
          <dd class="mono">{{ booking.specialistId ?? '—' }}</dd>
          <dt>时段</dt>
          <dd class="mono">{{ booking.slotId ?? '—' }}</dd>
          <dt>备注</dt>
          <dd>{{ booking.note ?? '—' }}</dd>
        </dl>
      </div>

      <div class="card">
        <div class="title">取消预约</div>
        <label class="field">
          <span class="label">原因（可选）</span>
          <input v-model="cancelReason" class="input" placeholder="原因" />
        </label>
        <button
          type="button"
          class="btn btn--danger"
          :disabled="busy === 'cancel'"
          @click="onCancel"
        >
          {{ busy === 'cancel' ? '处理中…' : '取消预约' }}
        </button>
      </div>

      <div class="card">
        <div class="title">改期</div>
        <label class="field">
          <span class="label">新 slotId</span>
          <input v-model="newSlotId" class="input" placeholder="从专家时段列表获取" />
        </label>
        <button
          type="button"
          class="btn"
          :disabled="busy === 'reschedule'"
          @click="onReschedule"
        >
          {{ busy === 'reschedule' ? '处理中…' : '提交改期' }}
        </button>
      </div>

      <p v-if="actionError" class="banner banner--error">{{ actionError }}</p>

      <p class="muted small">
        <button type="button" class="linkish" @click="router.push({ name: 'customer.bookings' })">
          返回我的预约
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
  grid-template-columns: 88px 1fr;
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
  margin-bottom: 10px;
}
.label {
  font-size: 13px;
  opacity: 0.85;
}
.input {
  width: 100%;
  max-width: 400px;
  padding: 10px 12px;
  border-radius: 12px;
  border: 1px solid rgba(255, 255, 255, 0.14);
  background: #ffffff;
  color: #111827;
}
.btn {
  padding: 10px 16px;
  border-radius: 10px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  background: rgba(255, 255, 255, 0.1);
  color: inherit;
  cursor: pointer;
}
.btn--danger {
  border-color: rgba(248, 113, 113, 0.5);
  background: rgba(248, 113, 113, 0.15);
}
.btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
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

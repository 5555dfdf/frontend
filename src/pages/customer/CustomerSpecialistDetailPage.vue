<script setup>
import { computed, ref, watch } from 'vue'
import { useRouter } from 'vue-router'
import { api } from '@/api/client'

const props = defineProps({
  id: { type: String, required: true }
})

const router = useRouter()
const specialist = ref(null)
const slots = ref([])
const slotDate = ref(new Date().toISOString().slice(0, 10))
const selectedSlotId = ref('')
const note = ref('')
const loading = ref(false)
const slotsLoading = ref(false)
const bookingLoading = ref(false)
const error = ref('')
const actionMsg = ref('')

const expertiseLabel = computed(() => {
  const ex = specialist.value?.expertise
  if (Array.isArray(ex) && ex.length) return ex.map((e) => e.name ?? e.id).join('、')
  return '—'
})

async function loadSpecialist() {
  error.value = ''
  loading.value = true
  specialist.value = null
  try {
    specialist.value = await api.getSpecialist(props.id)
  } catch (e) {
    error.value = e?.message || '加载专家失败'
  } finally {
    loading.value = false
  }
}

async function loadSlots() {
  if (!props.id) return
  slotsLoading.value = true
  actionMsg.value = ''
  try {
    slots.value = await api.listSpecialistSlots(props.id, { date: slotDate.value })
    selectedSlotId.value = ''
  } catch (e) {
    slots.value = []
    actionMsg.value = e?.message || '加载时段失败'
  } finally {
    slotsLoading.value = false
  }
}

watch(
  () => props.id,
  () => {
    loadSpecialist()
    loadSlots()
  },
  { immediate: true }
)

watch(slotDate, () => loadSlots())

async function onBook() {
  actionMsg.value = ''
  if (!selectedSlotId.value) {
    actionMsg.value = '请选择时段'
    return
  }
  bookingLoading.value = true
  try {
    const created = await api.createBooking({
      specialistId: props.id,
      slotId: selectedSlotId.value,
      note: note.value.trim() || undefined
    })
    const bid = created?.id
    if (bid) await router.push({ name: 'customer.bookingDetail', params: { id: bid } })
    else await router.push({ name: 'customer.bookings' })
  } catch (e) {
    actionMsg.value = e?.message || '预约失败'
  } finally {
    bookingLoading.value = false
  }
}
</script>

<template>
  <section class="page">
    <header class="page__header">
      <h1>专家详情</h1>
      <p class="muted mono">specialistId: {{ id }}</p>
    </header>

    <div v-if="error" class="banner banner--error" role="alert">{{ error }}</div>
    <div v-else-if="loading" class="card muted">加载中…</div>

    <template v-else-if="specialist">
      <div class="card">
        <div class="title">{{ specialist.name ?? '—' }}</div>
        <p class="bio">{{ specialist.bio ?? '暂无简介' }}</p>
        <p class="muted small">专长：{{ expertiseLabel }}</p>
        <p v-if="specialist.price != null" class="muted small">参考价：{{ specialist.price }}</p>
      </div>

      <div class="card">
        <div class="title">可预约时段</div>
        <label class="field">
          <span class="label">日期</span>
          <input v-model="slotDate" type="date" class="input" />
        </label>
        <p v-if="slotsLoading" class="muted small">加载时段中…</p>
        <ul v-else-if="slots.length" class="slots">
          <li v-for="sl in slots" :key="sl.slotId ?? sl.id" class="slot-row">
            <label class="pick">
              <input
                v-model="selectedSlotId"
                type="radio"
                name="slot"
                :value="sl.slotId ?? sl.id"
                :disabled="sl.available === false"
              />
              <span>{{ sl.start ?? sl.startTime }} — {{ sl.end ?? sl.endTime }}</span>
              <span v-if="sl.available === false" class="muted small">（已满）</span>
            </label>
          </li>
        </ul>
        <p v-else class="muted small">当日暂无可用时段。</p>
      </div>

      <div class="card">
        <div class="title">创建预约</div>
        <label class="field">
          <span class="label">备注（可选）</span>
          <textarea v-model="note" class="textarea" rows="3" placeholder="希望沟通的主题等" />
        </label>
        <p v-if="actionMsg" class="action-msg">{{ actionMsg }}</p>
        <button type="button" class="btn" :disabled="bookingLoading" @click="onBook">
          {{ bookingLoading ? '提交中…' : '提交预约' }}
        </button>
      </div>
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
}
.mono {
  font-family: ui-monospace, monospace;
  font-size: 13px;
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
  margin-bottom: 8px;
}
.bio {
  margin: 0 0 8px;
  line-height: 1.5;
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
.input,
.textarea {
  width: 100%;
  padding: 10px 12px;
  border-radius: 12px;
  border: 1px solid rgba(255, 255, 255, 0.14);
  background: #ffffff;
  color: #111827;
  outline: none;
}
.slots {
  list-style: none;
  padding: 0;
  margin: 8px 0 0;
  display: grid;
  gap: 6px;
}
.pick {
  display: flex;
  align-items: center;
  gap: 8px;
  flex-wrap: wrap;
  cursor: pointer;
}
.btn {
  margin-top: 8px;
  padding: 10px 18px;
  border-radius: 10px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  background: rgba(255, 255, 255, 0.12);
  color: inherit;
  cursor: pointer;
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
.action-msg {
  font-size: 13px;
  color: #fecaca;
  margin: 0 0 8px;
}
</style>

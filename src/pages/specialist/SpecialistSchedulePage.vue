<script setup>
import { computed, ref, watch } from 'vue'
import { useAuthStore } from '@/stores/auth'
import { api } from '@/api/client'

const auth = useAuthStore()
const specialistId = ref('')
const slotDate = ref(new Date().toISOString().slice(0, 10))
const slots = ref([])
const loading = ref(false)
const error = ref('')

const hintId = computed(() => auth.user?.specialistId ?? auth.user?.id ?? '')

watch(
  hintId,
  (v) => {
    if (v && !specialistId.value) specialistId.value = String(v)
  },
  { immediate: true }
)

async function loadSlots() {
  if (!specialistId.value.trim()) {
    slots.value = []
    return
  }
  error.value = ''
  loading.value = true
  try {
    slots.value = await api.listSpecialistSlots(specialistId.value.trim(), { date: slotDate.value })
  } catch (e) {
    error.value = e?.message || '加载时段失败'
    slots.value = []
  } finally {
    loading.value = false
  }
}

watch([specialistId, slotDate], () => loadSlots())
</script>

<template>
  <section class="page">
    <header class="page__header">
      <h1>我的日程</h1>
      <p class="muted">只读展示 GET /specialists/:id/slots；维护时段若后端提供专用接口可再接入。</p>
    </header>

    <div class="card">
      <div class="title">查询条件</div>
      <label class="field">
        <span class="label">专家 ID（可与登录用户 specialistId 对齐）</span>
        <input v-model="specialistId" class="input" placeholder="sp-1" />
      </label>
      <label class="field">
        <span class="label">日期</span>
        <input v-model="slotDate" type="date" class="input" />
      </label>
      <button type="button" class="btn" :disabled="loading" @click="loadSlots">刷新</button>
    </div>

    <div v-if="error" class="banner banner--error" role="alert">{{ error }}</div>

    <div class="card">
      <div class="title">可用时段</div>
      <p v-if="loading" class="muted">加载中…</p>
      <ul v-else-if="slots.length" class="slots">
        <li v-for="sl in slots" :key="sl.slotId ?? sl.id" class="slot">
          <span>{{ sl.start ?? sl.startTime }}</span>
          <span>—</span>
          <span>{{ sl.end ?? sl.endTime }}</span>
          <span v-if="sl.available === false" class="muted small">已满</span>
        </li>
      </ul>
      <p v-else class="muted small">暂无数据；请填写专家 ID 并选择日期。</p>
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
.small {
  font-size: 12px;
  margin-left: 8px;
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
.field {
  display: grid;
  gap: 6px;
  margin-bottom: 10px;
  max-width: 420px;
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
  padding: 10px 16px;
  border-radius: 10px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  background: rgba(255, 255, 255, 0.1);
  color: inherit;
  cursor: pointer;
}
.btn:disabled {
  opacity: 0.5;
}
.slots {
  list-style: none;
  padding: 0;
  margin: 0;
  display: grid;
  gap: 6px;
}
.slot {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 8px;
  padding: 8px 10px;
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.05);
  font-size: 14px;
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

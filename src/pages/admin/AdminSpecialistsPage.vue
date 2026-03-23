<script setup>
import { onMounted, ref } from 'vue'
import { api } from '@/api/client'

const expertiseList = ref([])
const page = ref({ items: [], total: 0 })
const loading = ref(false)
const error = ref('')

const form = ref({
  name: '',
  price: '',
  bio: '',
  expertiseIds: []
})
const creating = ref('')
const statusBusy = ref('')

async function loadExpertise() {
  try {
    expertiseList.value = await api.listExpertise()
  } catch {
    expertiseList.value = []
  }
}

async function loadSpecialists() {
  error.value = ''
  loading.value = true
  try {
    page.value = await api.listSpecialists({ pageSize: 100 })
  } catch (e) {
    error.value = e?.message || '加载专家失败'
    page.value = { items: [], total: 0 }
  } finally {
    loading.value = false
  }
}

onMounted(async () => {
  await loadExpertise()
  await loadSpecialists()
})

function toggleExpertise(id) {
  const set = new Set(form.value.expertiseIds)
  if (set.has(id)) set.delete(id)
  else set.add(id)
  form.value.expertiseIds = [...set]
}

async function onCreate() {
  error.value = ''
  if (!form.value.name.trim()) {
    error.value = '请填写姓名'
    return
  }
  if (!form.value.expertiseIds.length) {
    error.value = '请至少选择一个专长'
    return
  }
  creating.value = 'yes'
  try {
    const price = form.value.price === '' ? undefined : Number(form.value.price)
    await api.adminCreateSpecialist({
      name: form.value.name.trim(),
      expertiseIds: form.value.expertiseIds,
      price: Number.isFinite(price) ? price : undefined,
      bio: form.value.bio.trim() || undefined
    })
    form.value = { name: '', price: '', bio: '', expertiseIds: [] }
    await loadSpecialists()
  } catch (e) {
    error.value = e?.message || '创建失败'
  } finally {
    creating.value = ''
  }
}

async function setStatus(id, status) {
  statusBusy.value = id + status
  try {
    await api.adminSetSpecialistStatus(id, { status })
    await loadSpecialists()
  } catch (e) {
    error.value = e?.message || '更新状态失败'
  } finally {
    statusBusy.value = ''
  }
}
</script>

<template>
  <section class="page">
    <header class="page__header">
      <h1>专家管理</h1>
      <p class="muted">GET /specialists 列表；POST/PATCH/状态 见表单与操作列。</p>
    </header>

    <div v-if="error" class="banner banner--error" role="alert">{{ error }}</div>

    <div class="card">
      <div class="title">新建专家</div>
      <label class="field">
        <span class="label">姓名</span>
        <input v-model="form.name" class="input" />
      </label>
      <label class="field">
        <span class="label">默认价格（可选）</span>
        <input v-model="form.price" class="input" type="number" min="0" step="1" />
      </label>
      <label class="field">
        <span class="label">简介（可选）</span>
        <textarea v-model="form.bio" class="textarea" rows="2" />
      </label>
      <div class="label">专长（多选）</div>
      <div class="chips">
        <label v-for="e in expertiseList" :key="e.id" class="chip">
          <input
            type="checkbox"
            :checked="form.expertiseIds.includes(e.id)"
            @change="toggleExpertise(e.id)"
          />
          {{ e.name }}
        </label>
      </div>
      <button type="button" class="btn" :disabled="!!creating" @click="onCreate">
        {{ creating ? '创建中…' : '创建' }}
      </button>
    </div>

    <div class="toolbar">
      <span class="title">专家列表</span>
      <button type="button" class="btn btn--ghost" :disabled="loading" @click="loadSpecialists">
        刷新
      </button>
    </div>

    <div v-if="loading && !(page.items || []).length" class="card muted">加载中…</div>

    <div v-else class="table-wrap card">
      <table class="table">
        <thead>
          <tr>
            <th>ID</th>
            <th>姓名</th>
            <th>价格</th>
            <th>操作</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="s in page.items || []" :key="s.id">
            <td class="mono">{{ s.id }}</td>
            <td>{{ s.name }}</td>
            <td>{{ s.price ?? '—' }}</td>
            <td class="actions">
              <button
                type="button"
                class="btn-sm"
                :disabled="statusBusy === s.id + 'Active'"
                @click="setStatus(s.id, 'Active')"
              >
                上架
              </button>
              <button
                type="button"
                class="btn-sm btn-sm--danger"
                :disabled="statusBusy === s.id + 'Inactive'"
                @click="setStatus(s.id, 'Inactive')"
              >
                下架
              </button>
            </td>
          </tr>
        </tbody>
      </table>
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
.title {
  font-weight: 700;
  margin-bottom: 10px;
}
.toolbar {
  margin-top: 16px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
}
.field {
  display: grid;
  gap: 6px;
  margin-bottom: 10px;
  max-width: 480px;
}
.label {
  font-size: 13px;
  opacity: 0.85;
}
.input,
.textarea {
  padding: 10px 12px;
  border-radius: 12px;
  border: 1px solid rgba(255, 255, 255, 0.14);
  background: #ffffff;
  color: #111827;
}
.chips {
  display: flex;
  flex-wrap: wrap;
  gap: 8px 14px;
  margin: 8px 0 14px;
}
.chip {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 13px;
  cursor: pointer;
}
.btn {
  padding: 10px 18px;
  border-radius: 10px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  background: rgba(255, 255, 255, 0.12);
  color: inherit;
  cursor: pointer;
}
.btn--ghost {
  background: rgba(255, 255, 255, 0.06);
}
.btn:disabled {
  opacity: 0.5;
}
.table-wrap {
  overflow-x: auto;
  padding: 0;
}
.table {
  width: 100%;
  border-collapse: collapse;
  font-size: 14px;
}
.table th,
.table td {
  padding: 10px 12px;
  text-align: left;
  border-bottom: 1px solid rgba(255, 255, 255, 0.08);
}
.mono {
  font-family: ui-monospace, monospace;
  font-size: 12px;
}
.actions {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
}
.btn-sm {
  padding: 6px 10px;
  font-size: 12px;
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  background: rgba(255, 255, 255, 0.08);
  color: inherit;
  cursor: pointer;
}
.btn-sm--danger {
  border-color: rgba(248, 113, 113, 0.45);
}
.btn-sm:disabled {
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
</style>

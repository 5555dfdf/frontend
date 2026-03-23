<script setup>
import { onMounted, ref } from 'vue'
import { api } from '@/api/client'

const items = ref([])
const loading = ref(false)
const error = ref('')
const createName = ref('')
const createDesc = ref('')
const creating = ref(false)

async function load() {
  error.value = ''
  loading.value = true
  try {
    items.value = await api.listExpertise()
  } catch (e) {
    error.value = e?.message || '加载失败'
    items.value = []
  } finally {
    loading.value = false
  }
}

async function onCreate() {
  if (!createName.value.trim()) {
    error.value = '请填写专长名称'
    return
  }
  creating.value = true
  error.value = ''
  try {
    await api.adminCreateExpertise({
      name: createName.value.trim(),
      description: createDesc.value.trim() || undefined
    })
    createName.value = ''
    createDesc.value = ''
    await load()
  } catch (e) {
    error.value = e?.message || '创建失败'
  } finally {
    creating.value = false
  }
}

onMounted(load)
</script>

<template>
  <section class="page">
    <header class="page__header">
      <h1>专长类目管理</h1>
      <p class="muted">GET /expertise 列表；下方表单调用 POST /admin/expertise。</p>
    </header>

    <div class="card create">
      <div class="title">新建专长</div>
      <label class="field">
        <span class="label">名称</span>
        <input v-model="createName" class="input" />
      </label>
      <label class="field">
        <span class="label">描述（可选）</span>
        <input v-model="createDesc" class="input" />
      </label>
      <button type="button" class="btn" :disabled="creating" @click="onCreate">
        {{ creating ? '创建中…' : '创建' }}
      </button>
    </div>

    <div class="toolbar card">
      <div class="toolbar__left">
        <span class="title">专长列表</span>
        <span class="muted small">共 {{ items.length }} 条</span>
      </div>
      <button type="button" class="btn" :disabled="loading" @click="load">
        {{ loading ? '加载中…' : '刷新' }}
      </button>
    </div>

    <div v-if="error" class="banner banner--error" role="alert">{{ error }}</div>

    <div v-if="loading && !items.length" class="card muted">加载中…</div>

    <div v-else-if="!items.length && !error" class="empty">
      <div class="empty__title">暂无专长数据</div>
      <p class="muted">后端返回空列表，或尚未配置专长。</p>
    </div>

    <div v-else class="table-wrap card">
      <table class="table">
        <thead>
          <tr>
            <th scope="col">ID</th>
            <th scope="col">名称</th>
            <th scope="col">描述</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="row in items" :key="row.id ?? row.name">
            <td class="mono">{{ row.id ?? '—' }}</td>
            <td>{{ row.name ?? '—' }}</td>
            <td class="desc">{{ row.description ?? row.desc ?? '—' }}</td>
          </tr>
        </tbody>
      </table>
    </div>

    <div class="card foot">
      <div class="title">相关接口</div>
      <div class="muted small">GET /expertise</div>
      <div class="muted small">POST /admin/expertise</div>
      <div class="muted small">PATCH /admin/expertise/:id</div>
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
}
.create .field {
  display: grid;
  gap: 6px;
  margin-bottom: 10px;
  max-width: 400px;
}
.create .label {
  font-size: 13px;
  opacity: 0.85;
}
.create .input {
  padding: 10px 12px;
  border-radius: 12px;
  border: 1px solid rgba(255, 255, 255, 0.14);
  background: #ffffff;
  color: #111827;
}
.create .btn {
  padding: 8px 16px;
  border-radius: 10px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  background: rgba(255, 255, 255, 0.1);
  color: inherit;
  cursor: pointer;
}
.create .btn:disabled {
  opacity: 0.5;
}
.toolbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
}
.toolbar__left {
  display: flex;
  flex-direction: column;
  gap: 4px;
}
.btn {
  padding: 8px 14px;
  border-radius: 10px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  background: rgba(255, 255, 255, 0.08);
  color: inherit;
  font-size: 13px;
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
  font-size: 14px;
}
.banner--error {
  border: 1px solid rgba(248, 113, 113, 0.45);
  background: rgba(248, 113, 113, 0.12);
}
.empty {
  margin-top: 16px;
  padding: 18px;
  border: 1px dashed rgba(255, 255, 255, 0.16);
  border-radius: 14px;
}
.empty__title {
  font-weight: 700;
  margin-bottom: 6px;
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
  padding: 10px 14px;
  text-align: left;
  border-bottom: 1px solid rgba(255, 255, 255, 0.08);
}
.table th {
  font-weight: 600;
  opacity: 0.9;
  white-space: nowrap;
}
.table tbody tr:last-child td {
  border-bottom: none;
}
.mono {
  font-family: ui-monospace, monospace;
  font-size: 12px;
  opacity: 0.9;
}
.desc {
  max-width: 420px;
  line-height: 1.45;
}
.foot .title {
  margin-bottom: 8px;
}
.foot .small + .small {
  margin-top: 4px;
}
</style>

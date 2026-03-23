<script setup>
import { onMounted, ref } from 'vue'
import { RouterLink } from 'vue-router'
import { api } from '@/api/client'

const expertiseCount = ref(0)
const specialistTotal = ref(0)
const loading = ref(false)
const error = ref('')

async function load() {
  error.value = ''
  loading.value = true
  try {
    const [ex, sp] = await Promise.all([api.listExpertise(), api.listSpecialists({ pageSize: 1 })])
    expertiseCount.value = ex.length
    specialistTotal.value = sp.total ?? (sp.items || []).length
  } catch (e) {
    error.value = e?.message || '加载失败'
  } finally {
    loading.value = false
  }
}

onMounted(load)
</script>

<template>
  <section class="page">
    <header class="page__header">
      <h1>管理员概览</h1>
      <p class="muted">聚合 GET /expertise 与 GET /specialists</p>
    </header>

    <div v-if="error" class="banner banner--error" role="alert">{{ error }}</div>

    <div class="grid">
      <div class="card">
        <div class="label">专长类目</div>
        <div class="num">{{ loading ? '…' : expertiseCount }}</div>
        <RouterLink class="link" :to="{ name: 'admin.expertise' }">管理</RouterLink>
      </div>
      <div class="card">
        <div class="label">专家（约）</div>
        <div class="num">{{ loading ? '…' : specialistTotal }}</div>
        <RouterLink class="link" :to="{ name: 'admin.specialists' }">管理</RouterLink>
      </div>
    </div>

    <div class="card links">
      <div class="title">快速入口</div>
      <RouterLink :to="{ name: 'admin.slots' }">时段</RouterLink>
      <RouterLink :to="{ name: 'admin.pricing' }">定价试算</RouterLink>
      <RouterLink :to="{ name: 'admin.bookings' }">预约</RouterLink>
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
.grid {
  margin-top: 14px;
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 12px;
}
.card {
  padding: 14px;
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 14px;
  background: rgba(255, 255, 255, 0.04);
}
.label {
  font-size: 13px;
  opacity: 0.85;
}
.num {
  font-size: 32px;
  font-weight: 800;
  margin: 8px 0;
}
.link {
  color: inherit;
  font-weight: 600;
  text-decoration: underline;
  text-underline-offset: 3px;
}
.links {
  margin-top: 14px;
  display: flex;
  flex-wrap: wrap;
  gap: 12px 18px;
}
.title {
  width: 100%;
  font-weight: 700;
  margin-bottom: 4px;
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

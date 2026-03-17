<template>
  <div class="public">
    <header class="public__header">
      <div class="brand">Consultancy Booking</div>
      <nav class="nav">
        <router-link to="/login">登录</router-link>
        <router-link to="/register">注册</router-link>
      </nav>
    </header>
    <main class="public__main">
      <router-view />
    </main>
  </div>
</template>

<style scoped>
.public {
  min-height: 100vh;
  background: #0b1020;
  color: #e8ecff;
}
.public__header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px 20px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.08);
}
.brand {
  font-weight: 700;
  letter-spacing: 0.4px;
}
.nav {
  display: flex;
  gap: 12px;
}
.public__main {
  max-width: 980px;
  margin: 0 auto;
  padding: 28px 20px 60px;
}
a {
  color: #a9b6ff;
  text-decoration: none;
}
a.router-link-active {
  color: #ffffff;
  font-weight: 600;
}
</style>


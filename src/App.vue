<script setup>
import { RouterView } from 'vue-router'
import { Layout } from 'ant-design-vue'
</script>

<template>
  <a-layout style="min-height: 100vh; width: 100vw">
    <a-layout style="height: 100vh">
      <a-layout-header class="header" style="background: #fff; padding: 0 24px; display: flex; justify-content: space-between; align-items: center; box-shadow: 0 1px 4px rgba(0,21,41,.08)">
        <div class="logo" style="font-size: 20px; font-weight: bold">AI 助手</div>
      </a-layout-header>
      <a-layout-content class="chat-content">
        <router-view></router-view>
      </a-layout-content>
    </a-layout>
  </a-layout>
</template>

<style scoped>
.header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 24px;
  background: #fff;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  height: 64px;
  line-height: 64px;
  position: sticky;
  top: 0;
  z-index: 1;
}

.logo {
  color: #1890ff;
  font-size: 18px;
  font-weight: bold;
  transition: color 0.3s;
}

.logo:hover {
  color: #40a9ff;
}

.chat-content {
  padding: 24px;
  display: flex;
  flex-direction: column;
  height: calc(100vh - 64px);
  margin: 0;
  width: 100%;
}
</style>

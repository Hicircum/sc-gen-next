<script setup lang="ts">
import ThemeSwitch from './components/ThemeSwitch.vue'
import GeneratorPanel from './components/GeneratorPanel.vue';
</script>

<template>
  <div class="header max-container">
    <div>SC生成器 <el-tag type="primary" style="margin-left: 5px;margin-bottom: 3px;">Next</el-tag></div>
    <ThemeSwitch />
  </div>
  <div class="main max-container">
    <GeneratorPanel />
  </div>
  <div class="footer max-container">
    <a href="https://space.bilibili.com/347687924">Bug反馈&功能建议 私信</a>
  </div>
</template>

<style scoped lang="scss">
.header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 1.1rem;
  box-sizing: border-box;
  padding: 10px 20px;
  user-select: none;
}
.main {
  box-sizing: border-box;
  margin-top: 20px;
}
.footer {
  text-align: center;
  padding: 10px 0 30px 0;
  font-size: 0.9rem;
  a {
    color: inherit;
    text-decoration: none;
  }
}
.max-container {
  margin: 0 auto;
  width: 100%;
  max-width: 1100px;
}
@media screen and (max-width: 1920px) {
  .max-container {
    max-width: 1100px;
  }
}
@media screen and (max-width: 1280px) {
  .max-container {
    max-width: 1100px;
  }
}
@media screen and (max-width: 992px) {
  .max-container {
    max-width: 960px;
  }
}
@media screen and (max-width: 768px) {
  .max-container {
    max-width: 720px;
  }
}
@media screen and (max-width: 576px) {
  .max-container {
    max-width: 540px;
  }
}
</style>

<style>
body {
  padding: 0;
  margin: 0;
}
</style>
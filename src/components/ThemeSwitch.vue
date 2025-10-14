<script setup lang="ts">
import { Sunny, MostlyCloudy } from '@element-plus/icons-vue'
import { onMounted, ref } from 'vue'

const isDark = ref(false);

const toggleTheme = (value: boolean) => {
  if (value) {
    document.documentElement.setAttribute('class', 'dark');
    localStorage.setItem('theme', 'dark');
  } else {
    document.documentElement.setAttribute('class', 'light');
    localStorage.setItem('theme', 'light');
  }
}

onMounted(() => {
  const savedTheme = localStorage.getItem('theme');
  const prefersDark = window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches;
  if (savedTheme) {
    isDark.value = savedTheme === 'dark' ? true : false;
    toggleTheme(isDark.value);
  } else {
    isDark.value = prefersDark;
    toggleTheme(isDark.value);
  }
})
</script>

<template>
  <el-switch
    v-model="isDark"
    inline-prompt
    :active-icon="MostlyCloudy"
    :inactive-icon="Sunny"
    style="--el-switch-on-color: #2c2c2c"
    @change="toggleTheme"
  />
</template>
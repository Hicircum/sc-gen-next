<script setup lang="ts">
import { computed, ref } from 'vue';
import { ElMessage } from 'element-plus'
import SuperChat from './SuperChat.vue';
import html2canvas from 'html2canvas';

const loading = ref(false)

const form = ref({
  dataSource: 0,
  username: '',
  identity: 0,
  isMember: false,
  batteryCount: 300,
  message: '',
  superChatWidth: 270,
  avatar: 1,
});

const dataSourceOptions = ref([
  { label: '手工填写', value: 0 },
  { label: '录播姬xml', value: 1 }
]);
const identityOptions = ref([
  { label: '普通', value: 0 },
  { label: '舰长', value: 1 },
  { label: '提督', value: 2 },
  { label: '总督', value: 3 }
]);
const avatarOptions = ref([
  { label: '不显示', value: 0 },
  { label: '手动上传', value: 1 },
  { label: 'uid获取', value: 2 }
]);
const presetBatteryCounts = ref([300, 500, 1000, 5000, 10000, 20000]);
const showExplorer = ref(false);
const tableData = ref<{ user: string; uid: string; avatar: string; price: string; text: string }[]>([]);

const windowWidth = ref(1080);
window.addEventListener('resize', () => {
  windowWidth.value = window.innerWidth;
});
const labelPosition = computed(() => (windowWidth.value < 600 ? 'top' : 'right'));

const exportImage = async () => {
  const element = document.querySelector('.width-control') as HTMLElement;
  if (!element) return;
  const canvas = await html2canvas(element, {
    backgroundColor: null,
    scale: 10
  });
  const link = document.createElement('a');
  link.href = canvas.toDataURL('image/png');
  link.download = form.value.username + 'SC.png';
  link.click();
};

const imageURL = ref('');
const selectAvatar = () => {
  const input = document.createElement('input');
  input.type = 'file';
  input.accept = 'image/*';
  input.onchange = (event) => {
    const file = (event.target as HTMLInputElement).files?.[0];
    if (file) {
      const reader = new FileReader();
      reader.onload = (e) => {
        imageURL.value = e.target?.result as string;
      };
      reader.readAsDataURL(file);
    }
  };
  input.click();
  console.log(imageURL.value);
};

const bUID = ref('');
const fetchAvatarByUID = async () => {
  if (!bUID.value || loading.value) return;
  loading.value = true;
  try {
    const response = await fetch(`/api/bilibili/avatar/${bUID.value}`);
    if (response.status === 429) {
      console.error("请求过多 (429)，请稍后再试");
      ElMessage({
        message: '429 请求过快',
        type: 'error',
        plain: true,
      })
      return;
    }
    if (!response.ok) throw new Error('网络错误');
    const blob = await response.blob();
    const reader = new FileReader();
    reader.onloadend = () => {
      imageURL.value = reader.result as string;
    };
    reader.readAsDataURL(blob);
  } catch (error) {
    ElMessage({
        message: '网络错误',
        type: 'error',
        plain: true,
    })
  } finally {
    loading.value = false; // 无论成功失败都恢复
  }
};

const parseScTags = (xmlText: string) => {
  const regex = /<sc\s+([^>]*)>(.*?)<\/sc>/gs
  const matches = Array.from(xmlText.matchAll(regex))
  
  if (matches.length === 0) {
    return
  }

  for (const match of matches) {
    const attrs = match[1]
    const innerText = match[2]

    // 提取属性键值对
    const attrRegex = /(\w+)="(.*?)"/g
    const attrObj: Record<string, string> = {}
    if (typeof attrs === 'string') {
      for (const a of attrs.matchAll(attrRegex)) {
        if (a[1] !== undefined && a[2] !== undefined) {
          attrObj[a[1]] = a[2]
        }
      }
    }
    const regex = /&quot;(https:\/\/[^\s"&]+)&quot;/g
    const matches = attrObj['raw'] ? [...attrObj['raw'].matchAll(regex)].map(m => m[1]) : []
    tableData.value.push({
      user: attrObj['user'] || '',
      uid: attrObj['uid'] || '',
      avatar: matches[0] || '',
      price: attrObj['price'] || '0',
      text: innerText || ''
    })
  }
}

const selectXML = () => {
  const input = document.createElement('input');
  input.type = 'file';
  input.accept = '.xml';
  input.onchange = (event) => {
    const file = (event.target as HTMLInputElement).files?.[0];
    if (file) {
      const reader = new FileReader();
      reader.onload = (e) => {
        const text = e.target?.result as string;
        parseScTags(text);
      };
      reader.readAsText(file);
    }
  };
  input.click();
};

const setMessage = (item: { user: string; uid: string; price: string; text: string }) => {
  form.value.username = item.user;
  form.value.batteryCount = parseInt(item.price) * 10 || 0;
  form.value.message = item.text;
  form.value.isMember = false;
  bUID.value = item.uid;
  form.value.avatar = 2;
  showExplorer.value = false;
};
</script>

<template>
  <el-scrollbar max-height="180px" class="sc-container">
    <div class="width-control" :style="{ width: form.superChatWidth + 'px' }">
      <SuperChat
        :battery-count="form.batteryCount"
        :username="form.username"
        :identity="form.identity"
        :message="form.message"
        :member="form.isMember"
        :show-avatar="form.avatar !== 0"
        :avatar="imageURL"
      />
    </div>
  </el-scrollbar>
  <div class="setting">
    <el-form
      label-width="auto"
      :model="form"
      :label-position="labelPosition"
      style="max-width: 830px;margin: 0 auto;"
    >
      <el-form-item label="数据源">
        <el-segmented v-model="form.dataSource" :options="dataSourceOptions" />
        <el-button v-if="form.dataSource === 1"
          @click="showExplorer = true" style="margin-left: 10px;"
        >打开面板</el-button>
      </el-form-item>
      <el-form-item label="宽度">
        <el-slider v-model="form.superChatWidth" :min="270" :max="750"/>
      </el-form-item>
      <el-form-item label="用户名">
        <el-input v-model="form.username" placeholder="阿卡林"/>
        <el-checkbox v-model="form.isMember" name="type">
          大会员
        </el-checkbox>
      </el-form-item>
      <el-form-item label="身份">
        <el-segmented v-model="form.identity" :options="identityOptions" />
      </el-form-item>
      <el-form-item label="头像">
        <el-segmented v-model="form.avatar" :options="avatarOptions" />
        <el-button v-if="form.avatar === 1"
          @click="selectAvatar" style="margin-left: 10px;"
        >选择头像</el-button>
      </el-form-item>
      <el-form-item label=" " v-if="form.avatar === 2">
        <el-input
          style="max-width: 200px;"
          placeholder="输入B站UID"
          v-model="bUID"
        />
        <el-button style="margin-left: 10px;"
          @click="fetchAvatarByUID"
          :loading="loading"
          :disabled="loading"
          >获取</el-button>
      </el-form-item>
      <el-form-item label="留言内容">
        <el-input v-model="form.message"
          type="textarea"
          :autosize="{ minRows: 4, maxRows: 6 }"
          placeholder="输入留言内容传递自己的心意吧！" />
      </el-form-item>
      <el-form-item label="电池数">
        <div class="battery">
          <el-button class="preset" size="large"
            v-for="i in presetBatteryCounts" :key="i" @click="form.batteryCount = i">
            <img src="../assets/battery.png" alt=""> {{ i }}
          </el-button>
        </div>
      </el-form-item>
      <el-form-item label=" ">
        <el-input-number v-model="form.batteryCount" :min="0" />
      </el-form-item>
      <el-form-item label=" ">
        <el-button type="primary" @click="exportImage">导出图片</el-button>
      </el-form-item>
    </el-form>
  </div>
  <el-dialog v-model="showExplorer" width="80%" style="min-height: 500px;">
    <div class="alert">由于跨域限制，头像无法直接使用，请右键复制链接，在新标签粘贴并打开</div>
    <template #header>
      <span>数据浏览</span>
      <el-button size="small" style="margin-left: 10px;" @click="selectXML">选择文件</el-button>
    </template>
    <el-table :data="tableData" style="width: 100%">
      <el-table-column label="操作" width="80">
        <template #default="scope">
          <el-button size="small" @click="setMessage(scope.row)" type="primary">使用</el-button>
        </template>
      </el-table-column>
      <el-table-column prop="user" label="用户名" width="120" />
      <el-table-column prop="uid" label="UID" width="120" />
      <el-table-column prop="avatar" label="头像" width="120">
        <template #default="scope">
          <img v-if="scope.row.avatar" :src="scope.row.avatar" alt="avatar" style="height: 40px;" referrerpolicy="no-referrer"/>
          <span v-else>无</span>
        </template>
      </el-table-column>
      <el-table-column prop="price" label="价格" width="100" />
      <el-table-column prop="text" label="内容" min-width="350" />
    </el-table>
  </el-dialog>
</template>

<style scoped lang="scss">
.sc-container{
  min-height: 120px;
  margin-bottom: 20px;
  .width-control {
    margin: 0 auto;
  }
}
.setting {
  padding: 40px 20px;
  border-radius: 12px;
  box-shadow: var(--custom-box-shadow);
  background-color: var(--custom-bg-color);
}
.battery {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  .preset {
    margin-left: 0 !important;
  }
  img {
    height: 1.1em;
    margin-right: 4px;
    transform: translateY(-.05rem);
    user-select: none;
  }
}
</style>

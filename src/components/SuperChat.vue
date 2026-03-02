<script setup lang="ts">
import { computed } from 'vue';

const props = defineProps({
  username: {
    type: String,
    default: '用户名',
  },
  identity: {
    type: Number,
    default: 0,
  },
  isMember: {
    type: Boolean,
    default: false,
  },
  batteryCount: {
    type: Number,
    default: 300,
  },
  message: {
    type: String,
    default: '输入留言内容传递自己的心意吧！',
  },
  avatar: {
    type: String
  },
  showAvatar: {
    type: Boolean,
    default: true,
  },
  member: {
    type: Boolean,
    default: false,
  },
});

const SC = computed(() => {
  return {
    username: props.username || '阿卡林',
    message: props.message || '输入留言内容传递自己的心意吧！',
    batteryCount: props.batteryCount || 300,
    avatar: props.avatar
  };
});

const priceLevel = computed(() => {
  if (props.batteryCount >= 20000) {
    return 'lv6';
  }
  if (props.batteryCount >= 10000) {
    return 'lv5';
  }
  if (props.batteryCount >= 5000) {
    return 'lv4';
  }
  if (props.batteryCount >= 1000) {
    return 'lv3';
  }
  if (props.batteryCount >= 500) {
    return 'lv2';
  }
  return 'lv1';
});
</script>

<template>
  <div class="sc-main" :price="priceLevel">
    <div class="sc-header">
      <div class="user-info" :avatar="showAvatar">
        <div class="avatar" v-if="showAvatar">
          <img class="border" v-if="props.identity === 1" src="../assets/BSC_jz.webp" alt="">
          <img class="border" v-if="props.identity === 2" src="../assets/BSC_td.webp" alt="">
          <img class="border" v-if="props.identity === 3" src="../assets/BSC_zd.webp" alt="">
          <img v-if="avatar" :src="SC.avatar" alt="avatar" referrerpolicy="no-referrer"/>
          <img v-else src="../assets/Transparent_Akkarin.jpg" alt="avatar"/>
        </div>
        <div class="username" :class="{ member: props.member }">{{ SC.username }}</div>
        <div class="price">{{ SC.batteryCount }} 电池</div>
      </div>
      <div class="background">
        <img src="../assets/BSC_Background.webp" alt="">
      </div>
    </div>
    <div class="sc-body">
      {{ SC.message }}
    </div>
  </div>
</template>

<style scoped lang="scss">
.sc-main[price="lv1"] {
  --border-color: rgb(42, 96, 178);
  --header-bg-color: rgb(237, 245, 255);
  --body-bg-color: rgb(42, 96, 178);
}
.sc-main[price="lv2"] {
  --border-color: rgb(66, 125, 158);
  --header-bg-color: rgb(219, 255, 253);
  --body-bg-color: rgb(66, 125, 158);
}
.sc-main[price="lv3"] {
  --border-color: rgb(226, 181, 43);
  --header-bg-color: rgb(255, 241, 197);
  --body-bg-color: rgb(226, 181, 43);
}
.sc-main[price="lv4"] {
  --border-color: rgb(224, 148, 67);
  --header-bg-color: rgb(255, 234, 210);
  --body-bg-color: rgb(224, 148, 67);
}
.sc-main[price="lv5"] {
  --border-color: rgb(229, 77, 77);
  --header-bg-color: rgb(255, 231, 228);
  --body-bg-color: rgb(229, 77, 77);
}
.sc-main[price="lv6"] {
  --border-color: rgb(171, 26, 50);
  --header-bg-color: rgb(255, 216, 216);
  --body-bg-color: rgb(171, 26, 50);
}
.user-info[avatar="false"] {
  padding-left: 10px !important;
}
.sc-main {
  border: 1px solid var(--border-color);
  border-radius: 6px;
  overflow: hidden;
  font-family: Arial, "Microsoft YaHei", "Microsoft Sans Serif", "Microsoft SanSerf", "微软雅黑";
  .sc-header {
    height: 48px;
    padding: 8px 0;
    box-sizing: border-box;
    background-position: 100% 0;
    background-repeat: no-repeat;
    background-color: var(--header-bg-color);
    position: relative;
    .user-info {
      padding-top: 3px;
      padding-left: 55px;
      color: #333;
      position: relative;
      .avatar {
        top: 0;
        left: 15px;
        width: 34px;
        height: 34px;
        position: absolute;
        border-radius: 50%;
        img {
          width: 100%;
          height: 100%;
          object-fit: cover;
          object-position: center center;
          border-radius: 50%;
        }
        .border {
          position: absolute;
          width: 100%;
          height: 100%;
          z-index: 1;
          border-radius: 0%;
          transform: scale(1.2);
        }
      }
      .username {
        max-width: 100%;
        font-size: 12px;
        line-height: 15px;
        color: #333;
        opacity: .78;
        white-space: nowrap;
        text-overflow: ellipsis;
        overflow: hidden;
        &.member {
          color: #f9708e;
        }
      }
      .price {
        font-size: 14px;
        line-height: 15px;
      }
    }
    .background {
      position: absolute;
      top: 0;
      right: -35px;
      height: 48px;
      img {
        height: 100%;
        object-fit: cover;
      }
    }
  }
  .sc-body {
    min-height: 37px;
    background-color: var(--body-bg-color);
    width: 100%;
    padding: 7px 10px 7px 10px;
    box-sizing: border-box;
    color: #fff;
    font-size: 12px;
    unicode-bidi: isolate;
    line-height: 17px;
    color: rgb(255, 255, 255);
    word-break: break-all;
    white-space: pre-line;
  }
}
</style>

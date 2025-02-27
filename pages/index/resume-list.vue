<template>
  <view class="resume-list">
    <view v-for="(resume, index) in resumeList" :key="index" class="resume-item">
      <image class="resume-icon" :src="resume.icon"></image>
      <text class="resume-name">{{ resume.name }}</text>
      <text class="resume-time">{{ resume.time }}</text>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      resumeList: [] // 用来接收传递过来的简历列表
    }
  },
  onLoad(options) {
    if (options.resumeList) {
      this.resumeList = JSON.parse(options.resumeList) // 解析传递过来的简历列表
    }
  }
}
</script>

<style scoped>
.resume-list {
  padding: 20rpx;
}

.resume-item {
  display: flex;
  align-items: center;
  margin-bottom: 20rpx;
}

.resume-icon {
  width: 50rpx;
  height: 50rpx;
  border-radius: 50%;
  margin-right: 20rpx;
}

.resume-name {
  font-size: 28rpx;
  color: #333;
}

.resume-time {
  margin-left: auto;
  font-size: 24rpx;
  color: #999;
}
</style>

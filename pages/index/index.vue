<template>
  <view class="container">
    <!-- 顶部横幅图 -->
    <image 
      class="banner" 
      src="/static/img/back.png" 
      mode="widthFix"
    ></image>

    <!-- 四宫格功能区域 -->
    <view class="grid-container">
      <!-- 第一行 -->
      <view class="grid-row">
        <view class="grid-item" @click="handleUploadResume">
          <image class="grid-icon" src="/static/icon/upload-file.png"></image>
          <text class="grid-text">上传简历</text>
        </view>
        
        <view class="grid-item">
          <image class="grid-icon" src="/static/icon/boost.png"></image>
          <text class="grid-text">简历优化</text>
        </view>
      </view>

      <!-- 第二行 -->
      <view class="grid-row">
        <view class="grid-item">
          <image class="grid-icon" src="/static/icon/heart.png"></image>
          <text class="grid-text">我的收藏</text>
        </view>
        
        <view class="grid-item">
          <image class="grid-icon" src="/static/icon/label.png"></image>
          <text class="grid-text">我的简历</text>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
import request from '@/utils/request.js'

export default {
  data() {
    return {
      title: '欢迎使用'
    }
  },
  methods: {
    // 上传简历处理
    async handleUploadResume() {
      try {
        // 1. 检查登录状态
        const token = uni.getStorageSync('token')
        if (!token) {
          uni.showToast({ title: '请先登录', icon: 'none' })
          return uni.switchTab({ url: '/pages/me/me' })
        }

        // 2. 选择文件
        const res = await uni.chooseMessageFile({
          count: 1,
          type: 'file',
          extension: ['.pdf', '.doc', '.docx']
        })

        // 3. 获取文件信息
        const file = res.tempFiles[0]

        // 4. 执行上传
        const uploadRes = await this.uploadFile(file)
        
        // 5. 处理上传结果
        if (uploadRes.code === 200) {
          uni.showToast({ title: '上传成功' })
          // 这里可以添加成功后的业务逻辑
        } else {
          throw new Error(uploadRes.msg || '上传失败')
        }
      } catch (error) {
        console.error('上传失败:', error)
        uni.showToast({ 
          title: error.message || '上传失败',
          icon: 'none'
        })
      }
    },

    // 封装上传方法
    uploadFile(file) {
      return new Promise((resolve, reject) => {
        uni.uploadFile({
          url: 'https://119.84.246.218:57534/resume/uploadResume',
          filePath: file.path,
          name: 'file',
          header: {
            'Content-Type': 'multipart/form-data',
            'token': uni.getStorageSync('token')
          },
          formData: {
            fileName: file.name,
            fileType: file.type
          },
          success: (res) => {
            try {
              const data = JSON.parse(res.data)
              resolve(data)
            } catch (e) {
              reject(new Error('响应解析失败'))
            }
          },
          fail: (err) => {
            reject(new Error('网络请求失败'))
          }
        })
      })
    }
  }
}
</script>

<style>
.container {
  background-color: #f8f8f8;
  min-height: 100vh;
}

/* 横幅图样式 */
.banner {
  width: 100%;
  height: 300rpx;
  display: block;
  margin-bottom: 40rpx;
}

/* 四宫格容器 */
.grid-container {
  background-color: #ffffff;
  border-radius: 20rpx;
  margin: 0 30rpx;
  padding: 30rpx 0;
  box-shadow: 0 4rpx 12rpx rgba(0,0,0,0.05);
}

/* 行样式 */
.grid-row {
  display: flex;
  justify-content: space-around;
  margin: 40rpx 0;
}

/* 单个功能项 */
.grid-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 300rpx;
}

/* 图标样式 */
.grid-icon {
  width: 100rpx;
  height: 100rpx;
  margin-bottom: 20rpx;
}

/* 文字样式 */
.grid-text {
  font-size: 28rpx;
  color: #333;
  font-weight: 500;
}
</style>
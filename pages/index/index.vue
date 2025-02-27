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
        
        <view class="grid-item" @click="handleMyResume">
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
      title: '欢迎使用',
      resumeList: []  // 用来保存简历列表
    }
  },
  methods: {
	// 上传简历处理
	async handleUploadResume() {
		try {
			// 1. 检查登录状态
			const token = uni.getStorageSync('token');
			if (!token) {
				uni.showToast({ title: '请先登录', icon: 'none' });
				return uni.switchTab({ url: '/pages/me/me' });
			}

			// 2. 选择文件
			const res = await uni.chooseMessageFile({
				count: 1,
				type: 'file',
				extension: ['.pdf', '.doc', '.docx']  // 限制文件类型为 .pdf, .doc, .docx
			});

			// 3. 获取文件信息
			const file = res.tempFiles[0];

			// 4. 如果文件类型不匹配，提示用户
			const fileExtension = this.getFileExtension(file.name);
			if (!['pdf', 'doc', 'docx'].includes(fileExtension.toLowerCase())) {
				uni.showToast({
					title: '请选择PDF或Word文件',
					icon: 'none'
				});
				return;
			}

			// 5. 执行上传
			const uploadRes = await this.uploadFile(file);
			
			// 6. 处理上传结果
			if (uploadRes.code === '200') {
				uni.showToast({ title: '上传成功' });
			} else {
				throw new Error(uploadRes.msg || '上传失败');
			}
		} catch (error) {
			console.error('上传失败:', error);
			uni.showToast({
				title: error.message || '上传失败',
				icon: 'none'
			});
		}
	},

	// 获取文件扩展名
	getFileExtension(fileName) {
		const index = fileName.lastIndexOf('.');
		if (index === -1) {
			return ''; // 无扩展名的文件
		}
		return fileName.substring(index + 1);
	},

	// 封装上传文件方法
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
					fileName: file.name,  // 只传递文件名
				},
				success: (res) => {
					try {
						const data = JSON.parse(res.data);
						resolve(data);
					} catch (e) {
						reject(new Error('响应解析失败'));
					}
				},
				fail: (err) => {
					reject(new Error('网络请求失败'));
				}
			});
		});
	},

    // 点击“我的简历”时获取简历列表
    async handleMyResume() {
      try {
        const token = uni.getStorageSync('token')
        if (!token) {
          uni.showToast({ title: '请先登录', icon: 'none' })
          return uni.switchTab({ url: '/pages/me/me' })
        }

        // 发送请求获取简历列表
        const response = await request.get('/myResume', {
          token: token  // 传递 token
        })

        if (response.code === '200') {
          this.resumeList = response.data // 假设返回的数据是一个简历列表
          uni.navigateTo({
            url: '/pages/resume-list/resume-list?resumeList=' + JSON.stringify(this.resumeList) // 将简历数据传递到新页面
          })
        } else {
          throw new Error(response.msg || '获取简历列表失败')
        }
      } catch (error) {
        console.error('获取简历列表失败:', error)
        uni.showToast({ 
          title: error.message || '获取简历列表失败',
          icon: 'none'
        })
      }
    }
  }
}
</script>

<style src="./index.css"></style>

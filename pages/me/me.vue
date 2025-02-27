<template>
  <view class="page_container">
    <!-- 顶部背景 -->
    <view class="top_bg"></view>

    <!-- 主内容区域 -->
    <view class="main_content">
      <!-- 用户信息展示 -->
      <view v-if="hasUserInfo" class="userinfo_container">
        <image class="avatar" :src="userInfo.avatarUrl" mode="widthFix" />
        <text class="username">{{ userInfo.nickName }}</text>
      </view>

      <!-- 未登录时显示默认头像 -->
      <view v-else class="userinfo_container">
        <image class="avatar" :src="defaultAvatar" mode="widthFix" />
        <text class="username">未登录</text>
      </view>

      <!-- 设置项展示 -->
      <view class="settings_container">
        <view class="setting_item" @click="updateNickName">
          <view class="left">
            <image class="setting_icon" src="@/static/icon/share.svg" />
            <view class="setting_text">修改昵称</view>
          </view>
          <image class="arrow_icon" src="@/static/icon/arrow.svg" />
        </view>
      </view>

      <!-- 登录按钮 -->
      <view v-if="!hasUserInfo" class="login-btn-container">
        <button class="login-btn" @click="handleLogin">微信授权登录</button>
      </view>
    </view>
  </view>
</template>

<script>
import request from '@/utils/request.js'; // 导入 request.js
export default {
  data() {
    return {
      userInfo: null, // 用户信息
      hasUserInfo: false, // 是否已获取用户信息
      defaultAvatar: require('@/static/icon/avatar.png'), // 默认头像路径
			token: null
    };
  },
  onLoad() {
    this.checkLogin();
	wx.showShareMenu({
	withShareTicket: true,
	menus: ["shareAppMessage", "shareTimeline"]
	});
  },
  methods: {
    // 检查登录状态
    checkLogin() {
      const userInfo = uni.getStorageSync('userInfo');
      if (userInfo) {
        this.userInfo = userInfo;
        this.hasUserInfo = true;
      } else {
        this.hasUserInfo = false;
      }
    },
    // 处理登录逻辑
    async handleLogin() {
      try {
        // 1. 获取临时凭证 code
        const loginRes = await new Promise((resolve, reject) => {
          uni.login({
            provider: 'weixin',
            success: resolve,
            fail: reject,
          });
        });
        const code = loginRes.code;
        if (!code) throw new Error('登录失败：无法获取 code');
        console.log('临时凭证 code:', code);

        // 2. 获取用户信息
        const userInfoRes = await new Promise((resolve, reject) => {
          uni.getUserInfo({
            provider: 'weixin',
            success: resolve,
            fail: reject,
          });
        });
        const wxUserInfo = userInfoRes.userInfo;
        console.log('用户头像:', wxUserInfo.avatarUrl);
        console.log('用户昵称:', wxUserInfo.nickName);
        console.log('用户详细信息:', wxUserInfo);

        const response = await request.post('/user/login', {
          code,
          userInfo: wxUserInfo,
        });
        if (response.data && response.data.token) {
          uni.setStorageSync('token', response.data.token); 
          console.log('Token已存储');
        } else {
          console.error('Token未找到于响应中');
        }
        
        if (response.code === '200') {
          // 登录成功，保存用户信息
          this.userInfo = wxUserInfo;
          this.hasUserInfo = true;
          uni.setStorageSync('userInfo', wxUserInfo);
          uni.showToast({
            title: '登录成功',
            icon: 'success',
          });
        } else {
          throw new Error('登录失败：后端接口返回错误');
        }
      } catch (error) {
        console.error('登录出错:', error);
        uni.showToast({
          title: '登录失败，请重试',
          icon: 'none',
        });
      }
    },
		async updateNickName() {
			try {
			// 弹出输入框，让用户输入新的昵称
			const res = await new Promise((resolve, reject) => {
			uni.showModal({
				title: '修改昵称',
				content: '请输入新的昵称',
				editable: true,
				success: resolve,
				fail: reject,
			});
			});

			if (res.confirm) {
			const newNickName = res.content;

			// 发送请求到后端更新昵称
			const response = await request.post('/user/updateNickName', {
				userId: this.userInfo.userId, // 假设用户信息中有 userId
				nickName: newNickName,
			});

			if (response.code === '200') {
				// 更新成功，更新本地用户信息
				this.userInfo.nickName = newNickName;
				uni.setStorageSync('userInfo', this.userInfo);
				uni.showToast({
				title: '昵称修改成功',
				icon: 'success',
				});
				} else {
					throw new Error('昵称修改失败：后端接口返回错误');
				}
			}
		} catch (error) {
			console.error('修改昵称出错:', error);
			uni.showToast({
			title: '修改昵称失败，请重试',
			icon: 'none',
			});
		}
	},
	onShareAppMessage(res) {
	if (res.from === 'button') {
	console.log(res.target);
	}
	return {
	title: '分享给微信好友',
	path: '/pages/index/index',
	mpId: 'wx6bf107b87c455b99'
	};
	},
	onShareTimeline(res) {
	return {
	title: '分享到朋友圈',
	query: 'pages/index/index',
	imageUrl: '/static/index.png'
	};
	}
  },
};
</script>

<style>
.page_container {
  background-color: #f5f5f5;
  height: 100vh;
  font-family: PingFang SC;
}

.top_bg {
  height: 120rpx;
}

.main_content {
  background-color: #fff;
  border-top-left-radius: 30rpx;
  border-top-right-radius: 30rpx;
  height: calc(100vh - 120rpx);
  position: relative;
}

.userinfo_container {
  display: flex;
  gap: 20rpx;
  align-items: center;
  position: absolute;
  top: -30rpx;
  padding: 0 60rpx;
}

.avatar {
  width: 150rpx;
  height: 150rpx;
  border-radius: 50%;
}

.username {
  color: rgba(0, 0, 0, 0.9);
  font-size: 32rpx;
  margin-top: 20rpx;
  max-width: 200rpx;
}

.login-btn-container {
  position: absolute;
  bottom: 200rpx; /* 调整按钮位置 */
  left: 50%;
  transform: translateX(-50%);
  width: 100%;
  text-align: center;
}

.login-btn {
  width: 400rpx;
  height: 80rpx;
  line-height: 80rpx;
  text-align: center;
  background-color: #07c160;
  color: #fff;
  border-radius: 40rpx;
  font-size: 32rpx;
}

.settings_container {
  position: relative;
  top: 150rpx;
  padding: 0 40rpx;
  display: flex;
  flex-direction: column;
  gap: 10rpx;
}

.setting_item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 30rpx 0rpx;
  border-bottom: 1rpx solid rgba(0, 0, 0, 0.08);
  font-size: 28rpx;
}

.left {
  display: flex;
  align-items: center;
  gap: 20rpx;
}

.setting_icon {
  width: 40rpx;
  height: 40rpx;
}

.setting_text {
  color: rgba(0, 0, 0, 0.9);
}

.arrow_icon {
  width: 30rpx;
  height: 30rpx;
}
</style>

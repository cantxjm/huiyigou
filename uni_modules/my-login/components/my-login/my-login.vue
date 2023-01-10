<template>
	<view>
		<view class="login-container">
			<uni-icons type="contact" size="100" color="#AFAFAF"></uni-icons>
			<button type="primary" class="btn-login" @click="getUserInfo">一键登录</button>
			<text class="tips-text">登录后尽享更多权益</text>
		</view>
	</view>
</template>
<script>
	import {
		mapMutations,
		mapState
	} from 'vuex'
	export default {
		data() {
			return {}
		},
		computed: {
			// 调用 mapState 辅助方法，把 m_user 模块中的数据映射到当前用组件中使用
			...mapState('m_user', ['redirectInfo']),
		},
		methods: {
			...mapMutations('m_user', ['updateUserInfo', 'updateToken', 'updateRedirectInfo']),
			// 用户授权之后，获取用户的基本信息
			getUserInfo() {
				uni.getUserProfile({
					desc: '获取本人信息',
					success: (res) => {
						// console.log(res);
						this.updateUserInfo(res.userInfo)
						this.getToken(res)
					},
					fail: () => {
						uni.$showMsg('您取消了登录授权')
					}
				})
			},
			async getToken(info) {
				// 调用微信登录接口
				const [err, res] = await uni.login().catch(err => err)
				// 判断是否 uni.login() 调用失败
				if (err || res.errMsg !== 'login:ok') return uni.$showError('登录失败！')

				// console.log(res);
				// 准备参数对象
				const query = {
					code: res.code,
					encryptedData: info.encryptedData,
					iv: info.iv,
					rawData: info.rawData,
					signature: info.signature
				}

				// 换取 token
				const chars =
					'Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1aWQiOjEyLCJpYXQiOjE1MjU0MDIyMjMsImV4cCI6MTUyNTQ4ODYyM30.g-4GtEQNPwT_Xs0Pq7Lrco_9DfHQQsBiOKZerkO-O-o ';
				const {
					data: loginResult
				} = await uni.$http.post('/api/public/v1/users/wxlogin', query)
				// console.log(loginResult);
				// 持久化存储token(接口有问题！！将就这么写)
				if (loginResult.meta.status !== 200) this.updateToken(chars)

				this.navigateBack()
			},

			navigateBack() {
				if (this.redirectInfo && this.redirectInfo.openType === 'switchTab') {
					uni.switchTab({
						url: this.redirectInfo.from,
						// 导航成功之后，把 vuex 中的 redirectInfo 对象重置为 null
						complete: () => {
							this.updateRedirectInfo(null)
						}
					})
				}
			}
		},

	}
</script>
<style lang="scss">
	.login-container {
		height: 750rpx;
		background-color: white;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		position: relative;
		overflow: hidden;

		&::after {
			content: ' ';
			display: block;
			width: 100%;
			height: 40px;
			background-color: #f5f5f5;
			position: absolute;
			bottom: 0;
			left: 0;
			border-radius: 100%;
			transform: translateY(50%);
		}

		.btn-login {
			width: 90%;
			border-radius: 100px;
			margin: 15px 0;
			background-color: #c00000;
		}

		.tips-text {
			font-size: 12px;
			color: gray;
		}

		.userinfo-avatar {
			overflow: hidden;
			width: 128rpx;
			height: 128rpx;
			margin: 20rpx;
			border-radius: 50%;
		}
	}
</style>

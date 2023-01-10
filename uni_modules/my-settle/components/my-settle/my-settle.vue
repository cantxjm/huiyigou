<template>
	<!-- 最外层的容器 -->
	<view class="my-settle-container">
		<!-- 全选区域 -->
		<label class="radio" @click="changeAllState">
			<radio color="#C00000" :checked="isFullCheck" /><text>全选</text>
		</label>

		<!-- 合计区域 -->
		<view class="amount-box">
			合计:<text class="amount">￥{{checkedGoodsAmount}}</text>
		</view>

		<!-- 结算按钮 -->
		<view class="btn-settle" @click="settlement">结算({{checkedCount}})</view>
	</view>
</template>
<script>
	import {
		mapGetters,
		mapMutations,
		mapState
	} from 'vuex'
	export default {
		data() {
			return {
				// 倒计时的秒数
				seconds: 3,
				// 定时器的 Id
				timer: null
			}
		},
		computed: {
			...mapGetters('m_cart', ['checkedCount', 'total', 'checkedGoodsAmount']),
			...mapGetters('m_user', ['addstr']),
			...mapState('m_user', ['token']),
			...mapState('m_cart', ['cart']),
			isFullCheck() {
				return this.total === this.checkedCount
			}
		},
		methods: {
			...mapMutations('m_cart', ['updateAllGoodsState']),
			...mapMutations('m_user', ['updateRedirectInfo']),
			changeAllState() {
				this.updateAllGoodsState(!this.isFullCheck)
			},
			// 用户点击了结算按钮
			settlement() {
				if (!this.checkedCount) return uni.$showMsg('请选择要结算的商品!')

				if (!this.addstr) return uni.$showMsg('请选择收货地址!')

				// if (!this.token) return uni.$showMsg('请先登录!')
				if (!this.token) return this.delayNavigate()

				// 实现微信支付功能
				this.payOrder()
			},
			// 展示倒计时的提示消息
			showTips(n) {
				// 调用 uni.showToast() 方法，展示提示消息
				uni.showToast({
					// 不展示任何图标
					icon: 'none',
					// 提示的消息
					title: '请登录后再结算！' + n + ' 秒后自动跳转到登录页',
					// 为页面添加透明遮罩，防止点击穿透
					mask: true,
					// 1.5 秒后自动消失
					duration: 1500
				})
			},
			// 延迟导航到 my 页面
			delayNavigate() {
				this.seconds = 3

				// 1. 展示提示消息，此时 seconds 的值等于 3
				this.showTips(this.seconds)

				// 2. 创建定时器，每隔 1 秒执行一次
				this.timer = setInterval(() => {
					// 2.1 先让秒数自减 1
					this.seconds--

					if (this.seconds <= 0) {
						clearInterval(this.timer)

						uni.switchTab({
							url: '/pages/my/my',
							success: () => {
								this.updateRedirectInfo({
									openType: 'switchTab',
									from: '/pages/cart/cart'
								})
							}
						})

						return
					}
					// 2.2 再根据最新的秒数，进行消息提示
					this.showTips(this.seconds)
				}, 1000)
			},
			// 微信支付
			async payOrder() {
				// 1. 创建订单
				// 1.1 组织订单的信息对象
				const orderInfo = {
					// 开发期间，注释掉真实的订单价格，
					// order_price: this.checkedGoodsAmount,
					// 写死订单总价为 1 分钱
					order_price: 0.01,
					consignee_addr: this.addstr,
					goods: this.cart.filter(x => x.goods_state).map(x => ({
						goods_id: x.goods_id,
						goods_number: x.goods_count,
						goods_price: x.goods_price
					}))
				}
				// 1.2 发起请求创建订单
				const {
					data: res
				} = await uni.$http.post('/api/public/v1/my/orders/create', orderInfo)
				// console.log(res);
				if (res.meta.status !== 200) return uni.$showMsg('创建订单失败！')
				// 1.3 得到服务器响应的“订单编号”
				const orderNumber = res.message.order_number
				
				console.log(orderNumber);

				// 2. 订单预支付

				// 3. 发起微信支付
			}
		}
	}
</script>
<style lang="scss">
	.my-settle-container {
		position: fixed;
		bottom: 0;
		left: 0;
		width: 100%;
		height: 50px;
		background-color: white;
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding-left: 5px;
		font-size: 14px;

		.radio {
			display: flex;
			align-items: center;
		}

		.amount {
			color: #c00000;
			font-weight: bold;
		}

		.btn-settle {
			height: 50px;
			min-width: 100px;
			background-color: #c00000;
			color: white;
			line-height: 50px;
			text-align: center;
			padding: 0 10px;
		}
	}
</style>

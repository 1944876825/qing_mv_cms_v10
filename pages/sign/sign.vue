<template>
	<view class="content flex-col" style="padding: 20rpx;">
		<view class="header">
			
		</view>
		<view class="center" style="padding: ;">
			<view class="">
				<view class="flex-row nT" style="background-color: #fff;border-radius: 20rpx;padding: 30rpx;">
					<text style="font-size: 14px;font-weight: 600;color: gray;">昵称：</text>
					<input type="text" v-model="name" placeholder="昵称" style="font-size: 14px;padding: 0 15rpx;width: 420rpx;">
				</view>
				<view class="flex-row nT" style="background-color: #fff;border-radius: 20rpx;padding: 30rpx;margin-top: 15rpx;">
					<text style="font-size: 14px;font-weight: 600;color: gray;">QQ：</text>
					<input type="text" v-model="qq" placeholder="QQ号(展示头像)" style="font-size: 14px;padding: 0 15rpx;width: 420rpx;">
				</view>
			</view>
			<view
				@tap="sign"
				class="nT"
				style="background-color: #fff;border-radius: 20rpx;margin-top: 30rpx;height: 80rpx;text-align: center;line-height: 80rpx;font-size: 16px;font-weight: 600;">
				设置
			</view>
		</view>
		<view class="bottom">
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				name: '',
				qq: '',
			};
		},
		onLoad() {
			var u = getApp().globalData.userInfo
			this.name = u['name']
			this.qq = u['qq']
		},
		methods: {
			sign() {
				var data = {
					name: this.name,
					qq: this.qq
				}
				getApp().globalData.userInfo = data
				uni.setStorage({
					key: 'userInfo',
					data: data,
					success:res=>{
						uni.showToast({
							icon:'none',
							title:'设置成功'
						})
						uni.switchTab({
							url:'/pages/center/center'
						})
					},
					fail:err=>{
						uni.showToast({
							icon:'none',
							title:'设置失败'
						})
					}
				})
				
			},
		}
	}
</script>

<style lang="scss" scoped>

</style>

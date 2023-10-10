<template>
	<view style="padding: 0 20rpx;">
		<view class="" v-if="videoList['length']>0">
			<view class="flex-row" style="padding: 5rpx 0;flex-wrap: wrap;">
				<view class="videoCardMain flex-col" v-for="video in videoList" @tap="playVideo(video['vod_id'])">
					<view class="videoCardImgBox">
						<image class="videoCardImg" :src="video['vod_pic']" mode="aspectFill"></image>
					</view>
					<view class="videoCardName line_1">
						{{video['vod_name']}}
					</view>
				</view>
			</view>
			<view class="" style="padding: 15rpx;" @tap="getPageMaccms">
				<u-loadmore :status="status" />
			</view>
		</view>
		<view class="" v-else style="margin-top: 60rpx;">
			<u-loading-page :loading="ifShowLoading"></u-loading-page>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				videoList: [],
				value: '',
				mold: 0,
				pg: 1,
				status: 'loadmore',
				ifShowLoading: true,
			};
		},
		onLoad(option) {
			this.value = option.value
			this.mold = option.mold
			if (option.label) {
				uni.setNavigationBarTitle({
					title: option.label
				})
			}
			this.getPageMaccms()
		},
		onPullDownRefresh() {
			this.videoList = []
			this.ifShowLoading = true
			this.status = 'loadmore'
			this.getPageMaccms()
			setTimeout(function() {
				uni.stopPullDownRefresh();
			}, 500);
		},
		onReachBottom() {
			this.getPageMaccms()
		},
		methods: {
			getPageMaccms() {
				if (this.status != 'nomore') {
					uni.$u.http.post('addons/appto/app.php/v2/video/lists', {
						'mold': this.mold,
						'value': this.value,
						'page': this.pg,
					}).then(res => {
						this.ifShowLoading = false
						this.videoList.push(...res.data['list'])
						if (this.videoList.length >= res.data['total']) {
							this.status = 'nomore'
						}
						this.pg++
					}).catch(err => {
						uni.showToast({
							icon: 'none',
							title: '获取失败'
						})
					})
				}
			}
		}
	}
</script>

<style lang="scss" scoped>
	.videoCardMain {
		width: 235rpx;
		padding: 10rpx 0;
		.videoCardImgBox {
			text-align: center;
			.videoCardImg {
				width: 220rpx;
				height: 280rpx;
				border-radius: 16rpx;
			}
		}
		.videoCardName {
			font-weight: 600;
			font-size: 12px;
			padding: 0 15rpx;
			text-align: center;
		}
		.videoCardRate {
			padding: 3rpx 15rpx;
		}
	}
</style>

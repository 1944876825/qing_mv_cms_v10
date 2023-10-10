<template>
	<view class="content flex-col" style="background-color: #fff;">
		<view class="header">
			<view class="head">
				
			</view>
			<view class="flex-row items-center justify-between" style="padding: 15rpx;">
				<view class="flex-row">
					<u-icon name="arrow-left" color="#5e5e5e" size="20" @tap="toIndex"></u-icon>
					<view class="" style="width: 550rpx;margin-left: 15rpx;">
						<u-search
							placeholder="电影 / 电视剧 / 动漫"
							v-model="wd"
							height="60rpx"
							:showAction="false"
							searchIconSize="16"
							style="font-size: 13px;"
							bgColor="#f2f4f3"
							focus
						></u-search>
					</view>
				</view>
				<view class="flex-col justify-center" style="">
					<text style="font-size: 15px;color: #FB7299;padding: 0 30rpx;" @tap="getSearchData">搜索</text>
				</view>
			</view>
		</view>
		<view class="center">
			<!-- 热搜功能 （由于找不到后端API而暂停开发） -->
			<!-- <view class="flex-col" style="padding: 0 20rpx;margin-top: 10rpx;">
				<view class="flex-col">
					<view class="">
						<text style="font-size: 14px;font-weight: 600;">热搜</text>
					</view>
					<view class="flex-row">
						<view class="">
							
						</view>
					</view>
				</view>
			</view> -->
			
			<!-- 搜索视频列表 -->
			<view class="flex-col" style="padding: 0 20rpx;">
				<scroll-view :style="{height: mHeight+'rpx'}" :scroll-y="true" >
					<view class="" v-for="vod in vodList" @tap="playVideo(vod['vod_id'])">
						<view class="flex-row" style="margin-top: 15rpx;">
							<view class="flex-col">
								<view class="">
									<image class="" style="width: 180rpx;height: 240rpx;border-radius: 16rpx;" :src="vod['vod_pic']" mode="aspectFill"></image>
								</view>
							</view>
							<view class="flex-col justify-between" style="margin-left: 20rpx;">
								<view class="flex-row justify-between" style="">
									<view class="flex-col" style="width: 300rpx;">
										<view class="flex-row" style="padding: 10rpx 0;">
											<text class="line_1" style="font-size: 15px;font-weight: 800;width: 300rpx;">{{vod['vod_name']}}</text>
										</view>
										<view class="flex-row">
											<text class="line_1" style="font-size: 13px;color: gray;width: 300rpx;">{{vod['vod_year']}} {{vod['vod_area']}}</text>
										</view>
										<view class="flex-row">
											<text class="line_1" style="font-size: 13px;color: gray;width: 300rpx;">{{vod['vod_remarks']}}</text>
										</view>
										<view class="flex-row" style="" v-if="vod['vod_director'] && vod['vod_director']!=''">
											<text class="line_1" style="font-size: 12px;color: gray;width: 300rpx;">导演：{{vod['vod_director']}}</text>
										</view>
										<view class="flex-row" style="" v-if="vod['vod_actor'] && vod['vod_actor']!=''">
											<text class="line_1" style="font-size: 12px;color: gray;width: 300rpx;">演员：{{vod['vod_actor']}}</text>
										</view>
									</view>
									<view class="flex-col" style="padding: 40rpx 60rpx;">
										<view class="" style="font-size: 12px;padding: 10rpx 20rpx;background-color: #FB7299;color: #fff;border-radius: 50px;">
											立即观看
										</view>
									</view>
								</view>
								<view class="flex-row">
									<view class="" style="padding: 20rpx 0;">
										<text style="color: #f19e38;font-size: 16px;font-weight: 800;">{{vod['vod_douban_score'] ?? '暂无评分'}}</text>
									</view>
								</view>
							</view>
						</view>
					</view>
				</scroll-view>
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
				wd: '',
				vodList: [],
				mHeight: 0,
			};
		},
		onLoad() {
			let system = uni.getSystemInfoSync();
			
			// #ifdef H5
				this.mHeight = system.windowHeight*2 - 120
			// #endif
			// #ifndef H5
				this.mHeight = system.windowHeight*2 - 170
			// #endif
		},
		methods: {
			toIndex() {
				uni.navigateBack()
			},
			playVideo(id) {
				uni.navigateTo({
					url:'/pages/play/play?id='+id
				})
			},
			// 获取视频列表
			getSearchData() {
				this.vodList = []
				uni.$u.http.post('addons/appto/app.php/index/page_vod_lists', {
					wd: this.wd, 
				}).then(res => {
					this.vodList = res.data['data']['list']
				}).catch(err => {
					uni.showToast({
						icon: 'none',
						title: '搜索不到'
					})
				})
			},
		}
	}
</script>

<style lang="scss" scoped>
	.content {
		height: 100%;
	}
	.header {
		.head {
			/* #ifndef H5 */
				height: 80rpx;
			/* #endif */
			/* #ifdef H5 */
				height: 30rpx;
			/* #endif */
		}
	}
</style>

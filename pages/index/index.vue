<template>
	<view class="content flex-col">
		<view class="header flex-col">
			<view class="head" style="background-color: #fff;"></view>
			<view class="flex-row items-center" style="background-color: #fff;height: 70rpx;">
				<view class="" style="padding-left: 30rpx;padding-right: 20rpx;" @tap="toCenter">
					<u-avatar :src="'https://q1.qlogo.cn/g?b=qq&nk='+userInfo['qq']+'&s=640'" size="32"></u-avatar>
				</view>
				<view class="" style="width: 500rpx;" @tap="toSearch">
					<u-search
						placeholder="电影 / 电视剧 / 动漫"
						v-model="keyword"
						height="60rpx"
						:showAction="false"
						searchIconSize="16"
						disabled
						style="font-size: 13px;"
					></u-search>
				</view>
				<view class="" style="padding: 0 30rpx;" @tap="toHistory">
					<u-icon name="clock" color="#9e9e9e" size="25px"></u-icon>
				</view>
			</view>
		</view>
		
		<view class="center flex-col">
			<scroll-view
				class="tabMain"
				:show-scrollbar="false" 
				:scroll-x="true" 
				:scroll-into-view="tabScrollInto">
				<view
					class="tabContent flex-col"
					style="width: 750rpx;background-color: #fff;border-bottom: 1px solid #e7e7e7;">
					<view class="flex-row">
						<view class="tabItem" 
							v-for="(tab, tabItemIndex) in home_cate" 
							:id="'tabb' + tabItemIndex" 
							:data-id="tabItemIndex"
							@click="pressScrollViewItem(tabItemIndex)">
							<text
								class="tabItemTitle"
								:class="tabIndex==tabItemIndex ? 'tabItemTitleActive' : ''"
								style="padding: 0 30rpx;height: 70rpx;"
							>
								{{tab.title}}
							</text>
						</view>
					</view>
					
					<view class="tabLineView flex-row">
						<view class="tabLine tabLineActive"
							:style="{
								left: `${tabItemLineLeft}px`,
								width: `${tabItemLineWidth}px`,
							}"
						>
						</view>
					</view>
				</view>
			</scroll-view>
		</view>
		
		<swiper
			class="childPageView"
			:current="tabIndex"
			@change="swiperChange"
			:style="{height: swiperHeight +'rpx'}">
			<swiper-item class="childPageViewItem"
				v-for="(page, childPageViewItemIndex) in home_cate"
				:key="childPageViewItemIndex">
				<scroll-view
					class="flex-col"
					:scroll-y="true"
					scroll-with-animation
					:style="{height: swiperHeight +'rpx'}" lower-threshold="300">
					<view v-if="page['show']" class="">
						<!-- 轮播图 -->
						<view class="mPd" v-if="page['banners']['length'] > 0" style="margin-bottom: 15rpx;">
							<u-swiper
								:list="page['banners']"
								height="405rpx"
								radius="10rpx"
								indicator
								indicatorMode="line"
								imgMode="aspectFill"
								keyName="image_url"
								showTitle
								circular
								@click="tapSwiper"
							></u-swiper>
						</view>
						
						<!-- 推荐卡片 -->
						<view class="mPd flex-col" v-if="page['sections']['length'] > 0" v-for="(card, index) in page['sections']">
							<view class="flex-col" style="background-color: #fff;border-radius: 20rpx;padding: 30rpx;margin-bottom: 15rpx;">
								<view class="hotMain flex-col">
									<view class="flex-row justify-between" style="padding: 10rpx 0;">
										<text style="font-weight: 600;font-size: 16px;">{{card['title']}}</text>
										<view class="" style="border-radius: 50px;padding: 5rpx 20rpx;"  @tap="toMore(card)">
											<u-icon name="arrow-right" color="#9e9e9e" labelColor="#9e9e9e" label="更多" labelPos="left" labelSize="14px" size="14px"></u-icon>
										</view>
									</view>
									<view class="flex-row flex-wrap" style="padding: 5rpx 0;">
										<view class="videoCardMain flex-col" v-for="video in card['items'].slice(0, 6)" @tap="playVideo(video['vod_id'])">
											<view class="videoCardImgBox">
												<image class="videoCardImg" :src="video['vod_pic']" mode="aspectFill"></image>
											</view>
											<view class="videoCardName line_1">
												{{video['vod_name']}}
											</view>
										</view>
									</view>
								</view>
							</view>
						</view>
					</view>
					<view class="" v-else>
						<u-loading-page :loading="ifShowLoading"></u-loading-page>
					</view>
					<view class="" style="height: 30rpx;text-align: center;">
						
					</view>
				</scroll-view>
				
			</swiper-item>
		</swiper>
		
		<view class="bottom">
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				title: 'Hello',
				keyword: '',
				swiperList: [],
				hotList: [],
				cardList: [],
				tabIndex: 0,
				tabScrollInto:'',
				tabItemLineLeft: 0,
				tabItemLineWidth: 24,
				windowHeight:0,
				windowWidth:0,
				tabListSize:[],
				swiperHeight: 0,
				home_cate: [], // 首页分类
				home_data: [],
				userInfo: {},// 用户信息
				ifShowLoading: true,
			}
		},
		onLoad() {
			uni.getStorage({
				key: 'userInfo',
				success:res=>{
					this.userInfo = res.data
					getApp().globalData.userInfo = res.data
				}
			})
			
			let system = uni.getSystemInfoSync();
			this.windowHeight = system.windowHeight;
			this.windowWidth = system.windowWidth;
			var rpx = this.windowWidth / 750;
			this.swiperHeight = this.windowHeight / rpx - 170;
			// #ifndef H5
				this.swiperHeight = this.swiperHeight - 50
			// #endif
			
			uni.$u.http.setConfig((config) => {
			    config.baseURL = getApp().globalData.api;
			    return config
			})
			this.getHomeCate()
			
		},
		onShow() {
			var u = getApp().globalData.userInfo
			if (!this.userInfo.qq || this.userInfo['qq']!=u['qq'] || this.userInfo['name']!=u['name']) {
				this.userInfo = u
			}
		},
		onPullDownRefresh() {
			this.getHomeData(true)
			setTimeout(function() {
				uni.stopPullDownRefresh();
			}, 500);
		},
		methods: {
			toHistory() {
				uni.navigateTo({
					url:'/pages/history/history'
				})
			},
			toSearch() {
				uni.navigateTo({
					url:'/pages/serach/serach'
				})
			},
			// 点击头像后跳转个人中心
			toCenter() {
			 	uni.switchTab({
			 		url:'/pages/center/center'
			 	})
			},
			toMore(card) {
				uni.navigateTo({
					url: '/pages/more/more?value='+card['section']['value']+'&label='+card['section']['label']+'&mold='+card['section']['mold']
				})
			},
			getHomeCate() {
				uni.$u.http.get('addons/appto/app.php/index/home_cate').then(res => {
					this.home_cate = res.data['data']
					this.getHomeData()
					this.jcsetTabItemDistance()
				}).catch(err => {
					uni.showToast({
						icon: 'error',
						title: '失败'
					})
				})
			},
			getHomeData(ifsx=false) {
				var i = this.tabIndex
				var e = this.home_cate[i]
				if (e['show'] && ifsx==false) {
					return
				}
				if (ifsx) {
					this.home_cate[i]['sections'] = []
					this.home_cate[i]['notices'] = []
					this.home_cate[i]['banners'] = []
					this.home_cate[i]['banner'] = {}
				}
				this.ifShowLoading = true
				uni.$u.http.get('addons/appto/app.php/index/home_data?id='+e['id']).then(res => {
					this.home_cate[i] = {
						...this.home_cate[i],
						...res.data['data'],
						show: true,
					}
					this.ifShowLoading = false
				}).catch(err => {
					uni.showToast({
						icon: 'error',
						title: '失败'
					})
				})
			},
			playVideo(id) {
				uni.navigateTo({
					url:'/pages/play/play?id='+id
				})
			},
			tapSwiper(e) {
				if (e['payload']) {
					getApp().mRoute(e['type'], e['payload'])
				}
			},
			swiperChange(e) {
				let index = e.detail.current
				this.setTabSelect(index)
				this.setTabItemLinePosition(this.tabListSize[this.tabIndex].left, this.tabListSize[this.tabIndex].width);
				this.getHomeData()
			},
			pressScrollViewItem(index) {
				this.setTabSelect(index);
			},
			jcsetTabItemDistance() {
				var queryTabSize = uni.createSelectorQuery().in(this);
				var timer = setInterval(res=>{
					queryTabSize.select('#tabb' + (this.home_cate.length-1)).boundingClientRect(data => {
						if (data) {
							clearInterval(timer);
							this.setTabItemDistance()
						}
					}).exec()
				}, 100)
			},
			setTabItemDistance() {
				var queryTabSize = uni.createSelectorQuery().in(this);
				for (var i = 0; i < this.home_cate.length; i++) {
					queryTabSize.select('#tabb' + [i]).boundingClientRect()
				}
				queryTabSize.exec(rects => {
					rects.forEach((rect) => {
						this.tabListSize[rect.dataset.id] = rect
						
					})
					this.setTabItemLinePosition(this.tabListSize[this.tabIndex].left, this.tabListSize[this.tabIndex].width);
				});
			},
			setTabItemLinePosition(left, width) {
				this.tabItemLineLeft = left + width/2 - 12
			},
			setTabSelect(index) {
				this.tabScrollInto = 'tabb' + index;
				this.tabIndex = index;
			},
		}
	}
</script>

<style scoped lang="scss">
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
	.videoCardMain {
		width: 220rpx;
		padding: 8rpx 0;
		.videoCardImgBox {
			text-align: center;
			.videoCardImg {
				width: 200rpx;
				height: 280rpx;
				border-radius: 16rpx;
			}
		}
		.videoCardName {
			font-size: 12px;
			padding: 0 15rpx;
			font-weight: 600;
			text-align: center;
		}
		.videoCardRate {
			padding: 3rpx 15rpx;
		}
	}
	.tabMain {
		width: 750rpx;
		display: flex;
		flex-direction: row;
		white-space: nowrap;
	}
	.tab {
		width: 750rpx;
		display: flex;
		flex-direction: row;
		white-space: nowrap;
	}
	
	/* 隐藏滚动条 */
	
	.tab ::-webkit-scrollbar {
		display: none;
		width: 0 !important;
		height: 0 !important;
		-webkit-appearance: none;
		background: transparent;
	}
	
	.tabLineView {
		position: relative;
		height: 2px;
	}
	
	.tabLine {
		position: absolute;
		height: 2px;
		top: 0;
		bottom: 0;
		width: 0;
		background-color: $BLBL;
	}
	
	.tabLineActive {
		transition-duration: 0.2s;
		transition-property: left;
	}
	
	.childPageView {
		display: flex;
		flex-direction: column;
	}
	
	.tabItemTitle {
		color: #505050;
		font-size: 15px;
		height: 50rpx;
		line-height: 50rpx;
		display: flex;
		flex-wrap: nowrap;
		align-items: center;
		justify-content: center;
	}
	
	.tabItemTitleActive {
		color: $BLBL;
	}
	
	.childPageViewItem {
		flex: 1;
		flex-direction: column;
		margin-top: 10rpx;
	}
</style>

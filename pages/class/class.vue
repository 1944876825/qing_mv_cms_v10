<template>
	<view class="content flex-col" style="padding: 0 20rpx;">
		<view class="header">
			<view class="head">
				
			</view>
			<view class="">
				
			</view>
		</view>
		<u-sticky>
			<view class="" style="background-color: #fff;border-radius: 20rpx;padding: 20rpx 10rpx;">
				<view class="">
					<text style="padding: 10rpx;font-size: 16px;font-weight: 1000;">分类</text>
					<view class="" style="margin-top: 30rpx;">
						<!-- 主分类 -->
						<scroll-view class="flex-row hideScrollbar" :scroll-x="true" :show-scrollbar="false">
							<view class="flex-row">
								<view
									v-for="(vclass, vclIndex) in vodClassList"
									class="typeDef"
									:class="vodClass==vclIndex ? 'typeAct' : ''"
									@tap="chooseClass(0, vclIndex)"
									>
									{{vclass}}
								</view>
							</view>
						</scroll-view>
						<!-- 扩展分类 -->
						<scroll-view class="flex-row hideScrollbar" :scroll-x="true" :show-scrollbar="false">
							<view class="flex-row">
								<view
									v-for="(vextend, vexIndex) in vodExtendList[vodClass]"
									class="typeDef"
									:class="vodExtend==vexIndex ? 'typeAct' : ''"
									@tap="chooseClass(1, vexIndex)"
									>
									{{vextend}}
								</view>
							</view>
						</scroll-view>
						<!-- 年 -->
						<scroll-view class="flex-row hideScrollbar" :scroll-x="true" :show-scrollbar="false">
							<view class="flex-row">
								<view
									v-for="(vyear, vyeIndex) in vodYearList"
									class="typeDef"
									:class="vodYear==vyeIndex ? 'typeAct' : ''"
									@tap="chooseClass(2, vyeIndex)"
									>
									{{vyear}}
								</view>
							</view>
						</scroll-view>
						<!-- 排序 -->
						<scroll-view class="flex-row hideScrollbar" :scroll-x="true" :show-scrollbar="false">
							<view class="flex-row">
								<view
									v-for="(vpx, vpxIndex) in vodPxList"
									class="typeDef"
									:class="vodPx==vpxIndex ? 'typeAct' : ''"
									@tap="chooseClass(3, vpxIndex)"
									>
									{{vpx}}
								</view>
							</view>
						</scroll-view>
					</view>
				</view>
			</view>
		</u-sticky>
		<view class="center">
			<view class="" style="background-color: #fff;margin-top: 15rpx;border-radius: 20rpx;padding: 20rpx;">
				<view class="flex-row" style="padding: 10rpx 0;flex-wrap: wrap;">
					<view
						v-for="vod in vodList"
						class="videoCardMain flex-col"
						@tap="playVideo(vod['vod_id'])">
						<view class="videoCardImgBox">
							<image class="videoCardImg" :src="vod['vod_pic']" mode="aspectFill"></image>
						</view>
						<view class="videoCardName line_1">
							{{vod['vod_name']}}
						</view>
						<view class="videoCardRate">
							<u-rate readonly allowHalf v-model="vod['vod_douban_score']" activeColor="#f19e38" size="13px"></u-rate>
						</view>
					</view>
				</view>
				<view class="" style="padding: 15rpx;" @tap="getVodList">
					<u-loadmore :status="status" />
				</view>
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
				vodClass: 0,
				vodExtend: 0,
				vodYear: 0,
				vodPx: 0,
				vodClassList: ['电影', '电视剧', '综艺', '动漫'],
				vodExtendList: [
					['动作','喜剧','爱情','恐怖','科幻','剧情','战争','犯罪','奇幻','武侠','冒险','恐怖','悬疑','惊悚','古装','历史','运动'],
					['古装','喜剧','犯罪','动作','奇幻','剧情'],
					['科幻','热血','推理','搞笑','冒险','萝莉','校园','动作','机战','运动','战争','少年','少女','社会','原创','亲子','益智','励志'],
					['选秀','情感','访谈','播报','旅游','音乐','美食','纪实','曲艺','生活','游戏','财经','求职']
				],
				vodYearList: ['2023','2022','2021','2020','2019','2018','2017','2016','2015','2014','2013','2012','2011','2010','2009','2008','2007','2006','2005','2004','2003','2002','2001','2000'],
				vodPxList: ['按时间', '按评分', '按年份'],
				vodList: [],
				page: 1,
				status: 'loadmore',
			};
		},
		onLoad() {
			this.getVodList()
		},
		onPullDownRefresh() {
			this.page = 1
			this.status = 'loadmore'
			this.vodList = []
			this.getVodList()
			setTimeout(function() {
				uni.stopPullDownRefresh();
			}, 500);
		},
		onReachBottom() {
			this.getVodList()
		},
		methods: {
			playVideo(id) {
				uni.navigateTo({
					url:'/pages/play/play?id='+id
				})
			},
			// 获取视频列表
			getVodList() {
				if (this.status != 'nomore') {
					uni.$u.http.post('addons/appto/app.php/index/page_vod_lists', {
						page: this.page, 
						type: this.vodClass,
						class: this.vodExtendList[this.vodClass][this.vodExtend], 
						year: this.vodYearList[this.vodYear],
						order: this.vodPxList[this.vodPx]
					}).then(res => {
						this.vodList.push(...res.data['data']['list'])
						this.page++
						if (this.vodList.length >= res.data['data']['total']) {
							this.status = 'nomore'
						}
					}).catch(err => {
						uni.showToast({
							icon: 'none',
							title: '获取失败'
						})
					})
				}
			},
			// 选择分类
			chooseClass(index, data) {
				// 主分类
				if (index == 0) {
					this.vodClass = data
				}
				// 扩展分类
				if (index == 1) {
					this.vodExtend = data
				}
				// 年份
				if (index == 2) {
					this.vodYear = data
				}
				// 评分
				if (index == 3) {
					this.vodPx = data
				}
				this.vodList = []
				this.getVodList()
			},
		}
	}
</script>

<style lang="scss" scoped>
	.head {
		/* #ifndef H5 */
			height: 80rpx;
		/* #endif */
		/* #ifdef H5 */
			height: 30rpx;
		/* #endif */
	}
	.typeDef {
		font-size: 14px;
		padding: 5rpx 15rpx;
		border-radius: 10rpx;
		color: #000;
		margin: 5rpx;
		font-weight: 600;
	}
	.typeAct {
		background-color: $BLBL;
		color: #fff;
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
			text-align: center;
		}
		.videoCardRate {
			padding: 3rpx 15rpx;
		}
	}
</style>

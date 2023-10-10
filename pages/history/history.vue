<template>
	<view>
		<view class="head" v-if="mvlist['length'] == 0">
			啥也没有...
		</view>
		<view class="center" v-else>
			<view class="hbox flex-row" v-for="item in mvlist" @click="play(item)">
				<view class="himg fl">
					<image :src="item['vod_pic']" mode="aspectFill"></image>
				</view>
				<view class="hdet fr">
					<view class="hdett flex-col">
						<text class="line_1" style="font-weight: 800;font-size: 14px;">{{item['vod_name']}}</text>
						<text class="line_1" style="color: gray;font-size: 12px;margin-top: 10rpx;">{{item['juji_name']}}</text>
					</view>
					<view class="hdetb">
						点击播放
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				mvlist:[]
			};
		},
		onLoad() {
			this.gethistory()
		},
		onPullDownRefresh() {
			this.mvlist = [];
			this.gethistory()
			setTimeout(function() {
				uni.stopPullDownRefresh();
			},500);
		},
		methods:{
			gethistory() {
				uni.getStorage({
					key:'history',
					success:res=>{
						this.mvlist = res.data;
					}
				})
			},
			play(hvod) {
				uni.navigateTo({
					url:'../../pages/play/play?id=' + hvod['vod_id'] + '&xl='+hvod['xl_index']+'&juji='+hvod['juji_index']
				})
			}
		}
	}
</script>

<style lang="scss" scoped>
	.head {
		text-align: center;
		color: $BLBL;
		font-size: 16px;
	}
	.center {
		padding: 0 15rpx;
		.hbox {
			background-color: #fff;
			margin: 15rpx 0;
			padding: 20rpx;
			border-radius: 15rpx;
			.himg {
				width: 40%;
				height: 160rpx;
				image {
					width: 100%;
					height: 160rpx;
					border-radius: 20rpx;
				}
			}
			.hdet {
				width: 60%;
				height: 160rpx;
				position: relative;
				view {
					margin-left: 30rpx;
					font-size: 14px;
				}
				.hdetb {
					color: #9c9c9c;
					position: absolute;
					font-size: 12px;
					bottom: 0;
				}
			}
		}
	}
</style>

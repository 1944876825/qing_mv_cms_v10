<template>
	<view class="myVideoPlayer">
		<view class="" style="width: 750rpx;background-color: #000;">
			<view class="" style="width: 750rpx;">
				<video
					id="myVideo"
					:autoplay="videoPlayUrl ? true : false"
					:controls="false"
					:show-fullscreen-btn="false"
					:show-center-play-btn="false"
					:show-play-btn="false"
					:enable-progress-gesture="false"
					:vslide-gesture-in-fullscreen="false"
					:header="videoPlayHeader"
					:src="videoPlayUrl"
					style="position: relative;width: 750rpx;"
					@play="onVideoPlayBack"
					@pause="onVideoPauseBack"
					@ended="onVideoEndBack"
					@timeupdate="onTimeUpdate"
					@fullscreenchange="onFullScreenChange"
					>
					
					<!-- 遮罩层 -->
					<cover-view
						style="width: 750rpx;height: 470rpx;position: absolute;left: 0;top: 0;right: 0;bottom: 0;">
						<view 
							@touchstart.stop="onTouchStart"
							@touchend.stop="onTouchEnd"
							@touchmove="onTouchMove"
							style="width: 750rpx;height: 470rpx;position: absolute;left: 0;top: 0;right: 0;bottom: 0;"
							>
						</view>
					</cover-view>
					
					<!-- 播放器顶部 -->
					<cover-view
						class="cmShow"
						:class="isShowCM ? '' : 'cmShowActive'"
						style="z-index: 999;width: 750rpx;position: absolute;top: 20rpx;">
						<view v-if="isFull && !isLock" class="" style="width: 750rpx;display: flex;flex-direction: row;justify-content: space-between;align-items: center;padding: 0 20rpx;" >
							<view style="display: flex;flex-direction: row;justify-content: space-between;align-items: center;">
								<u-icon name="arrow-left" color="#fff" size="20" @click="fullScreen"></u-icon>
								<text style="color: #fff;padding: 0 10rpx;">{{videoTitle + ' ' + videoTopTitle}}</text>
							</view>
							<view style="display: flex;flex-direction: row;justify-content: space-between;align-items: center;">
								<u-icon name="more-dot-fill" color="#fff" size="20"></u-icon>
							</view>
						</view>
						<view v-if="!isFull" class="" style="width: 750rpx;display: flex;flex-direction: row;justify-content: space-between;align-items: center;padding: 0 20rpx;" >
							<view style="display: flex;flex-direction: row;justify-content: space-between;align-items: center;">
								<u-icon name="arrow-left" color="#fff" size="20" @click="exitPlay"></u-icon>
								<text style="color: #fff;padding: 0 10rpx;">{{videoTitle + ' ' + videoTopTitle}}</text>
							</view>
						</view>
					</cover-view>
					
					<!-- 锁 -->
					<cover-view
						v-if="isFull && isShowCM"
						style="z-index: 999;width: 750rpx;position: absolute;top: 150rpx;left: 50rpx;">
						<view class="" :style="{width: (windowWidth/2-10)+'px'}" >
							<text class="iconfont" style="color: #fff;font-size: 25px;" @tap="isLock=!isLock">{{isLock ? '&#xe600;' : '&#xe601;'}}</text>
						</view>
					</cover-view>
					
					<!-- 亮度 音量设置 -->
					<cover-view v-if="isFull && !isLock && (TouchMoveType=='rightYingLiang' || TouchMoveType == 'leftLiangDu')" style="width: 750rpx;position: absolute;top: 60rpx;">
						<view class="flex-row justify-center" style="padding: 0 300rpx;">
							<!-- 滑动条 太丑了 暂弃 -->
							<!-- <slider
								block-size="0"
								step="5"
								activeColor="#FB7299"
								:value="TouchMoveType == 'leftLiangDu' ? SliderLiangduValue : SliderYingliangValue"
								/> -->
							<text style="color: #FB7299;font-size: 16px;font-weight: 800;">{{TouchMoveType == 'leftLiangDu' ? SliderLiangduValue : SliderYingliangValue}}</text>
						</view>
					</cover-view>
					
					<cover-view
						class="cmShow"
						:class="isShowCM ? '' : 'cmShowActive'"
						style="z-index: 999;width: 750rpx;position: absolute;bottom: 10rpx;">
						<!-- 全屏底部控制面板 -->
						<view v-if="isFull && !isLock" style="width: 750rpx;display: flex;flex-direction: column;justify-content: space-between;align-items: center;">
							<view class="flex-row" style="width: 750rpx;justify-content: space-between;align-items: center;padding-left: 30rpx;padding-right: 20rpx;">
								<text style="color: #fff;font-size: 12px;">{{videoCurrentTime}}</text>
								<slider
									style="padding: 0 15rpx;"
									block-size="12"
									activeColor="#FB7299"
									:value="sliderValue"
									@change="sliderChange"/>
								<text style="color: #fff;font-size: 12px;">{{videoDuration}}</text>
							</view>
							<view class="" style="height: 20rpx;"></view>
							<view class="" style="width: 750rpx;display: flex;flex-direction: row;justify-content: space-between;align-items: center;padding-left: 30rpx;padding-right: 20rpx;">
								<view class="" style="display: flex;flex-direction: row;align-items: center;">
									<u-icon :name="isPlay ? 'pause' : 'play-right-fill'" color="#fff" size="30" @tap="videoPlay"></u-icon>
									<view class="" style="width: 20rpx;"></view>
									<text class="iconfont" style="color: #fff;font-size: 25px;" @tap="fullScreen">&#xe7ba;</text>
								</view>
								<view class="" style="display: flex;flex-direction: row;">
									<text style="color: #fff;padding: 0 10rpx;" @tap="showChooseJuJi">选集</text>
									<text style="color: #fff;padding: 0 10rpx;" @tap="showChooseBeiSu">倍速</text>
									<!-- <text style="color: #fff;padding: 0 10rpx;" @tap="showChooseJK">线路</text> -->
								</view>
							</view>
						</view>
						<!-- 未全屏底部控制面板 -->
						<view v-if="!isFull" style="width: 750rpx;display: flex;flex-direction: row;justify-content: space-between;align-items: center;padding: 20rpx;">
							<u-icon :name="isPlay ? 'pause' : 'play-right-fill'" color="#fff" size="20"  @tap="videoPlay"></u-icon>
							<slider
								style="padding: 0 15rpx;"
								block-size="12"
								activeColor="#FB7299"
								:value="sliderValue"
								@change="sliderChange"/>
							<text style="color: #fff;font-size: 12px;">{{videoCurrentTime}}/{{videoDuration}}</text>
							<view class="" style="width: 20rpx;"></view>
							<text class="iconfont" style="color: #fff;font-size: 20px;" @tap="fullScreen">&#xe629;</text>
						</view>
					</cover-view>
					
					<!-- 剧集 -->
					<cover-view v-if="isFull"
						class="noActive"
						:class="isShowJuJi ? 'Active' : ''"
						style="position: absolute;top: 0;width: 750rpx;background-color: rgba(40,40,40,.5);">
						<view class="" v-if="isShowJuJi" 
							:style="{height: windowWidth + 'px'}"
							style="display: flex;flex-direction: row;width: 750rpx;justify-content: space-between;"
							@tap="hideShowJuJi"
							>
							<view class="" style="display: flex;flex-direction: column;align-items: center;width: 300rpx;padding-top: 20rpx;">
								<text style="color: #fff;font-size: 16px;width: 260rpx;">选集</text>
							</view>
							<scroll-view
								:scroll-y="true"
								:style="{height: windowWidth + 'px'}"
								style="display: flex;flex-direction: column;padding: 10rpx;align-items: center;width: 300rpx;">
								<view class="" 
									v-for="(vod, index) in videoList"
									@tap="chooseJuJi(index)">
									<view
										v-if="vod"
										:style="{border: currentJuJi==index ? '1px solid #FB7299' : '1px solid #cdcdcd'}"
										style="width: 260rpx;padding: 10rpx;border-radius: 5rpx;margin: 3rpx 0;">
										<text class="" :style="{color: currentJuJi==index ? '#FB7299' : '#fff'}" >{{vod['name']}}</text>
									</view>
								</view>
							</scroll-view>
						</view>
					</cover-view>
					
					<!-- 倍速 -->
					<cover-view v-if="isFull"
						class="noActive"
						:class="isShowBeiSu ? 'Active' : ''"
						style="position: absolute;top: 0;width: 750rpx;background-color: rgba(40,40,40,.5);">
						<view class="" v-if="isShowBeiSu" 
							:style="{height: windowWidth + 'px'}"
							style="display: flex;flex-direction: row;width: 750rpx;justify-content: space-between;"
							@tap="hideShowBeiSu"
							>
							<view class="" style="display: flex;flex-direction: column;align-items: center;width: 300rpx;padding-top: 20rpx;">
								<text style="color: #fff;font-size: 16px;width: 260rpx;">倍速</text>
							</view>
							<scroll-view
								:scroll-y="true"
								:style="{height: windowWidth + 'px'}"
								style="display: flex;flex-direction: column;padding: 10rpx;align-items: center;width: 300rpx;">
								<view
									class=""
									v-for="rate in rateList"
									@tap="chooseBeiSu(rate)"
									:style="{border: currentRate==rate ? '1px solid #FB7299' : '1px solid #cdcdcd'}"
									style="width: 260rpx;padding: 10rpx;border-radius: 5rpx;margin: 3rpx 0;">
									<text class="" :style="{color: currentRate==rate ? '#FB7299' : '#fff'}">{{rate}}</text>
								</view>
							</scroll-view>
						</view>
					</cover-view>
					
					<!-- 接口 此处可做画质选择 线路选择等 暂时用不上 -->
					<!-- <cover-view v-if="isFull"
						class="noActive"
						:class="isShowJK ? 'Active' : ''"
						style="position: absolute;top: 0;width: 750rpx;background-color: rgba(40,40,40,.5);">
						<view class="" v-if="isShowJK" 
							:style="{height: windowWidth + 'px'}"
							style="display: flex;flex-direction: row;width: 750rpx;justify-content: space-between;"
							@tap="hideShowJK"
							>
							<view class="" style="display: flex;flex-direction: column;align-items: center;width: 300rpx;padding-top: 20rpx;">
								<text style="color: #fff;font-size: 16px;width: 260rpx;">线路</text>
							</view>
							<scroll-view
								:scroll-y="true"
								:style="{height: windowWidth + 'px'}"
								style="display: flex;flex-direction: column;padding: 10rpx;align-items: center;width: 300rpx;">
								<view
									class=""
									v-for="(jk,index) in playUrlArr"
									@tap="chooseJK(index)"
									:style="{border: currentJK==jk ? '1px solid #FB7299' : '1px solid #cdcdcd'}"
									style="width: 260rpx;padding: 10rpx;border-radius: 5rpx;margin: 3rpx 0;">
									<text class="" :style="{color: currentJK==index ? '#FB7299' : '#fff'}">{{jk['title']}}</text>
								</view>
							</scroll-view>
						</view>
					</cover-view> -->
				
				</video>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		name:"myVideoPlayer",
		props:['modelValue', 'videoTitle', 'videoList', 'type', 'pwd'],
		emits: ['update:modelValue'],
		data() {
			return {
				windowWidth:0,// 屏幕宽度
				windowHeight:0,
				videoContext: {},
				isFull: false,// 是否是全屏
				isPlay: false,// 是否在播放
				isShowJuJi: false,// 是否展示剧集选择
				isShowBeiSu: false,// 是否展示倍速选择
				isShowJK: false,// 是否展示接口选择
				isShowCM: true,// 是否展示控制面板
				ifShowNext: true,// 是否显示下一集按钮
				videoDuration: '00.00',
				videoCurrentTime: '00.00',
				videoDurationNum: 0,
				sliderValue: 0,
				videoPlayUrl: '',// 当前播放链接
				playUrlArr: [],//播放链接数组
				currentXL: 0,// 当前播放线路
				currentJuJi: this.modelValue, // 当前播放剧集
				currentJK: 0,// 当前接口
				rateList: ['0.5','0.8','1.0','1.25','1.5','2.0'],
				currentRate: '1.0',
				videoTopTitle: '',
				lastTapDiffTime: {},// 记录点击时间间隔
				TouchMoveType: '',
				oldTouces: {},
				zuoyou_Width: 0,
				DataY: 0,
				SliderLiangduValue: 0,// 系统亮度
				SliderYingliangValue: 0,// 系统音量
				isfangdou: false,
				isTouchMove: false,
				isClickOver: false,// 是否点击结束
				isPressRate: false,// 是否长按倍速
				oldValue: {
					LiangduValue: 0,
					YingLiangValue: 0,
				},
				isLock:false,// 锁屏
				panUserInfo: {},// 账号信息
				share_token: '',//阿里share_token
				videoPlayHeader: {},
				Aliyun: null,
			};
		},
		created() {
			// #ifdef APP-NVUE
				var domModule = weex.requireModule('dom');
				domModule.addRule('fontFace', {
					fontFamily: 'fontFamilyName',
					src: "url('https://at.alicdn.com/t/c/font_3867346_yxkt1x9z21e.ttf')",
				});
			// #endif
			let system = uni.getSystemInfoSync();
			this.windowWidth = system.windowWidth;
			this.windowHeight = system.windowHeight;
		},
		beforeMount() {
			this.videoContext = uni.createVideoContext('myVideo')
		},
		mounted() {
			this.lessPlay()
		},
		watch: {
			modelValue(val) {
				if (val == null) {
					val = this.currentJuJi
				}
				this.chooseJuJi(val)
			},
		},
		methods: {
			lessPlay() {
				var timer = setInterval(res=>{
					if (this.videoList && this.videoList[this.currentJuJi]) {
						this.chooseJuJi(this.currentJuJi)
						clearInterval(timer);
					}
				}, 500)
			},
			// 解析
			async jkParse() {
				var f = this.videoList[this.currentJuJi]
				var data = {
					from: f['from'],
					url: f['url']
				}
				uni.$u.http.post('addons/appto/app.php/index/page_parsing_config_v2', data).then(res => {
					if (res.data['data']['url']) {
						this.videoPlayUrl = res.data['data']['url']
					} else {
						uni.showToast({
							icon: 'none',
							title: '解析失败'
						})
					}
				}).catch(err => {
					uni.showToast({
						icon: 'none',
						title: '解析失败'
					})
				})
			},
			// 隐藏播放控制
			hideShowCM() {
				if (!this.isShowBeiSu && !this.isShowJK && !this.isShowJuJi) {
					if (this.isPlay) {
						this.isShowCM = !this.isShowCM
					} else {
						this.isShowCM = true
					}
				}
			},
			// 选择线路
			chooseJK(index) {
				console.log(this.videoPlayHeader);
				this.currentJK = index
				this.videoPlayUrl = this.playUrlArr[index]['url']
				uni.showToast({
					icon:'none',
					title: this.playUrlArr[index]['title']
				})
			},
			// 展示接口
			showChooseJK() {
				this.isShowCM = false
				this.isShowJK = true
			},
			// 隐藏接口
			hideShowJK() {
				this.isShowJK = false
				this.isShowCM = true
			},
			// 选择倍速
			chooseBeiSu(rate) {
				this.currentRate = rate
				this.videoContext.playbackRate(parseFloat(rate))
				uni.showToast({
					icon:'none',
					title:rate
				})
			},
			// 展示倍速
			showChooseBeiSu() {
				this.isShowCM = false
				this.isShowBeiSu = true
			},
			// 隐藏倍速
			hideShowBeiSu() {
				this.isShowBeiSu = false
				this.isShowCM = true
			},
			// 选集
			chooseJuJi(index) {
				if (this.videoList && this.videoList[index]) {
					this.currentJuJi = index
					// 同时修改双向绑定的值
					this.$emit("update:modelValue", this.currentJuJi);
					this.videoTopTitle = this.videoList[index]['name']
					this.jkParse()
				}
			},
			// 展示选集
			showChooseJuJi() {
				this.isShowCM = false
				this.isShowJuJi = true
			},
			// 隐藏选集
			hideShowJuJi() {
				this.isShowJuJi = false
				this.isShowCM = true
			},
			// 进/退全屏事件回调
			onFullScreenChange(e) {
				this.isFull = e.detail.fullScreen
				if (this.isFull == false) {
					this.isShowJuJi = false
					this.isShowCM = true
				}
			},
			// 退出页面
			exitPlay() {
				uni.navigateBack()
			},
			// 改变进度条回调
			sliderChange(e) {
				this.videoContext.seek(e.detail.value / 100 * this.videoDurationNum)
			},
			// 播放进度改变回调
			onTimeUpdate(e) {
				this.videoDuration = this.getForMatTime(e.detail.duration)
				this.videoCurrentTime = this.getForMatTime(e.detail.currentTime)
				this.sliderValue = e.detail.currentTime / e.detail.duration * 100
				this.videoDurationNum = e.detail.duration
			},
			// 格式化播放时间
			getForMatTime(time) {
				var dz = parseInt(time)
				if (dz < 60) {
					if (dz < 10) {
						dz = `0${dz}`
					}
					d = `00:${dz}`
				} else {
					var dm = parseInt(dz / 60)
					var dzy = dz % 60
					if (dzy < 10) {
						dzy = `0${dzy}`
					}
					if (dm < 60) {
						if (dm < 10) {
							dm = `0${dm}`
						}
						d = `${dm}:${dzy}`
					} else {
						var dh = parseInt(dm / 60)
						var dmy = dm % 60
						if (dmy < 10) {
							dmy = `0${dmy}`
						}
						d = `${dh}:${dmy}:${dzy}`
					}
				}
				return d
			},
			// 进入/退出全屏
			fullScreen() {
				if (this.isFull) {
					this.videoContext.exitFullScreen()
				} else {
					this.videoContext.requestFullScreen()
				}
				this.isFull = !this.isFull
			},
			// 播放/暂停视频
			videoPlay() {
				if (this.isPlay) {
					this.videoContext.pause()
				} else {
					this.videoContext.play()
				}
				this.isPlay = !this.isPlay
			},
			// 视频播放回调
			onVideoPlayBack() {
				this.isPlay = true
				if (this.isShowCM == true) {
					setTimeout(res=> {
						this.isShowCM = false
					}, 2000);
				}
			},
			// 视频暂停回调
			onVideoPauseBack() {
				this.isPlay = false
				this.isShowCM = true
			},
			// 视频结束回调
			onVideoEndBack() {
				this.isPlay = false
				this.isShowCM = true
				this.videoContext.stop()
			},
			/* 获取屏幕亮度值 */
			getScreenLiangdu(){
				// #ifndef H5
				uni.getScreenBrightness({
					success: (res) => {
						// 获取当前屏幕亮度值
						var Value = res.value
						if (Value == Number(-1)){
							Value = 0
						}
						// 设置亮度进度条的值
						this.SliderLiangduValue = Math.ceil(Value*100);// 每0.1值等于10进度值
					}
				})
				// #endif
			},
			/* 获取设备的系统音量 */
			getDeviceVolume(){
				// #ifdef APP-PLUS
				// 获取当前音量值
				let Value = plus.device.getVolume()
				if (Value == Number(-1)){
					Value = 0
				}
				// 设置音量进度条的值
				this.SliderYingliangValue = Math.ceil(Value*100);// 每0.1值等于10进度值
				// #endif
			},
			onTouchMove(event) {
				if (this.isLock) return false
				this.isTouchMove = true
				// 手指触摸开始时的坐标
				let oldTouces = this.oldTouces
				// 手指触摸移动时的坐标
				let newTouces = event.changedTouches[0]
				// 记录手指触摸移动时的坐标
				this.newTouces = newTouces
				// 左右滑动时的距离 [用于视频进度控制]
				var dataX = newTouces.pageX - oldTouces.pageX
				// 上下滑动时的距离 [用于音量和亮度控制]
				var dataY = newTouces.pageY - oldTouces.pageY
				// 记录Y轴的滑动距离，如果手指只是点击后离开，就是0，不执行更新音量和亮度。用于手指触摸结束判断
				this.DataY = dataY
				
				// 判断是否防抖结束
				if (this.isfangdou == true){
					// 手指触摸开始时的坐标 = 移动的当前坐标
					let oldTouces = newTouces
				}
				if (this.isFull){
					// 进入全屏时
					this.zuoyou_Width = Number(this.windowHeight/2)
				}else {
					// 退出全屏时
					this.zuoyou_Width = Number(this.windowWidth/2)
				}
				
				// 手势类型
				if (!this.TouchMoveType){
					// 播放进度
					
					// 进度条控制 (如果手指左右滑动大于20则设置为“调节视频进度”)
					this.TouchMoveType = Math.abs(newTouces.pageX - oldTouces.pageX ) >= 20 ? 'conterVideoJinDu' : null
					// 左边/亮度控制 (如果手指上下滑动并且X“横”坐标小于或等于this.zuoyou_Width时,则设置为“调节亮度”)
					this.TouchMoveType = Math.abs(newTouces.pageY - oldTouces.pageY ) > 35 && newTouces.pageX < this.zuoyou_Width ? 'leftLiangDu' : this.TouchMoveType
					// 右边/音量控制 (如果手指上下滑动并且X“横”坐标大于this.zuoyou_Width时,则设置为“调节音量”)
					this.TouchMoveType = Math.abs(newTouces.pageY - oldTouces.pageY ) > 35 && newTouces.pageX > this.zuoyou_Width ? 'rightYingLiang' : this.TouchMoveType
					// 已有手指类型
					if (this.TouchMoveType){
						// 开启触摸时显示的面板
						// this.isTouchFull = true
						// 存入moveX （X轴的偏移距离）
						this.moveX = ( newTouces.pageX - oldTouces.pageX )
						// 存入moveY （Y轴的偏移距离）
						this.moveY = ( newTouces.pageY - oldTouces.pageY )
						// 开启防抖
						this.isfangdou = true
					}
				}else if (this.TouchMoveType == 'leftLiangDu') {
					// 亮度
					
					// Y轴的偏移距离
					let	curr_pageY = this.moveY > 0 ? this.moveY : 0
					
					// 灵感值 = 用户设置的灵感度值 
					let sensitivity = Number(0.3)
					
					// 计算手指滑动的距离值 + 触摸开始时获取的系统亮度值
					this.newTouchLiangduValue = ((oldTouces.pageY - newTouces.pageY - curr_pageY) * sensitivity) + this.oldValue.LiangduValue
					// 亮度进度条满值 = 100
					this.newTouchLiangduValue = this.newTouchLiangduValue > 100 ? 100 : this.newTouchLiangduValue
					this.newTouchLiangduValue = this.newTouchLiangduValue < 0? 0 : this.newTouchLiangduValue
					// 相等时重置oldTouces
					if(this.newTouchLiangduValue==this.oldValue.LiangduValue) this.oldTouces = newTouces
					// 去除小数点后的所有数字
					this.SliderLiangduValue = parseInt(this.newTouchLiangduValue)
					// 执行更新系统亮度
					var Liangdu = this.SliderLiangduValue/100
					if (Liangdu>=1){
						Liangdu = 1 //亮度值范围0~1
						this.SliderLiangduValue = 100 //亮度进度条的值
					}else if (Liangdu<=0){
						Liangdu = 0 //亮度值范围0~1
						this.SliderLiangduValue = 0 //亮度进度条的值
					}
					// 定义入口参数
					let options = {value:Liangdu}
					// 设置屏幕亮度值
					uni.setScreenBrightness({
						value: Liangdu
					})
				} else if (this.TouchMoveType == 'rightYingLiang' && this.isFull==true && dataY!=0){
					// 音量
					
					// Y轴的偏移距离
					let	curr_pageY = this.moveY > 0 ? this.moveY : 0
					// 灵感值 = 用户设置的灵感度值 
					let sensitivity = Number(0.3)
					// 计算手指滑动的距离值 + 触摸开始时获取的系统音量值
					this.newTouchYingliangValue = ((oldTouces.pageY - newTouces.pageY - curr_pageY) * sensitivity) + this.oldValue.YingLiangValue
					// 音量进度条满值 = 100
					this.newTouchYingliangValue = this.newTouchYingliangValue > 100? 100 : this.newTouchYingliangValue
					this.newTouchYingliangValue = this.newTouchYingliangValue < 0? 0 : this.newTouchYingliangValue
					// 相等时重置oldTouces
					if(this.newTouchYingliangValue==this.oldValue.YingLiangValue) this.oldTouces = newTouces
					// 去除小数点后的所有数字
					this.SliderYingliangValue = parseInt(this.newTouchYingliangValue)
					var YingLiang = Math.abs(this.SliderYingliangValue/100)
					if (YingLiang <= 0.1){
						YingLiang = 0.1 // 最低音量，不能设置0，0会让手机振动。0.09也不行，最低0.1
					}
					if (YingLiang>=1){
						YingLiang = 1 //亮度值范围0~1
						this.SliderYingliangValue = 100 //亮度进度条的值
					}
					// 执行更新系统音量
					// #ifdef APP-PLUS
						// 设置当前音量值
						plus.device.setVolume(YingLiang)
					// #endif
				}else if (this.TouchMoveType == 'conterVideoJinDu'){
					// X轴的偏移距离	
					let	curr_pageX = this.moveX > 0 ? this.moveX : 0
					// n分钟 = 总时长(秒)/60(秒)
					let nfenzhong = parseInt(this.timeUpdate[0].duration / 60)
					// 精准灵感值 = 用户设置的灵感度值 * n分钟
					let sensitivity = Number(this.Config.sensitivity.X * nfenzhong)
					// 计算公式
					this.newTouchCurrentTime = ((newTouces.pageX - oldTouces.pageX - curr_pageX) * sensitivity) + (100/this.timeUpdate[0].duration + this.oldValue.currentTime*1)
					// 三元运算
					this.newTouchCurrentTime = this.newTouchCurrentTime >= this.timeUpdate[0].duration? this.timeUpdate[0].duration : this.newTouchCurrentTime
					this.newTouchCurrentTime = this.newTouchCurrentTime <= 0? 0 : this.newTouchCurrentTime
					// 相等时重置oldTouces (保留后6位小数，和官方进度回调，小数位数保持一致)
					if (this.newTouchCurrentTime == this.timeUpdate[0].currentTime) this.oldTouces = newTouces
					
					// 用来拦截进度条拖滑和触摸改变进度时两个事件同时触发
					if (this.changingSliderX==false){
						
					} else {
						
					}
				}
			},
			// 判断单双击
			async onTouchStart(event) {
				this.isClickOver = false
				
				// 获取系统音量的值，写入音量进度条
				await this.getDeviceVolume()
				// 储存当前获取的系统音量值
				this.oldValue.YingLiangValue = this.SliderYingliangValue
				// 获取系统亮度的值，写入亮度进度条
				await this.getScreenLiangdu()
				// 储存当前获取的系统亮度值
				this.oldValue.LiangduValue = this.SliderLiangduValue
				// 手指按下时的X坐标和Y坐标
				this.oldTouces = event.changedTouches[0]
				
				if (!this.isTouchMove) {
					var touchStartTimeoutFunc = setTimeout(res=> {
						if (!this.isClickOver && !this.isTouchMove) {
							var timer = setInterval(res=>{
								if (!this.isClickOver) {
									this.isPressRate = true
									if (this.currentRate != '2.0') {
										this.chooseBeiSu('2.0')
									}
								} else if (this.isPressRate == true) {
									if (this.currentRate != '1.0') {
										this.chooseBeiSu('1.0')
									}
									this.isPressRate = false
									clearInterval(timer);
								} else {
									clearInterval(timer);
								}
							}, 100)
						}
					}, 500);
				}
			},
			onTouchEnd() {
				this.isClickOver = true
				this.TouchMoveType = null
				if (!this.isPressRate && !this.isTouchMove) {
					let curTime = new Date().getTime();
					let lastTime = this.lastTapDiffTime;
					this.lastTapDiffTime = curTime;
					//两次点击间隔小于300ms, 认为是双击
					let diff = curTime - lastTime;
					if (diff < 300) {
						this.videoPlay()
						clearTimeout(this.lastTapTimeoutFunc); // 成功触发双击事件时，取消单击事件的执行
					} else {
						// 单击事件延时300毫秒执行
						this.lastTapTimeoutFunc = setTimeout(res=> {
							this.hideShowCM()
						}, 300);
					}
				}
				
				var touchTimer = setTimeout(res=> {
					this.isTouchMove = false
				}, 500);
			},
		}
	}
</script>

<style lang="scss" scoped>
	.iconfont {
		font-family: fontFamilyName;
	}
	.Active {
		left: 0;
	}
	.noActive {
		left: 750rpx;
		transition-property: left;
		transition-duration: 0.3s;
	}
	.cmShow {
		opacity: 1;
		transition-property: opacity;
		transition-duration: 1s;
	}
	.cmShowActive {
		opacity: 0;
	}
</style>
<template>
	<view class='container'>
		<view class='header bg-ff'>
			<view class='row padding border-b font-26'>
				<view class='col ellipsis-1'>
					<view class='ellipsis-1'>
						<view class=''>
							<text class='color'>[当前] </text>{{address.title}}
						</view>
						<view class='color-99 ellipsis-1'>
							{{address.address}}
						</view>
					</view>
				</view>
				<view class='padding-l' @click='addressSearch'>
					<icon class='icon_search' type='search' size='22' color='#666' />
				</view>
			</view>
			<!-- <view class='row padding font-26'>
				<input class='col' placeholder='补充详细地址：门牌号、楼房、房间号' v-model='detail' @confirm='submit'></input>
				<view class='bg color-ff padding-lr btn border' @click='submit'>确定</view>
			</view> -->
		</view>
		<map id='map' :scale='map.scale' show-location="true" :longitude='map.longitude' :latitude='map.latitude'
		 :width='map.width' :height='map.height' :controls='map.controls' :markers='map.markers' @regionchange='mapChange'
		 :style="{height:mapHeight}" :enable-overlooking="false" :enable-satellite="false" :enable-3D="false">
			<!-- <cover-view class='icon-position' style="margin-top: 100px;"> -->
			<cover-image src="../../static/Home/icon_position.png" class="icon-img"></cover-image>
			<!-- </cover-view> -->
		</map>
		<view class='footer bg-ff font-26'>
			<scroll-view scroll-y class='scroll' :scroll-top='scrollTop'>
				<view class="">
					<view class='padding border-b position-r' v-for='(item, index) in list' :key='index' @click='bindAddress(index)'>
						<view class='row'>{{item.title}}</view>
						<view class='row color-99'>{{item.address}}</view>
						<icon type='success' color='#E74246' size='22' class='icon_circle' v-if='checked === index' />
					</view>
				</view>

			</scroll-view>
		</view>
	</view>
</template>
<script>
	const app = getApp({
		allowDefault: true
	})
	var QQMapWX = require('@/libs/qqmap-wx-jssdk.min.js')
	var qqmapsdk = new QQMapWX({
		//key: 'LXCBZ-NNIKD-UZ64F-H6AFI-UNJLH-OCFGE'
		//key: 'JDFBZ-I5HHD-7IJ4M-PZDRR-4MDHE-4JFDS',
		key:'ZURBZ-WWTL6-ZNDSL-M7YHC-L67Q3-HRB7V',//黄
		// key:'HKLBZ-WM4KF-QYEJF-JANQ6-OPPMV-5IFS2' //傅
	})
	export default {
		data() {
			return {
				detail: '',
				mapHeight: '',
				map: {
					longitude: 113.76927057974245,
					latitude: 34.76670519464811,
					showLocation: true,
					width: 40,
					height: 40,
					scale: 16,
					controls: []
				},
				list: [],
				oftenList: [],
				address: {
					title: '',
					address: ''
				},
				checked: 0,
				scrollTop: 0,
				mapStatus: 1, // 控制选择地址时 地图不加载附近列表
				mapCtx: '',
				Name: '',
			}
		},
		onNavigationBarButtonTap() {
			this.submit();
		},
		created() {
			uni.setNavigationBarTitle({
				title: '搜索地址'
			})
		},
		onLoad(option) {
			let map = this.map;
			if (option.name) {
				this.Name = option.name;
			}
			let that = this;
			uni.getLocation({
				type: 'gcj02', // 返回国测局坐标
				geocode: true,
				success: function(res) {
					that.map.latitude = res.latitude;
					that.map.longitude = res.longitude;
				},
				fail: function(e) {
					uni.showToast({
						icon: 'none',
						title: '获取地址失败, 请检查是否开启定位权限~~'
					})
				}
			});
			uni.getSystemInfo({
				success(res) {
					that.mapHeight = res.windowHeight - (res.windowWidth*120)/750 - 210 + 'px';
				}
			})
			// this.getWidthHeight(e => {
			// 	console.log(e);
			//   map.controls[0].position.top = e.height - 45
			//   map.controls[0].position.left = e.width/2 - 10
			//   this.setData({
			// 	map: map
			//   })
			// })

		},
		mounted() {
			let that = this
			uni.getSystemInfo({
				success(res) {
					that.mapHeight = res.windowHeight - (res.windowWidth*120)/750 - 210 + 'px';
				}
			})
		},
		methods: {
			getAddress() {
				let that = this
				uni.getLocation({
					type: 'gcj02',
					success: function(res) {
						console.log(res)
						let map = that.data.map
						map.longitude = res.longitude
						map.latitude = res.latitude
						that.getWidthHeight(e => {
							map.controls[0].position.top = e.height / 2 - 35
							map.controls[0].position.left = e.width / 2 - 20
							that.setData({
								map: map,
								position: {
									longitude: res.longitude,
									latitude: res.latitude
								}
							})
							that.getAddressList(1);
						})
					},
				})
			},
			getWidthHeight(fn) {
				var query = uni.createSelectorQuery()
				query.select('#map').boundingClientRect()
				query.exec(res => {
					fn(res[0])
				})
			},
			getAddressList(s = 0) {
				let that = this
				let position = that.position
				// console.log(position)
				qqmapsdk.reverseGeocoder({
					location: {
						latitude: position.latitude,
						longitude: position.longitude
					},
					get_poi: 1,
					poi_options: "page_size=20;page_index=1",
					success: function(e) {
						if (s) {
							e.result.pois[0].select = 1
							that.setData({
								list: e.result.pois,
								address: e.result.pois[0],
								checked: 0
							})
						} else {
							that.setData({
								list: e.result.pois
							})
						}
						setTimeout(() => {
							that.scrollTop = 1
						}, 1000)
					},
					fail: err => {
						console.log(err)
					}
				})
			},
			mapChange(e) {
				let that = this
				// console.log(this.mapStatus);
				clearTimeout(this.timer)
				console.log(e);
				this.timer = setTimeout(() => {
					if (e.type == 'regionchange' || e.type == 'end') {
						// //#ifndef APP-PLUS
						// that.setData({
						// 	position: {
						// 		latitude: e.detail.center.latitude,
						// 		longitude: e.detail.center.longitude,
						// 	},
						// })
						// if (that.mapStatus) { // 防止地图点击时 进行多次加载
						// 	that.getAddressList(1)
						// } else {
						// 	that.mapStatus = 1
						// }
						// //#endif
						that.mapCtx = uni.createMapContext('map')
						that.mapCtx.getCenterLocation({
							success: res => {
								console.log(res)
								that.setData({
									position: {
										latitude: res.latitude,
										longitude: res.longitude,
									},
								})
								if (that.mapStatus) { // 防止地图点击时 进行多次加载
									that.getAddressList(1)
								} else {
									that.mapStatus = 1
								}
							}
						})
					}
				}, 500)
			},
			bindAddress(index) {
				let list = this.list
				let map = this.map
				map.latitude = list[index].location.lat
				map.longitude = list[index].location.lng
				this.setData({
					map: map,
					checked: index,
					address: list[index],
					mapStatus: 0
				})
				this.submit();
			},
			setData(obj) {
				Object.assign(this, obj)
			},
			addressSearch() {
				uni.navigateTo({
					url: '/pages/Home/MapSearch'
				})
			},
			submit() {
				let that = this
				let detail = that.detail || ''
				let address = that.address
				// console.log(address)
				let AddressData = {
					addressName: address.title + detail,
					address: address.address,
					lat: address.location.lat,
					lng: address.location.lng,
					adcode: address.ad_info.adcode,
					district: address.ad_info.district,
					city: address.ad_info.city,
				}
				if (this.Name == "qidian") {
					uni.setStorage({
						key: "StartPoint",
						data: AddressData,
						success() {
							uni.navigateBack({
								delta: 1
							})
						}
					})
				}
				if (this.Name == "zhongdian") {
					uni.setStorage({
						key: "EndPoint",
						data: AddressData,
						success() {
							uni.navigateBack({
								delta: 1
							})
						}
					})
				}
				// uni.navigateTo({
				// 	url:'../map/map',
				// 	success() {
				// 		uni.setStorage({
				// 			key:that.AddressData,
				// 			data:a,
				// 		})
				// 	}
				// })

				// console.log(a)
			}
		}
	}
</script>

<style lang="scss">
	/**app.wxss**/
	page,
	view,
	scroll-view,
	swiper,
	block,
	icon,
	text,
	rich-text,
	button,
	input,
	label,
	picker,
	picker-view,
	slider,
	textarea,
	navigator,
	image,
	video,
	map,
	video {
		margin: 0;
		padding: 0;
		box-sizing: border-box;
	}

	page {
		background: #F6F6F6;
		font-size: 32rpx;
	}

	image {
		display: block;
	}

	.item,
	.item-forward {
		background: #fff;
		padding: 25rpx 90rpx 25rpx 25rpx;
		position: relative;
		line-height: 46rpx;
	}

	.item-forward::before {
		content: '';
		width: 20rpx;
		height: 20rpx;
		border-top: 2px solid #a9a9a9;
		border-right: 2px solid #a9a9a9;
		border-radius: 2px;
		position: absolute;
		top: 50%;
		right: 35rpx;
		transform: rotate(45deg) translateY(-50%);
	}

	.item image,
	.item-forward image {
		width: 46rpx;
		height: 46rpx;
		display: block;
		margin-right: 10rpx;
		position: relative;
	}

	.active {
		background: #eee;
	}

	.row,
	.item,
	.item-forward,
	.coupons {
		display: flex;
		width: 100%;
	}

	.row-wrap {
		flex-wrap: wrap;
	}

	.col,
	.coupons .left {
		flex: 1;
		display: block;
		width: 100%;
	}

	.col-center {
		height: 100%;
		display: flex;
		align-items: center;
	}

	.float-r {
		float: right;
	}

	.padding {
		padding: 20rpx 25rpx;
	}

	.padding-t {
		padding-top: 20rpx;
	}

	.padding-b {
		padding-bottom: 20rpx;
	}

	.padding-l {
		padding-left: 25rpx;
	}

	.padding-r {
		padding-right: 25rpx;
	}

	.padding-tb {
		padding-top: 20rpx;
		padding-bottom: 20rpx;
	}

	.padding-lr {
		padding-left: 25rpx;
		padding-right: 25rpx;
	}

	.margin {
		margin: 20rpx 25rpx;
	}

	.margin-t {
		margin-top: 20rpx;
	}

	.margin-b {
		margin-bottom: 20rpx;
	}

	.margin-tb {
		margin-top: 20rpx;
		margin-bottom: 20rpx;
	}

	.margin-lr {
		margin-left: 25rpx;
		margin-right: 25rpx;
	}

	.border,
	.border-t,
	.border-r,
	.border-b,
	.border-l {
		position: relative;
	}

	.border {
		border: .5px solid #eee;
	}

	.border-t::after,
	.border-r::after,
	.border-b::after,
	.border-l::after {
		content: '';
		position: absolute;
		/*background: #eee;*/
		background: linear-gradient(to top, #eee .7px, transparent .7px);
	}

	.border-t::after,
	.border-b::after {
		height: 1px;
		left: 25rpx;
		right: 25rpx;
		top: 0;
	}

	.border-b::after {
		top: auto;
		bottom: 0;
	}

	.border-l::after,
	.border-r::after {
		width: 1px;
		top: 0;
		bottom: 0;
		left: 0;
		background: linear-gradient(to right, #eee .7px, transparent .7px);
	}

	.border-r::after {
		left: auto;
		right: 0;
	}

	.ellipsis-1 {
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
	}

	.bg {
		background: #E74246;
	}

	.bg-ff {
		background: #fff;
	}

	.color {
		color: #E74246;
	}

	.color-00 {
		color: #000;
	}

	.color-ff {
		color: #fff;
	}

	.color-99,
	.icon_img_tip {
		color: #999;
	}

	.color-6c {
		color: #6c6c6c;
	}

	.text-right {
		text-align: right;
	}

	.font-26 {
		font-size: 26rpx;
	}

	.position-r {
		position: relative;
	}

	page {
		position: relative;
	}

	// .map,
	// map {
	// 	width: auto;
	// 	height: auto;
	// 	position: fixed;
	// 	left: 0;
	// 	top: 60px;
	// 	right: 0;
	// 	bottom: 210px;
	// }
	.map,
	map {
		width: 750rpx;
		// height: 424px;
		position: absolute;
		left: 0;
		top: 120rpx;
		right: 0;
		bottom: 210px;
	}


	.map {
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.header {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		height: 120rpx;
		overflow: hidden;
	}

	.icon_search {
		margin-top: 20rpx;
	}

	.btn {
		line-height: 25px;
		border-radius: 4rpx;
		margin-top: -5rpx;
	}

	.footer {
		position: fixed;
		left: 0;
		right: 0;
		bottom: 0;
		height: 210px;
	}

	.foot-border {
		border-bottom: 1px solid #eee;
		line-height: 32rpx;
	}

	.foot-border .padding {
		padding: 25rpx;
	}

	.foot-active {
		color: #E74246;
		position: relative;
	}

	.foot-active::after {
		content: '';
		position: absolute;
		height: 2px;
		background: #E74246;
		border-radius: 2px;
		bottom: 0;
		width: 5em;
		left: 50%;
		transform: translateX(-50%);
	}

	.scroll {
		position: absolute;
		left: 0;
		right: 0;
		top: 0;
		bottom: 0;
	}

	.scroll .padding {
		padding-right: 40px;
	}

	.scroll .icon_circle {
		position: absolute;
		right: 25rpx;
		top: 50%;
		transform: translateY(-50%);
	}

	.icon_img_tip {
		padding: 20rpx 0;
	}

	.icon-img {
		width: 36px;
		height: 36px;
		display: block;
		position: relative;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
	}
</style>

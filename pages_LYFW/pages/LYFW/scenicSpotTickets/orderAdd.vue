<template>
	<view>
		<view style="width: 100%;height: 150rpx;">
			
		</view>
		
		<view class="tab-item" style="text-align: center;">
			<text>{{admissionTicket.admissionTicketName}}</text>
		</view>
		
		<view class="tab-item" style="display: flex;flex-direction: row;justify-content: space-between;">
			<view>
				<text>使用日期</text>
			</view>
			<view>
				<text class="MP_textDate" @click="open">{{date}}&nbsp;> </text>
				<text class="MP_textReminder">{{dateReminder}}</text>
			</view>
		</view>
		
		<view class="tab-item">
			<view style="display: flex;flex-direction: row;justify-content: space-between;">
				<view>
					<text>购票人信息</text>
				</view>
				<view>
					<text @click="choiceUser(1)">选择</text>
				</view>
			</view>
			<view class="MP_userInformation" v-for="(item,index) in addressData" :key="index">
				<text>{{item.userName}}</text>
				<text class="Mp_sex">{{item.userSex}}</text>
				<text class="Mp_delete  jdticon icon-fork" @click="deleteUser(index)"></text>
				<text class="Mp_text">身份证：{{item.userCodeNum}}</text>
				<text class="Mp_text">手机号：{{item.userPhoneNum}}</text>
			</view>
			
		</view>
		<view class="tab-item">
			<view>
				<text>实付款</text>
			</view>
			<view>
				<text>
					￥{{actualPayment}}
				</text>
			</view>
			<view @click="submitState">
				<text style="color: #DD524D;">立即预订</text>
			</view>
		</view>
		
		<uni-calendar ref="calendar" :insert="false" :lunar="true" @confirm="confirm"></uni-calendar>
	</view>
</template>

<script>
	import uniPopup from "@/pages_LYFW/components/LYFW/scenicSpotTickets/uni-popup/uni-popup.vue"
	import uniCalendar from '@/pages_LYFW/components/LYFW/scenicSpotTickets/uni-calendar/uni-calendar.vue'
	import $lyfw from '@/common/LYFW/LyfwFmq.js' //旅游服务
	export default {
		data() {
			const currentDate = this.getDate({
				format: true
			})
			return {
				submissionState: false, //提交状态
				actualPayment: '', //实际付款
				selectedValue: 1, //同意须知的选中值
				dateReminder: '今天', //日期提醒
				date: currentDate, //默认时间
				maskState: 0, //优惠券面板显示状态

				admissionTicket: '', //门票内容
				userInfo: '', //个人信息

				couponIndex: '请选择优惠券', //优惠券默认内容
				couponColor: '', //优惠券couponID
				couponCondition: '', //优惠券的满足条件值

				notice: '', // 预订须知

				addressData: '', //购票人信息
				adultIndex: '', //成人数量
				childrenIndex: '', //儿童数量

				couponList: []
			}
		},

		onLoad:function(options){
			this.lyfwData();
		},
		onShow:function(){
			this.selectedValue = 1;
			this.getUserInfo();
			this.userData();
			this.getCoupons();
		},
		components: {
			//加载多方弹框组件
			uniPopup,
			//加载日期组件 
			uniCalendar,
		},
		methods: {
			//获取用户信息
			getUserInfo:function(){
				uni.getStorage({
					key:'userInfo',
					success:(res)=>{
						this.userInfo = res.data;
					}
				})
			},
			
			//获取用户优惠券
			getCoupons:function(){
				uni.request({
					url: $lyfw.Interface.spt_GetcouponByuserId.value,
					method: $lyfw.Interface.spt_GetcouponByuserId.method,
					data: {
						userId: this.userInfo.userId
					},
					header: {
						'content-type': 'application/json'
					},
					success: (res) => {
						// console.log(res)
						this.couponList = res.data.data;
					}
				})
			},
			
			//读取静态数据
			lyfwData(e) {
				uni.getStorage({
					key: 'ticketInformation',
					success: (res) => {
						this.admissionTicket = res.data;
						uni.request({
							url: $lyfw.Interface.spt_GetticketSecurityByticketIde.value,
							method: $lyfw.Interface.spt_GetticketSecurityByticketIde.method,
							data: {
								ticketId: res.data.ticketId
							},
							header: {
								'content-type': 'application/json'
							},
							success: (res) => {
								// console.log(res)
								this.notice = res.data.data[0];
							}
						})

						// console.log(res)
					}
				})
				



			},

			//删除出行人
			deleteUser: function(e) {
				var a = this.addressData;
				if (e < 0) {
					return false;
				} else {
					var b = a.slice(0, e).concat(a.slice(e + 1, a.length));
					this.addressData = b;
					uni.setStorage({
						key: "passengerList",
						data: b
					})
					this.screenUser();
				}
			},

			//选择用户
			choiceUser: function(e) {
				if (e == 0) {
					uni.getStorage({
						key: 'userInfo',
						fail() {
							uni.showToast({
								icon: 'none',
								title: '未登录无法添加乘车人,请先登录'
							})
							//#ifdef APP-PLUS
							setTimeout(function() {
								uni.navigateTo({
									//loginType=1,泉运登录界面
									//loginType=2,今点通登录界面
									//loginType=3,武夷股份登录界面
									url: '../../../../pages/GRZX/userLogin?loginType=1'
								})
							}, 500);
							//#endif
							//#ifdef MP-WEIXIN
							uni.navigateTo({
								url:'/pages/Home/wxAuthorize',
							})
							// #endif
						},
						success() {
							uni.navigateTo({
								url: '../../../../pages/GRZX/addPassenger?type=add',
							})
						}
					})
				} else if (e == 1) {
					uni.navigateTo({
						url: '../../../../pages/GRZX/passengerInfo?submitType=1'
					})
				}


			},

			//用户数据读取
			userData() {
				uni.getStorage({
					key: 'passengerList',
					success: (res) => {
						this.addressData = res.data;
						this.screenUser();
					}
				});
			},

			//数组提取
			screenUser: function() {
				let adult = this.addressData.filter(item => {
					return item.userType == '成人' || item.userType == '军人' || item.userType == '教师' || item.userType == '学生';
				})
				let children = this.addressData.filter(item => {
					return item.userType == '儿童';
				})
				this.adultIndex = adult.length;
				this.childrenIndex = children.length;
				this.numberChange();
			},

			//显示优惠券面板
			toggleMask(type) {
				let timer = type === 'show' ? 10 : 300;
				let state = type === 'show' ? 1 : 0;
				this.maskState = 2;
				setTimeout(() => {
					this.maskState = state;
				}, timer)
			},

			//优惠券赋值
			couponEvent: function(index) {
				if (this.actualPayment >= this.couponList[index].condition) {
					this.couponIndex = '-' + this.couponList[index].price;
					this.couponColor = this.couponList[index].couponID;
					this.couponCondition = this.couponList[index].condition;
					this.numberChange();
					this.toggleMask();
				} else {
					uni.showToast({
						title: '您的实付款未达到条件，请重新选择',
						icon: 'none'
					})
				}

			},

			//取消优惠券
			couponReset: function(index) {
				this.couponIndex = '请选择优惠券';
				this.couponColor = '';
				this.numberChange();
				this.toggleMask();
			},

			//仿穿透事件
			stopPrevent() {},

			// 数量+计价
			numberChange() {
				const b = (this.admissionTicket.ticketAdultPrice * this.adultIndex) + (this.admissionTicket.ticketChildPrice * this
					.childrenIndex);
				const a = b.toFixed(2);
				if (this.couponColor == '') {
					this.actualPayment = a;
				} else if (a >= this.couponCondition) {
					var total = a - this.couponList[this.couponColor].price;
					this.actualPayment = total;
				} else if (a < this.couponCondition) {
					uni.showToast({
						title: '您的金额不满足优惠券条件，已取消优惠券',
						icon: 'none',
						duration: 2000
					})
					this.couponIndex = '请选择优惠券';
					this.couponColor = '';
					this.couponCondition = 0;
					this.actualPayment = a;
				}
			},

			//提交按钮状态赋值
			submitState: function() {
				//这边还得加上是否选择人数和勾选同意的判断
				if (this.selectedValue == 1 && this.addressData.length > 0) {
					if (this.submissionState == false) {
						this.submissionState = true;
						this.submit();
					} else if (this.submissionState == true) {
						uni.showToast({
							title: '请勿重复点击提交',
							icon: 'none',
							duration: 2000
						})
					}

				} else if (this.addressData.length == 0) {
					uni.showToast({
						title: '请添加购票人信息',
						icon: 'none'
					})
				} else {
					uni.showToast({
						title: '请同意游客须知',
						icon: 'none'
					})
				}
			},

			//提交表单
			submit: function() {
				var that = this;
				
				uni.showLoading({
					title: '提交订单中...'
				})
				
				// #ifdef H5
				if(that.userInfo.openId_wx){
					uni.request({
						url: $lyfw.Interface.spt_AddtouristOrder.value,
						method: $lyfw.Interface.spt_AddtouristOrder.method,
						data: {
							userId: that.userInfo.userId,
							ticketId: that.admissionTicket.ticketId,
							userPhone: that.userInfo.phoneNumber,
							ticketProductId: that.admissionTicket.admissionTicketID,
							couponID: that.couponColor,
							orderDate: that.date,
							orderInsure: '',
							orderInsurePrice: '',
							orderActualPayment: that.actualPayment,
							ticketContain: that.admissionTicket.ticketContain,
							sellerCompanyCode: 'H5',
							tppId: that.userInfo.openId_wx,
							addressData: that.addressData,
							
						},
						//向服务器发送订单数据，返回订单编号
						success: (res) => {
							// console.log(res)
							if (res.data.msg == '抱歉!下单失败,当日已取消订单次数超过限额,已被限制下单操作') {
								uni.hideLoading()
								uni.showToast({
									title: '当日已取消订单次数超过限额',
									icon: 'none',
								})
								that.submissionState = false;
					
							} else if (res.data.msg == '抱歉!下单失败,您当前有未支付完成的订单') {
								uni.hideLoading()
								uni.showToast({
									title: '下单失败,您当前有未支付订单',
									icon: 'none',
									duration: 2000,
									success: function() {
										setTimeout(function(){
											uni.switchTab({
												url: '../../../../pages/order/OrderList'
											})
										},2000)
										
										that.submissionState = false;
								
									}
								})
					
							} else if (res.data.msg == '订单下单成功') {
								uni.hideLoading()
								uni.redirectTo({
									url: 'selectivePayment?orderNumber=' + res.data.data.orderNumber
								})
							} else if (res.data.msg == '抱歉,订单下单失败') {
								uni.hideLoading()
								uni.showToast({
									title: '下单失败，请联系客服',
									icon: 'none',
								})
								that.submissionState = false;
							} else {
								uni.hideLoading()
								uni.showToast({
									title: '下单失败，请返回并重进页面',
									icon: 'none',
								})
								that.submissionState = false;
					
							}
					
						}
					})
				}else{
					uni.hideLoading()
					uni.showToast({
						title: '请允许授权给公众号，即将为您返回主页！',
						icon:'none'
					})
					uni.switchTab({
						url:'../../../../pages/Home/zy_zhcx'
					})
					that.submissionState = false;
				}
				// #endif

				// #ifdef APP-PLUS
				if(that.userInfo.userId !==''){
					uni.request({
						url: $lyfw.Interface.spt_AddtouristOrder.value,
						method: $lyfw.Interface.spt_AddtouristOrder.method,
						data: {
							userId: that.userInfo.userId,
							ticketId: that.admissionTicket.ticketId,
							userPhone: that.userInfo.phoneNumber,
							ticketProductId: that.admissionTicket.admissionTicketID,
							couponID: that.couponColor,
							orderDate: that.date,
							orderInsure: '',
							orderInsurePrice: '',
							orderActualPayment: that.actualPayment,
							ticketContain: that.admissionTicket.ticketContain,
							sellerCompanyCode: 'APP',
							tppId: 0,
							addressData: that.addressData,
						},
						header: {
							'content-type': 'application/json'
						},
						//向服务器发送订单数据，返回订单编号
						success: (res) => {
							// console.log(res)
							if (res.data.msg == '抱歉!下单失败,当日已取消订单次数超过限额,已被限制下单操作') {
								uni.hideLoading()
								uni.showToast({
									title: '当日已取消订单次数超过限额',
									icon: 'none',
								})
								that.submissionState = false;
					
							} else if (res.data.msg == '抱歉!下单失败,您当前有未支付完成的订单') {
								uni.hideLoading()
								uni.showToast({
									title: '下单失败,您当前有未支付订单',
									icon: 'none',
									duration: 2000,
									success: function() {
										setTimeout(function(){
											uni.switchTab({
												url: '../../../../pages/order/OrderList'
											})
										},2000)
										that.submissionState = false;
								
									}
								})
					
							} else if (res.data.msg == '订单下单成功') {
								uni.hideLoading()
								uni.redirectTo({
									url: 'selectivePayment?orderNumber=' + res.data.data.orderNumber
								})
					
							} else if (res.data.msg == '抱歉,订单下单失败') {
								uni.hideLoading()
								uni.showToast({ 
									title: '下单失败，请联系客服',
									icon: 'none',
								})
								that.submissionState = false;
							} else {
								uni.hideLoading()
								uni.showToast({
									title: '下单失败，请返回并重进页面',
									icon: 'none',
								})
								that.submissionState = false;
					
							}
					
						},
						fail: function(ee) {
							console.log(ee)
						}
					})
				}else{
					uni.hideLoading()
					uni.showToast({
						title: '未登录账号，即将跳转登录！'
					})
					uni.navigateTo({
						url:'../../../../pages/GRZX/userLogin?loginType=1&&urlData=2'
					})
					that.submissionState = false;
				}
				
				// #endif

				// #ifdef MP-WEIXIN
				if(that.userInfo.openId_xcx){
					uni.request({
						url: $lyfw.Interface.spt_AddtouristOrder.value,
						method: $lyfw.Interface.spt_AddtouristOrder.method,
						data: {
							userId: that.userInfo.userId,
							ticketId: that.admissionTicket.ticketId,
							userPhone: that.userInfo.phoneNumber,
							ticketProductId: that.admissionTicket.admissionTicketID,
							couponID: that.couponColor,
							orderDate: that.date,
							orderInsure: '',
							orderInsurePrice: '',
							orderActualPayment: that.actualPayment,
							ticketContain: that.admissionTicket.ticketContain,
							sellerCompanyCode: '小程序',
							tppId: that.userInfo.openId_xcx,
							addressData: that.addressData,
						},
						//向服务器发送订单数据，返回订单编号
						success: (res) => {
							// console.log(res)
							if (res.data.msg == '抱歉!下单失败,当日已取消订单次数超过限额,已被限制下单操作') {
								uni.hideLoading()
								uni.showToast({
									title: '当日已取消订单次数超过限额',
									icon: 'none',
								})
								that.submissionState = false;
									
							} else if (res.data.msg == '抱歉!下单失败,您当前有未支付完成的订单') {
								uni.hideLoading()
								uni.showToast({
									title: '下单失败,您当前有未支付订单',
									icon: 'none',
									duration: 2000,
									success: function() {
										setTimeout(function(){
											uni.switchTab({
												url: '../../../../pages/order/OrderList'
											})
										},2000)
										that.submissionState = false;
								
									}
								})
									
							} else if (res.data.msg == '订单下单成功') {
								uni.hideLoading()
								uni.redirectTo({
									url: 'selectivePayment?orderNumber=' + res.data.data.orderNumber
								})
							} else if (res.data.msg == '抱歉,订单下单失败') {
								uni.hideLoading()
								uni.showToast({
									title: '下单失败，请联系客服',
									icon: 'none',
								})
								that.submissionState = false;
							} else {
								uni.hideLoading()
								uni.showToast({
									title: '下单失败，请返回并重进页面',
									icon: 'none',
								})
								that.submissionState = false;
									
							}
									
						}
					})
				}else{
					uni.hideLoading()
					uni.showToast({
						title: '请允许授权给小程序，即将跳转登录！'
					})
					uni.navigateTo({
						url:'../../../../pages/Home/wxAuthorize'
					})
					that.submissionState = false;
				}
				// #endif



			},

			//打开选择器
			open() {
				this.$refs.calendar.open();
			},

			//打开popup下弹框
			open2(e) {
				if (e == 1) {
					this.$refs.popup1.open()
				} else {
					this.$refs.popup2.open()
				}
			},

			close(e) {
				if (e == 1) {
					this.$refs.popup1.close()
				} else {
					this.$refs.popup2.close()
				}
			},



			//获取当前时间并格式化
			getDate(type) {
				const date = new Date();
				let year = date.getFullYear();
				let month = date.getMonth() + 1;
				let day = date.getDate();
				if (type === 'start') {
					year = year - 60;
				} else if (type === 'end') {
					year = year + 2;
				}
				month = month > 9 ? month : '0' + month;;
				day = day > 9 ? day : '0' + day;
				return `${year}-${month}-${day}`;
			},


			//时间选择器，确认选择
			confirm(e) {
				this.getTime(e);
			},

			//时间效验，已提示
			getTime: function(e) {
				let date = new Date(),
					day = date.getDate(),
					second = date.getSeconds() < 10 ? "0" + date.getSeconds() : date.getSeconds();
				day >= 0 && day <= 9 ? (day = "0" + day) : "";
				var index = e.date - day;
				// console.log(e)
				if (index < 0) {
					uni.showToast({
						title: '请勿选择以往日期',
						icon: 'none'
					})
				} else if (index == 0) {
					this.dateReminder = '今天'
					this.date = e.fulldate;
				} else if (index == 1) {
					this.dateReminder = '明天'
					this.date = e.fulldate;
				} else if (index == 2) {
					this.dateReminder = '后天'
					this.date = e.fulldate;
				} else {
					this.dateReminder = e.lunar.ncWeek;
					this.date = e.fulldate;
				}
			},


		}
	}
</script>

<style lang="scss">
	page {
		background: #F5F5F5;
		padding-bottom: 100upx;
	}
	
	.tab-item{
		padding: 20rpx;
	}

	//背景图样式
	.ob_background {
		position: absolute;
		width: 100%;
		height: 320upx;

		image {
			width: 100%;
			height: 100%;
		}
	}

	//整体容器样式
	.cover-container {
		position: relative;
		top: 148upx;
		padding: 32upx 30upx;
		padding-bottom: 180upx;
	}

	/* #ifdef MP-WEIXIN */
	//整体容器样式 -微信版
	.cover-container {
		top: 64upx;
	}

	/* #endif */


	//公共样式 - 适用多个数据框
	.MP_information1 {
		border-radius: 16upx;
		background: #FFFFFF;
		padding: 24upx 32upx;
		font-size: 32upx;
		box-shadow: 0px 0.2px 0px #aaa;

		.MP_title {
			font-size: 34upx;
			display: flex;
			font-weight: bold;
			margin-top: 20upx;
		}

		.MP_text {
			margin-top: 20upx;
			color: #3EABFC;
			font-size: 28upx;
			display: block; // 让字体换行
		}
	}

	//公共样式2 - 适用单选框
	.MP_information2 {
		border-radius: 16upx;
		background: #FFFFFF;
		padding: 36upx 32upx;
		font-size: 32upx;
		box-shadow: 0px 0.2px 0px #aaa;
		margin-top: 24upx;

		.kj {
			font-size: 34upx;
			display: flex;
			font-weight: bold;
			margin-top: 8upx;
		}

		.MP_text {
			font-size: 26upx;
			margin-top: 20upx;
			display: block; // 让字体换行
		}
	}


	//须知弹框
	.boxVlew {
		width: 90%;
		padding: 16upx 40upx;
		padding-bottom: 92upx;
		background: #FFFFFF;

		.titleView {
			margin: 24upx 0;

			//弹框标题
			.Nb_text1 {
				position: relative;
				font-size: 38upx;
				font-weight: bold;
				top: 8upx;
				margin-bottom: 16upx;
			}

			//弹框关闭按钮
			.Nb_text2 {
				margin-top: 8upx;
				float: right;
				color: #333;
				font-size: 32upx;
			}
		}

		.noticeBox {
			height: 800upx;
			line-height: 32upx;

			.Nb_text3 {
				display: block;
				margin-top: 32upx;
				font-size: 34upx;
				font-weight: bold;
			}

			.Nb_text4 {
				display: block;
				line-height: 64upx;
				margin: 32upx 0;
				font-size: 30upx;
			}
		}
	}


	//使用日期
	.MP_selectionDate {
		width: 100%;
		line-height: 120upx;
		margin-top: 46upx;
		border-top: 1px #F5F5F5 dashed;

		.MP_textDate {
			float: right;
		}

		.MP_textReminder {
			font-size: 26upx;
			color: #aaa;
			float: right;
			margin-right: 24upx;
		}
	}

	// 用户信息
	.MP_userInformation {
		width: 100%;
		margin-top: 32upx;
		border-top: 1px #F5F5F5 dashed;
		padding-top: 32upx;

		.Mp_sex {
			margin-left: 24upx;
		}

		.Mp_text {
			font-size: 28upx;
			display: block;
			color: #888;
			margin-top: 20upx;
		}

		.Mp_square {
			margin-left: 24upx;
			padding: 2upx 20upx;
			background: #3DABFC;
			font-size: 26upx;
			color: #fff;
			text-align: center;
			border-radius: 8upx;
		}

		.Mp_delete {
			float: right;
			color: #f85e52;
			font-size: 34upx;
		}

		.Mp_addTo {
			float: left;
			font-size: 30upx;
			margin-left: 64upx;
			width: 200upx;
		}

		.Mp_Selection {
			font-size: 30upx;
			margin-right: 64upx;
			width: 200upx;
		}
	}

	//选项框样式
	.MP_optionBar {
		.Mp_title {
			font-size: 32upx;
		}

		.Mp_text {
			margin-top: 6upx;
			float: right;
			font-size: 28upx;
			color: #f85e52;
		}

		.Mp_textBlue {
			margin-left: 16upx;
			font-size: 26upx;
			color: #3EABFC;
		}

		.Mp_box {
			float: right;
			position: relative;
			bottom: 6upx;
			right: -12upx;
		}

		.Mp_arrow {
			margin-top: 6upx;
			margin-left: 24upx;
			float: right;
			font-size: 28upx;
			color: #aaa;
		}
	}

	/* 优惠券面板 */
	.mask {
		display: flex;
		align-items: flex-end;
		position: fixed;
		left: 0;
		top: var(--window-top);
		bottom: 0;
		width: 100%;
		background: rgba(0, 0, 0, 0);
		z-index: 9995;
		transition: .3s;

		.mask-content {
			width: 100%;
			min-height: 30vh;
			max-height: 70vh;
			background: #f3f3f3;
			transform: translateY(100%);
			transition: .3s;
			overflow-y: scroll;
		}

		&.none {
			display: none;
		}

		&.show {
			background: rgba(0, 0, 0, .4);

			.mask-content {
				transform: translateY(0);
			}
		}
	}


	/* 优惠券列表 */

	//下弹框标题
	.couponTitle {
		padding: 40upx;
		padding-bottom: 16upx;

		.Co_text1 {
			font-size: 38upx;
			font-weight: bold;
		}

		.Co_text2 {
			margin-top: 8upx;
			float: right;
			color: #f85e52;
			font-size: 28upx;
		}
	}

	.coupon-item {
		display: flex;
		flex-direction: column;
		margin: 20upx 24upx;
		background: #fff;

		.con {
			display: flex;
			align-items: center;
			position: relative;
			height: 120upx;
			padding: 0 30upx;

			&:after {
				position: absolute;
				left: 0;
				bottom: 0;
				content: '';
				width: 100%;
				height: 0;
				border-bottom: 1px dashed #f3f3f3;
				transform: scaleY(50%);
			}
		}

		.left {
			display: flex;
			flex-direction: column;
			justify-content: center;
			flex: 1;
			overflow: hidden;
			height: 100upx;
		}

		.title {
			font-size: 32upx;
			color: #f85e52;
			margin-bottom: 10upx;
		}

		.time {
			font-size: 24upx;
			color: #999999;
		}

		.right {
			display: flex;
			flex-direction: column;
			justify-content: center;
			align-items: center;
			font-size: 26upx;
			color: #999999;
			height: 100upx;
		}

		.price {
			font-size: 44upx;
			color: #f85e52;

			&:before {
				content: '￥';
				font-size: 34upx;
			}
		}

		.tips {
			font-size: 24upx;
			color: #999999;
			line-height: 60upx;
			padding-left: 30upx;
		}

		.circle {
			position: absolute;
			left: -6upx;
			bottom: -10upx;
			z-index: 10;
			width: 20upx;
			height: 20upx;
			background: #f3f3f3;
			border-radius: 100px;

			&.r {
				left: auto;
				right: -6upx;
			}
		}
	}

	//底部
	.footer {
		position: fixed;
		left: 0;
		bottom: 0;
		z-index: 995;
		display: flex;
		align-items: center;
		width: 100%;
		height: 90upx;
		justify-content: space-between;
		font-size: 30upx;
		background: #fff;
		z-index: 998;
		color: #f85e52;
		box-shadow: 0 -1px 5px rgba(0, 0, 0, .1);

		.price-content {
			padding-left: 30upx;
		}

		.price-tip {
			color: #f85e52;
			margin-left: 8upx;
		}

		.price {
			font-size: 36upx;
			color: #f85e52;
		}

		.people {
			font-size: 28upx;
			margin-left: 24upx;
			color: #999999;
		}

		.submitChange {
			display: flex;
			align-items: center;
			justify-content: center;
			width: 280upx;
			height: 100%;
			background: #aaa;

			.submit {
				color: #fff;
				font-size: 32upx;
			}

			&.submitColor {
				background: #06B4FD;
			}
		}
	}
</style>

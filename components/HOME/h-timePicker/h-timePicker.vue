<!--
 * @Description: 时间选择
 * @Author: hjh
 * @Date: 2019-08-17 23:19:32
 * @LastEditors: hjh
 * @LastEditTime: 2019-08-19 09:27:18
 * @Sign: 扬眉剑出鞘
 -->

<template>
	<view>
		<picker class='time-picker' mode="multiSelector" @change="bindStartMultiPickerChange" @columnchange="bindMultiPickerColumnChange"
		 :value="multiIndex" :range="multiArray">
			<slot name="pCon"></slot>
		</picker>
	</view>
</template>

<script>
	export default {
		props: {
			sTime: { //开始小时
				type: [Number, String],
				default: "0"
			},
			cTime: { //结束小时
				type: [Number, String],
				default: "23"
			},
			timeNum: { //延迟小时
				type: [Number, String],
				default: "0"
			},
			interval: { //分钟间隔
				type: [Number, String],
				default: "1"
			},
			sDay: { //开始天数
				type: [Number, String],
				default: "0"
			},
			dayNum: { //预约天数
				type: [Number, String],
				default: "2"
			},

		},
		data() {
			return {
				sDayNum: 0,
				multiArray: [
					['今天', '明天', '3-2', '3-3', '3-4', '3-5'],
					[0, 1, 2, 3, 4, 5, 6],
					[0, 10, 20]
				],
				multiIndex: [0, 0, 0],
				multiSelector: '',
			};
		},
		beforeMount() {
			this.pickerTap()
		},
		methods: {
			timeFormat: function(num) {

				if (num < 10 && (num + '').length == 1) {
					return '0' + num;
				}
				return num;
			},
			pickerTap: function() {
				let date = new Date();
				let monthDay = [];
				let hours = [];
				let minute = [];
				this.sDayNum = this.sDay;

				// 时
				let date1 = new Date(date);
				let sT = +this.sTime;
				let eT = +this.cTime;

				if (sT <= eT) {
					let h = date1.getHours() < sT ? sT : date1.getHours();
					h = h + parseInt(this.timeNum);
					if (h > eT || this.sDayNum > 0) {
						this.sDayNum = this.sDayNum <= 0 ? parseInt(this.sDay) + 1 : parseInt(this.sDay);
						for (let i = sT; i <= eT; i++) {
							hours.push(this.timeFormat(i) + '时');
						}
					} else {
						for (let i = h; i <= eT; i++) {
							hours.push(this.timeFormat(i) + '时');
						}
					}

				} else {
					let h = date1.getHours() < sT ? sT : date1.getHours();
					h = h + parseInt(this.timeNum);
					if (h > eT && h < sT || h > 23 || this.sDayNum > 0) {
						this.sDayNum = this.sDayNum <= 0 ? parseInt(this.sDay) + 1 : parseInt(this.sDay);
						for (let i = 0; i <= 23; i++) {
							if (i < sT && i > eT) {

							} else {
								hours.push(this.timeFormat(i) + '时');
							}
						}
					} else {

						for (let i = h; i <= 23; i++) {
							if (i < sT && i > eT) {

							} else {
								hours.push(this.timeFormat(i) + '时');
							}
						}

					}
				}

				// 月-日
				for (let i = +this.sDayNum; i <= (parseInt(this.sDayNum) + parseInt(this.dayNum)); i++) {
					let date1 = new Date(date);
					date1.setDate(date.getDate() + i);
					let md = date1.getFullYear() + '-' + this.timeFormat(date1.getMonth() + 1) + "-" + this.timeFormat(date1.getDate());
					monthDay.push(md);
				}

				// 分
				let inter = +this.interval < 60 ? +this.interval : 59;
				let m = date1.getMinutes() < 0 ? sT : date1.getMinutes();

				m = Math.ceil(m / inter) * inter;
				// m=m+4;  
				//  console.log(m);				
				if (m > 0) {
					for (let i = m; i < 60; i += inter) {
						minute.push(i < 10 ? '0' + i + '分' : i + '分');
					}
				} else {
					for (let i = 0; i < 60; i += inter) {
						minute.push(i < 10 ? '0' + i + '分' : i + '分');
					}
				}

				let data = {
					multiArray: this.multiArray,
					multiIndex: this.multiIndex
				};

				data.multiArray[0] = monthDay;
				data.multiArray[1] = hours;
				data.multiArray[2] = minute;
				this.multiArray = data.multiArray;
				this.multiIndex = data.multiIndex;
			},
			bindMultiPickerColumnChange(e) {
				console.log(e)
				console.log(this.multiArray );
				if (e.detail.column == 0) {
					this.multiIndex[0]=[];
					this.multiIndex[0]= e.detail.value;
				}
				if (e.detail.column == 1) {
					this.multiIndex[1]=[];
					this.multiIndex[1]=e.detail.value;
				}
				if (e.detail.column == 2) {
					this.multiIndex[2]=[];
					this.multiIndex[2]= e.detail.value;
				}
				let hours = [];
				if (e.detail.column == 0 && e.detail.value == 0 && +this.sDayNum == 0) {
					let date = new Date();
					// 时
					let date1 = new Date(date);
					let sT = +this.sTime;
					let eT = +this.cTime;

					if (sT <= eT) {
						let h = date1.getHours() < sT ? sT : date1.getHours();
						h = h + parseInt(this.timeNum);
						if (h > eT || this.sDayNum > 0) {
							this.sDayNum = this.sDayNum <= 0 ? parseInt(this.sDay) + 1 : parseInt(this.sDay);
							for (let i = sT; i <= eT; i++) {
								hours.push(this.timeFormat(i) + '时');
							}
						} else {
							for (let i = h; i <= eT; i++) {
								hours.push(this.timeFormat(i) + '时');
							}
						}

					} else {
						let h = date1.getHours() < sT ? sT : date1.getHours();
						h = h + parseInt(this.timeNum);
						if (h > eT && h < sT || h > 23 || this.sDayNum > 0) {
							this.sDayNum = this.sDayNum <= 0 ? parseInt(this.sDay) + 1 : parseInt(this.sDay);
							for (let i = 0; i <= 23; i++) {
								if (i < sT && i > eT) {

								} else {
									hours.push(this.timeFormat(i) + '时');
								}
							}
						} else {

							for (let i = h; i <= 23; i++) {
								if (i < sT && i > eT) {

								} else {
									hours.push(this.timeFormat(i) + '时');
								}
							}

						}
					}
					this.multiArray.splice(1, 1, hours)
				} else if (e.detail.column == 0 && e.detail.value != 0) {
					let sT = +this.sTime;
					let eT = +this.cTime;
					if (sT <= eT) {
						for (let i = sT; i <= eT; i++) {
							hours.push(this.timeFormat(i) + '时');
						}
					} else {
						for (let i = 0; i <= 23; i++) {
							if (i < sT && i > eT) {

							} else {
								hours.push(this.timeFormat(i) + '时');
							}
						}
					}
					this.multiArray.splice(1, 1, hours)
				}
				//分钟
				let minute = [];
				let date = new Date();
				let date1 = new Date(date);
				let inter = +this.interval < 60 ? +this.interval : 59;
				let m = date1.getMinutes() < 0 ? sT : date1.getMinutes();
				// m=m+30;
				m = Math.ceil(m / inter) * inter;
				console.log(this.multiIndex);
				if ((e.detail.column == 0 && e.detail.value == 0 && +this.sDayNum == 0&&this.multiIndex[1]==0) || (this.multiIndex[0]==0&&e.detail.column == 1 && e.detail.value ==
						0)) {
					if (m > 0) {
						for (let i = m; i < 60; i += inter) {
							minute.push(i < 10 ? '0' + i + '分' : i + '分');
						}
					} else {
						for (let i = 0; i < 60; i += inter) {
							minute.push(i < 10 ? '0' + i + '分' : i + '分');
						}
					}
					this.multiArray.splice(2, 2, minute);
				} else {
					if(this.multiIndex[0]!=0||this.multiIndex[1]!=0){
						for (let i = 0; i < 60; i += inter) {
							minute.push(i < 10 ? '0' + i + '分' : i + '分');
						}
						this.multiArray.splice(2, 2, minute);
					}
				}
			},
			bindStartMultiPickerChange(e) {
				this.multiIndex = e.detail.value;
				let da = this.multiArray;
				let di = e.detail.value;
				let caseDate = da[0][di[0]] + ' ' + da[1][di[1]].replace('时', ':') + this.timeFormat(da[2][di[2]].replace('分', ''))


				let appointTime = new Date(caseDate).getTime() / 1000;

				if (appointTime < new Date().getTime() / 1000) {
					uni.showToast({
						title: '不能选择过去时间',
						icon: 'none'
					})
					return false;
				}
				this.$emit('changeTime', caseDate, appointTime * 1000)
			},
		}
	}
</script>

<style lang="scss">

</style>

<template>
	<view class="container">
		<!-- 自定义导航栏 -->
		<view class="navBarBox">
			<!-- 状态栏占位 -->
			<view class="statusBar" :style="{ paddingTop: 'calc(' + statusBarHeight +  'px + 10px);'}"></view>

			<!-- 真正的导航栏内容 -->
			<view class="navBar">
				<!-- 显示选择的日期 -->
				<view class="selected-date" @click="toggleCalendar">
					<text class="selected-text">{{ selectedDate }} ▼</text>
				</view>
			</view>
		</view>

		<!-- 日期选择器遮罩层 -->
		<view v-if="showCalendar" class="calendar-overlay" @click="closeCalendar">
			<view class="calendar-container" @click.stop>
				<!-- 显示当前月份和星期 -->
				<view class="calendar-header">
					<text>{{ currentMonth }}</text>
				</view>
				<view class="calendar-weekdays">
					<view v-for="(weekday, index) in weekdays" :key="index" class="calendar-weekday">
						<text>{{ weekday }}</text>
					</view>
				</view>
				<!-- 日期网格 -->
				<view class="calendar-grid">
					<view class="calendar-day" v-for="day in monthDays" :key="day" @click="selectDay(day)" :class="{
                  'today': isToday(day), 
                  'selected': isSelected(day),
                  'disabled': !isSelectable(day)
                }">
						<text class="calendar-day-text">{{ day }}</text>
					</view>
				</view>
			</view>
		</view>
	</view>

	<view class="paper-container" :style="{ paddingTop: 'calc(' + statusBarHeight +  'px + 70px);'}">
		<image :src="paperImage" class="full-screen-image" mode="widthFix" />
	</view>
</template>


<script>
	import axios from 'axios';
	import {
		parse
	} from 'node-html-parser';

	export default {
		data() {
			const today = new Date();
			const yyyy = today.getFullYear();
			const mm = String(today.getMonth() + 1).padStart(2, '0'); // 当前月份（从1开始）
			const dd = String(today.getDate()).padStart(2, '0'); // 今天的日期
			return {
				selectedDate: `${yyyy}-${mm}-${dd}`, // 默认日期为今天
				statusBarHeight: 40, // 状态栏高度
				showCalendar: false, // 控制是否显示日期网格
				currentMonth: `${yyyy}-${mm}`, // 当前月份
				monthDays: [], // 当前月份的所有日期
				weekdays: ['日', '一', '二', '三', '四', '五', '六'], // 星期的名称
				paperImage: ``,
			};
		},
		methods: {
			// 切换日期网格的显示与隐藏
			toggleCalendar() {
				this.showCalendar = !this.showCalendar;
				if (this.showCalendar) {
					this.updateCalendar(); // 显示日期网格时，更新该月份的所有日期
				}
			},
			// 更新当前月份的日期
			updateCalendar() {
				const selectedDate = new Date(this.selectedDate);
				const currentMonth = selectedDate.getMonth(); // 获取月份
				const currentYear = selectedDate.getFullYear(); // 获取年份
				const firstDay = new Date(currentYear, currentMonth, 1); // 获取当前月的第一天
				const lastDay = new Date(currentYear, currentMonth + 1, 0); // 获取当前月的最后一天
				const daysInMonth = lastDay.getDate(); // 当前月的天数

				// 生成日期数组
				this.monthDays = [];
				for (let i = 1; i <= daysInMonth; i++) {
					this.monthDays.push(i);
				}
			},
			// 选择具体某天的日期
			selectDay(day) {
				const selectedDate =
					`${this.selectedDate.split('-')[0]}-${this.selectedDate.split('-')[1]}-${String(day).padStart(2, '0')}`;

				const today = new Date();
				const selectedDay = new Date(selectedDate);

				// 如果选择的日期是今天或以后，则不更新日期
				if (selectedDay > today) {
					return;
				}

				this.selectedDate = selectedDate;
				this.showCalendar = false; // 选择日期后隐藏日历
				this.updatePaperImage(selectedDate);
			},
			// 关闭日历
			closeCalendar() {
				this.showCalendar = false; // 点击遮罩层关闭日历
			},
			// 判断是否是今天
			isToday(day) {
				const today = new Date();
				return today.getDate() === day && today.getMonth() === new Date(this.selectedDate).getMonth() && today
					.getFullYear() === new Date(this.selectedDate).getFullYear();
			},
			// 判断是否是选择的日期
			isSelected(day) {
				const selectedDay = parseInt(this.selectedDate.split('-')[2], 10); // 获取选中的日期
				return selectedDay === day;
			},
			// 判断是否是可选日期
			isSelectable(day) {
				const today = new Date();
				const currentDay = new Date(this.selectedDate.split('-')[0], this.selectedDate.split('-')[1] - 1, day);
				return currentDay <= today; // 如果是今天或者更早的日期，则可选
			},
			// 更新纸质新闻的图片链接
			updatePaperImage(date) {
				const curDate = date.split('-');
				const imageBaseUrl = `http://paper.people.com.cn/rmrb/pc/layout/` + curDate[0] + curDate[1] + '/' + curDate[2] +
					`/node_01.html`;

				// Send a request to fetch the HTML content
				uni.request({
					url: imageBaseUrl,
					method: 'GET',
					success: (response) => {
						const html = response.data;
						const regex = /<div class=['"][^'"]*paper[^'"]*['"][^>]*>.*?<img[^>]*src=['"](.*?)['"]/s;
						const match = html.match(regex);
						const imgSrc = match[1];
						const fullImageUrl = `http://paper.people.com.cn/rmrb/pc/` + imgSrc.replace('../../../', '');
						this.paperImage = fullImageUrl;  // 设置图片的完整 URL
					},
					fail: (error) => {
						console.error('Request failed:', error);
					}
				});
			}

		},
		mounted() {
			this.updateCalendar(); // 初始化页面时加载当前月份的日期
			this.updatePaperImage(this.selectedDate); // Load paper image for today's date
		}
	}
</script>

<style scoped>
	/* 页面布局 */
	.container {
		display: flex;
		flex-direction: column;
		background-color: #F8F8F8;
		height: 100%;
	}

	/* 自定义导航栏 */
	.navBarBox {
		background-color: #ffffff;
		width: 100%;
		position: fixed;
	}

	.statusBar {
		background-color: #ffffff;
		width: 100%;
	}

	.navBar {
		display: flex;
		align-items: left;
		justify-content: left;
		padding: 10rpx;
		background-color: #ffffff;
		color: black;
		position: relative;
		height: 80rpx;
	}

	/* 选择的日期 */
	.selected-date {
		display: flex;
		/* 启用flexbox布局 */
		justify-content: center;
		/* 水平居中 */
		align-items: center;
		/* 垂直居中 */
		padding: 10rpx;
		font-size: 30rpx;
		color: #000000;
		cursor: pointer;
	}

	.selected-text {
		font-weight: bold;
	}

	/* 遮罩层 */
	.calendar-overlay {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		background-color: rgba(0, 0, 0, 0.5);
		/* 半透明黑色背景 */
		z-index: 1000;
		display: flex;
		justify-content: center;
		align-items: center;
	}

	/* 日期选择器样式 */
	.calendar-container {
		background-color: #ffffff;
		border-radius: 8px;
		padding: 20rpx;
		box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
		max-width: 80%;
		width: 600rpx;
		position: relative;
	}

	.calendar-header {
		text-align: center;
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
		padding-bottom: 20rpx;
	}

	/* 星期的样式 */
	.calendar-weekdays {
		display: grid;
		grid-template-columns: repeat(7, 1fr);
		margin-bottom: 10rpx;
	}

	.calendar-weekday {
		text-align: center;
		font-size: 28rpx;
		font-weight: bold;
		color: #333;
	}

	/* 日期网格 */
	.calendar-grid {
		display: grid;
		grid-template-columns: repeat(7, 1fr);
		gap: 10rpx;
	}

	.calendar-day {
		display: flex;
		justify-content: center;
		align-items: center;
		padding: 15rpx;
		background-color: #ffffff;
		border-radius: 5rpx;
		cursor: pointer;
		font-size: 28rpx;
	}

	.calendar-day:hover {
		background-color: #f0f0f0;
	}

	.calendar-day-text {
		font-size: 28rpx;
	}

	/* 高亮今天的日期 */
	.today {
		color: red;
	}

	/* 选中的日期样式 */
	.selected {
		background-color: red;
		color: white;
	}

	/* 不可选日期 */
	.calendar-day.disabled {
		background-color: #f0f0f0;
		color: #d3d3d3;
		pointer-events: none;
		/* 禁止点击 */
	}

	.paper-container {
		width: 100%;
		height: 100vh;
		/* 视口高度 */
		display: flex;
		justify-content: center;
		/* padding-top: 20px; */
		padding-bottom: 20px;
	}

	.full-screen-image {
		width: 100%;
		height: 100%;
	}
</style>
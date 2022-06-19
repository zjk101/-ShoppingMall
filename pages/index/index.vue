<template>
	<view class="index">
		<!-- #ifdef MP-WEIXIN -->
		<view class="wx-nav">
			<view class="iconfont icon-sousuo"></view>
			<text>百年莱奥</text>
			<view class="iconfont icon-31xiaoxi"></view>
		</view>
		<!-- #endif -->

		<scroll-view scroll-x="true" class='scroll-content' :scroll-into-view="scrollIntoIndex">
			<view class='scroll-item' v-for='(item,index) in topBar' :key='item.id' @tap='changeTab(index)'
				:id="'top'+index">
				<text :class='topBarIndex===index? "f-active-color":"f-color"'>{{item.name}}</text>
			</view>
		</scroll-view>
		<swiper @change='onChangeTab' :current="topBarIndex" :style="'height:'+clentHeight+'px;'">

			<swiper-item v-for='(item,index) in newtopBar' :key='index'>
				<scroll-view scroll-y="true" :style="'height:'+clentHeight+'px;'" @scrolltolower='loadMore(index)'>
					<block v-if="item.data.length>0">
						<block v-for="(k,i) in item.data" :key="i">
							<!-- 首页 -->
							<index-swiper v-if="k.type==='swiperList'" :dataList='k.data'></index-swiper>
							<template v-if="k.type==='recommendList'">
								<recommend :dataList='k.data'></recommend>
								<card cardTitle="猜你喜欢"></card>
							</template>

							<!-- 运动户外页面 -->
							<Banner v-if="k.type==='bannerList'" :dataList='k.imgUrl'></Banner>
							<template v-if="k.type==='iconsList'">
								<Icons :dataList='k.data'></Icons>
								<card cardTitle="热销爆品"></card>
							</template>
							<template v-if="k.type==='hotList'">
								<Hot :dataList='k.data'></Hot>
								<card cardTitle="推荐店铺"></card>
							</template>
							<template v-if="k.type==='shopList'">
								<Shop :dataList='k.data'></Shop>
								<card cardTitle="为您推荐"></card>
							</template>

							<commodity-list v-if="k.type==='commodityList'" :dataList='k.data'></commodity-list>
						</block>
					</block>
					<view v-else>
						暂无数据
					</view>
					<view class='load-text f-color'>
						{{item.loadText}}
					</view>
				</scroll-view>

			</swiper-item>


		</swiper>

	</view>

</template>

<script>
	import $http from '@/common/api/request.js'
	import indexSwiper from '@/components/index/indexswiper.vue'
	import recommend from '@/components/index/Recommend.vue'
	import card from '@/components/common/Card.vue'
	import commodityList from '@/components/common/CommodityList.vue'
	import Banner from '@/components/index/Banner.vue'
	import Icons from '@/components/index/Icons.vue'
	import Hot from '@/components/index/Hot.vue'
	import Shop from '@/components/index/Shop.vue'
	export default {
		data() {
			return {
				title: 'Hello',
				//选中的索引
				topBarIndex: 0,
				scrollIntoIndex: 'top0',
				//内容框的高度
				clentHeight: 0,
				//顶栏数据
				topBar: [],
				newtopBar: []
			}
		},
		components: {
			indexSwiper,
			recommend,
			card,
			commodityList,
			Banner,
			Icons,
			Hot,
			Shop
		},
		onLoad() {
			this.init()

		},
		onReady() {
			uni.getSystemInfo({
				success: (res) => {
					this.clentHeight = res.windowHeight - uni.upx2px(100) - this.getClientHeight();
				}
			})
		},
		// 标题栏按钮点击
		onNavigationBarButtonTap(e) {
			console.log(e);
			if(e.float=='left'){
				uni.navigateTo({
					url:'../search/search'
				})
			}
		},
		methods: {
			// 请求首页数据
			init() {
				$http.request({
					url: "/index_list/data"
				}).then((res) => {
					this.topBar = res.topBar;
					this.newtopBar = this.initData(res);
				}).catch(() => {
					uni.showToast({
						title: '请求失败',
						icon: 'none'
					})
				})
			},
			// 添加数据
			initData(res) {
				let arr = [];
				for (let i = 0; i < this.topBar.length; i++) {
					let obj = {
						data: [],
						load: "first",
						loadText: "上拉加载更多..."
					}
					if (i == 0) {
						obj.data = res.data;
					}
					arr.push(obj);
				};
				return arr;
			},
			// 点击顶栏
			changeTab(index) {
				if (this.topBarIndex === index) {
					return;
				}
				this.topBarIndex = index;
				this.scrollIntoIndex = 'top' + index;
				if (this.newtopBar[this.topBarIndex].load === 'first') {
					this.addData();
				}

			},
			onChangeTab(e) {
				this.changeTab(e.detail.current);
			},
			// 获取可视区域的高度
			getClientHeight() {
				const res = uni.getSystemInfoSync();
				const system = res.platform;
				if (system == 'ios') {
					return 44 + res.statusBarHeight;
				} else if (system == 'android') {
					return 48 + res.statusBarHeight;
				} else {
					return 0;
				}
			},
			// 对应显示不同的数据
			addData(callback) {
				let index = this.topBarIndex;
				let id = this.topBar[index].id;
				// 请求不同的数据
				let page = Math.ceil(this.newtopBar[index].data.length / 5) + 1;
				//请求数据
				$http.request({
					url: '/index_list/' + id + '/data/' + page + ''
				}).then((res) => {
					this.newtopBar[index].data = [...this.newtopBar[index].data, ...res];
				}).catch(() => {
					uni.showToast({
						title: '请求失败',
						icon: 'none'
					})
				})
				this.newtopBar[index].load = 'last';
				if (typeof callback === 'function') {
					callback();
				}
			},
			// 上拉加载更多
			loadMore(index) {
				this.newtopBar[index].loadText = '加载中...';
				this.addData(() => {
					this.newtopBar[index].loadText = '上拉加载更多...';
				})
			}


		}
	}
</script>

<style scoped>
	.wx-nav {
		text-align: center;
		height: 100rpx;
		width: 100%;
		line-height: 100rpx;
		margin-top: 30rpx;
		display: flex;
		justify-content: space-between;
		align-items: center;
	}

	.iconfont {
		padding: 0 20rpx;
	}

	.scroll-content {
		width: 100%;
		height: 100rpx;
		white-space: nowrap;

	}

	.scroll-item {
		display: inline-block;
		padding: 10rpx 30rpx;
		font-size: 36rpx;
	}

	.f-active-color {
		padding: 10rpx 0;
		border-bottom: 6rpx solid #49BDFB;
	}

	.load-text {
		border-top: 2rpx solid #636263;
		line-height: 60rpx;
		text-align: center;
	}
</style>

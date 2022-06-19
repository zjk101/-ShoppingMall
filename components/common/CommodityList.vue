<template>
	<!-- 商品列表组件 -->
	<view class="commodity-list">
		<commodity :dataList="dataList"></commodity>
	</view>
</template>

<script>
	import commodity from './Commodity.vue'
	export default {
		props:{
			dataList:Array
		},
		components: {
			commodity
		}
	}
</script>

<style>
</style>

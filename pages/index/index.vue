<template>
	<view class="index">
		<drop-down :filterData="filterData" :defaultVal="defaultVal" @confirm="confirm" @reset="reset"></drop-down>
	</view>
</template>

<script>
	import data from '@/common/data.js'; //筛选菜单数据
	export default {
		data() {
			return {
				filterData: [],
				defaultVal: [0, 1, 2, 3]
			};
		},
		onLoad() {
			this.filterData = data;
		},
		methods: {
			confirm(e) {
				console.log('eeee', e);
			},
			reset(val) {
				this.defaultVal = val
			}
		},
	}
</script>
<style lang="scss" scoped>

</style>

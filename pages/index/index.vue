<template>
	<view class="index">
		<drop-down :filterData="filterData" @confirm="confirm" @reset="reset">
			<template #title="{title}">
				<view>{{showTitle(title)}}</view>
			</template>
		</drop-down>
	</view>
</template>

<script>
	import data from '@/common/data.js'; //筛选菜单数据
	export default {
		data() {
			return {
				filterData: [],
				fileds: 'name'
			};
		},
		onLoad() {
			this.filterData = data;
		},
		computed: {
			showTitle() {
				return (item) => {
					return (item.checkedName && item.checkedName.length) ? item.checkedName.join('/') : item[this
						.fileds]
				}
			},
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

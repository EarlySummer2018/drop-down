<template>
	<view class="drop-item">
		<!-- 多选渲染 -->
		<block v-if="type=='filter'">
			<scroll-view class="menu-box" :scroll-y="true">
				<view class="box" v-for="pItem in newItem[childName]" :key="pItem.drop_item_key">
					<view class="title">{{pItem[fileds]}}</view>
					<view class="labels">
						<view v-for="cItem in pItem[childName]" :key="cItem.drop_item_key"
							@click="selectFilterLabel(cItem,cItem.drop_item_key)"
							:class="{'on':filter.findIndex(el=>el==cItem.drop_item_key)!==-1}">
							{{cItem[fileds]}}
						</view>
					</view>
				</view>
			</scroll-view>
		</block>

		<!-- 单选渲染 -->
		<block v-else-if="type=='radio'">
			<scroll-view class="menu-box" :scroll-y="true">
				<view class="box" v-for="pItem in newItem[childName]" :key="pItem.drop_item_key">
					<view class="title">{{pItem[fileds]}}</view>
					<view class="labels">
						<view v-for="cItem in pItem[childName]" :key="cItem.drop_item_key"
							@tap="selectRadioLabel(cItem,pItem.drop_item_key)"
							:class="{'on':radio.findIndex(el=>el.drop_item_key==cItem.drop_item_key)!==-1}"
							:cIndex="cItem.drop_item_key">
							{{cItem[fileds]}}
						</view>
					</view>
				</view>
			</scroll-view>

		</block>
	</view>
</template>

<script>
	import {
		guid
	} from '../../lib/guid.js'
	export default {
		/**
		 * @property {Object}						item			当前项
		 * @property {String}						fileds  		显示的字段（默认 name）
		 * @property {String}						childName  		子菜单字段（默认 submenu）
		 * @property {String}						type  			类型	（默认 filter，可选项 radio）
		 * */

		props: {
			item: {
				type: Object,
				default () {
					return {}
				}
			},
			fileds: {
				type: String,
				default: "name"
			},
			childName: {
				type: String,
				default: 'submenu'
			},
			type: {
				type: String,
				default: 'filter'
			},
		},
		created() {
			this.init()
		},
		data() {
			return {
				filter: [],
				radio: [],
				newItem: [],

				// 多选选中数组
				selectFilterArr: [],

				// 单选选中数组
				selectRadioArr: [],
			}
		},
		methods: {
			init() {
				this.newItem = JSON.parse(JSON.stringify(this.item))
				this.newItem.drop_item_key = guid()
				this.setKey(this.newItem)
			},
			setKey(item, pKey = '') {
				let child = item[this.childName]
				if (child && child.length) {
					child.forEach(el => {
						el.drop_item_key = guid()
						el.checked = el.checked ? el.checked : false
						el.parent_type = this.type
						if (el.checked) {
							if (el.parent_type === 'filter') {
								this.selectFilterArr.push(el)
								this.filter.push(el.drop_item_key)
							} else if (el.parent_type === 'radio') {
								this.selectRadioLabel(el, pKey)
							}
						}
						this.setKey(el, el.drop_item_key)
					})
				} else {
					item[this.childName] = []
					this.deepIndex = -1
				}
			},

			// 多选
			selectFilterLabel(item, key) {
				// 已选中，已选中 ? 移除选中 : 添加选中
				const sIdx = this.selectFilterArr.findIndex(el => el.drop_item_key == key)
				if (sIdx >= 0) {
					item.checked = false
					this.selectFilterArr.splice(sIdx, 1)
				} else {
					item.checked = true
					this.selectFilterArr.push(item)
				}

				// 已选中，已选中 ? 移除选中的 key : 添加选中的 key
				const fIdx = this.filter.findIndex(el => el == key)
				if (sIdx >= 0) this.filter.splice(fIdx, 1)
				else this.filter.push(key)
			},

			// 单选
			selectRadioLabel(item, key) {
				// 已选中 ? 移除选中 : 添加选中
				const sIdx = this.selectRadioArr.findIndex(el => el.pKey == key)
				if (sIdx >= 0 && this.selectRadioArr[sIdx].drop_item_key === item.drop_item_key) {
					this.selectRadioArr
						.splice(sIdx, 1)
					item.checked = false
				} else if (sIdx >= 0 && this.selectRadioArr[sIdx].drop_item_key !== item.drop_item_key) {
					this.selectRadioArr[sIdx].checked = false
					item.checked = true
					this.selectRadioArr
						.splice(sIdx, 1, {
							pKey: key,
							...item
						})
				} else {
					item.checked = true
					this.selectRadioArr.push({
						pKey: key,
						...item
					})
				}


				// 选中之前判断是否已经选中，已选中 ? radio = -1 : radio = key
				const fIdx = this.radio.findIndex(el => el.pKey == key)
				if (fIdx >= 0 && this.radio[fIdx].drop_item_key === item.drop_item_key) {
					this.radio.splice(fIdx, 1)
				} else if (fIdx >= 0 && this.radio[fIdx].drop_item_key !== item.drop_item_key) {
					this.radio.splice(fIdx, 1, {
						pKey: key,
						drop_item_key: item.drop_item_key
					})
				} else {
					this.radio.push({
						pKey: key,
						drop_item_key: item.drop_item_key
					})
				}
			},
		}
	}
</script>

<style lang="scss" scoped>
	view {
		display: flex;
		flex-wrap: nowrap;
	}

	.drop-item {
		width: 100%;
		align-items: center;
		flex-direction: column;

		.title {
			width: 100%;
			font-size: 26rpx;
			color: #888;
		}

		.labels {
			flex-direction: row;
			flex-wrap: wrap;

			.on {
				border-color: $uni-color-primary;
				background-color: $uni-color-primary;
				color: #fff;
			}

			>view {
				width: calc((698rpx - 30rpx * 3) / 4 - 1rpx * 2);
				height: 60rpx;
				border: solid 1rpx #adadad;
				border-radius: 4rpx;
				margin-right: 20rpx;
				margin-top: 16rpx;
				font-size: 24rpx;
				flex-direction: row;
				justify-content: center;
				align-items: center;
			}
		}

		.menu-box {
			width: 698rpx;
			height: calc(690rpx - 120rpx);
			flex-shrink: 1;

			.box {
				width: 100%;
				padding-top: 32rpx;
				flex-direction: column;
			}
		}
	}
</style>

<template>
	<view class="drop-first-item">
		<!-- 多级菜单 -->
		<block v-if="(newItem.type=='hierarchy'||newItem.type=='hierarchy-column')&& newItem[childName].length>0">
			<!-- 第一级菜单 -->
			<scroll-view class="sub-menu-list" :class="[newItem[childName].length>0?'first':'alone']" :scroll-y="true"
				:scroll-into-view="'first_id'+firstScrollInto">
				<block v-for="sub in newItem[childName]" :key="sub.drop_item_key">
					<view class="sub-menu" :class="{'on':current==sub.drop_item_key}" @click="selectFirst(sub)">
						<view class="menu-name">
							<text>{{sub[fileds]}}</text>
							<text class="iconfont selected"></text>
						</view>
					</view>
				</block>
			</scroll-view>
			<block v-if="newItem.type=='hierarchy'">
				<block v-for="sub in newItem[childName]" :key="sub.drop_item_key">
					<!-- 第二级菜单 -->
					<scroll-view class="sub-menu-list not-first" :scroll-y="true"
						v-if="current==sub.drop_item_key&&sub[childName].length>0"
						:scroll-into-view="'second_id'+secondScrollInto">
						<block v-for="sub_second in sub[childName]" :key="sub_second.drop_item_key">
							<view class="sub-menu" :class="{'on':sec_current==sub_second.drop_item_key}">
								<view class="menu-name" @tap="selectSec(sub_second)">
									<text>{{sub_second[fileds]}}</text>
									<text class="iconfont selected"></text>
								</view>
								<!-- 第三级菜单 -->
								<view class="more-sub-menu"
									v-if="sub[childName].length>0&&sub_second[childName].length>0">
									<block v-for="sub2 in sub_second[childName]" :key="sub2.drop_item_key">
										<text :class="{'on':sec2_current==sub2.drop_item_key}"
											@click="selectSec2(sub2)">{{sub2[fileds]}}</text>
									</block>
								</view>
							</view>
						</block>
					</scroll-view>
				</block>
			</block>

			<!-- 二三级菜单都是行形态 -->
			<block v-else-if="newItem.type=='hierarchy-column'">
				<!-- 第二级菜单 -->
				<block v-for="sub in newItem[childName]" :key="sub.drop_item_key">
					<scroll-view class="sub-menu-list not-first" :scroll-y="true"
						v-if="current==sub.drop_item_key&&sub[childName].length>0"
						:scroll-into-view="'second_id'+secondScrollInto">
						<block v-for="sub_second in sub[childName]" :key="sub_second.drop_item_key">
							<view class="sub-menu" :class="{'on':sec_current==sub_second.drop_item_key}">
								<view class="menu-name" @click="selectSec(sub_second)">
									<text>{{sub_second[fileds]}}</text>
								</view>
							</view>
						</block>
					</scroll-view>
				</block>
				<!-- 第三级菜单 -->
				<block v-for="sub in newItem[childName]" :key="sub.drop_item_key">
					<block v-for="sub_second in sub[childName]" :key="sub_second.drop_item_key">
						<scroll-view class="sub-menu-list not-first third" :scroll-y="true"
							v-if="current==sub.drop_item_key&&sec_current==sub_second.drop_item_key&&sub_second[childName].length>0"
							:scroll-into-view="'third_id'+thirdScrollInto">
							<block v-for="sub2 in sub_second[childName]" :key="sub2.drop_item_key">
								<view class="sub-menu" :class="{'on':sec2_current==sub2.drop_item_key}">
									<view class="menu-name" @click="selectSec2(sub2)">
										<text>{{sub2[fileds]}}</text>
									</view>
								</view>
							</block>
						</scroll-view>
					</block>
				</block>
			</block>
		</block>
	</view>
</template>

<script>
	import {
		guid
	} from '../../lib/guid.js'
	export default {
		/**
		 * @property {Object}										item					循环对象
		 * @property {Number, null}									firstScrollInto  		滚动区域定位，不需要开发者传值
		 * @property {Number, null}									secondScrollInto  		滚动区域定位，不需要开发者传值
		 * @property {Number, null}									thirdScrollInto  		滚动区域定位，不需要开发者传值
		 * @property {String}										childName				子级菜单名称，默认 submenu
		 * @property {String}										fileds					子级菜单名称，默认 name
		 * @property {Function() <Object:{show,item}> {}}			first  					返回选中的一级菜单对象，返回值为show和item，show 为否有下级菜单，item 为当前选中项
		 * @property {Function() <Object:{show,item}> {}}			second  				返回选中的二级菜单对象，返回值为show和item，show 为否有下级菜单，item 为当前选中项
		 * @property {Function() <Object:{show,item}> {}}			third  					返回选中的三级菜单对象，返回值为show和item，show 为否有下级菜单，item 为当前选中项
		 * */
		props: {
			item: {
				type: Object,
				default () {
					return {}
				}
			},
			firstScrollInto: {
				type: [Number, null],
				default: 0
			},
			secondScrollInto: {
				type: [Number, null],
				default: 0
			},
			thirdScrollInto: {
				type: [Number, null],
				default: 0
			},
			currentIndex: {
				type: Number,
				default: 0
			},
			childName: {
				type: String,
				default: 'submenu'
			},
			fileds: {
				type: String,
				default: 'name'
			},
			autoStow: {
				type: Boolean,
				default: true
			},
		},
		data() {
			return {
				current: -1,
				sec_current: -1,
				sec2_current: -1,
				newItem: [],
				deepIndex: 0,
				attr: ['current', 'sec_current', 'sec2_current'],
				selectArr: [],
			}
		},
		created() {
			this.init()
		},
		methods: {
			init() {
				this.newItem = JSON.parse(JSON.stringify(this.item))
				this.newItem.drop_item_key = guid()
				this.setKey(this.newItem)
				if (!this.selectArr || !this.selectArr.length) return
				this.$emit('change', this.selectArr)
			},
			setKey(item) {
				let child = item[this.childName]
				if (child && child.length) {
					child.forEach(el => {
						el.drop_item_key = guid()
						el.checked = el.checked ? el.checked : false
						if (el.checked) {
							this[this.attr[el.drop_item_identity - 1]] = el.drop_item_key
							this.selectArr.push(el)
						}
						this.setKey(el)
					})
				} else {
					item[this.childName] = []
					this.deepIndex = -1
				}
			},
			selectFirst(data) {
				this.result(data, 'current')
				this.sec_current = -1
				this.sec2_current = -1
				this.isExist(data, 0)
			},
			selectSec(data) {
				this.result(data, 'sec_current')
				this.sec2_current = -1
				this.isExist(data, 1)
			},
			selectSec2(data) {
				this.result(data, 'sec2_current')
				this.isExist(data, 2)
			},

			result(data, filed) {
				if (this[filed] !== data.drop_item_key) {
					this[filed] = data.drop_item_key
				} else {
					if (this.autoStow) return
					this[filed] = -1
				}
			},

			// 判断是否已经选择,如果已选中就剔除,否则选中
			isExist(item, type) {
				const current = this.selectArr[type]
				if (current && current.drop_item_key === item.drop_item_key) {
					item.checked = false
					if (type === 0) {
						this.selectArr = []
						this.current = -1
					} else {
						if (type === 1) this.sec_current = -1
						else if (type === 2) this.sec2_current = -1
						this.selectArr.splice(type, this.selectArr.length - 1)
					}
				} else if (current && current.drop_item_key !== item.drop_item_key) {
					this.selectArr.splice(type + 1, this.selectArr.length - 1)
					this.selectArr.fill(item, type, type + 1)
					this.selectArr[type].checked = false
					item.checked = true
				} else {
					this.selectArr.push(item)
					item.checked = true
				}
				const show = (!item[this.childName] || !item[this.childName].length) ? true : false
				this.$emit("close", show)
				this.$emit('change', this.selectArr)
			}
		}
	}
</script>

<style lang="scss" scoped>
	view {
		display: flex;
		flex-wrap: nowrap;
	}

	.drop-first-item {
		width: 100%;
	}

	.sub-menu-list {
		width: 100%;
		height: calc(690rpx - 150rpx);
		flex-direction: column;

		.sub-menu {
			min-height: 44px;
			font-size: 13px;
			flex-direction: column;
			padding-right: 15px;

			>.menu-name {
				height: 44px;
				flex-direction: row;
				align-items: center;
				justify-content: space-between;

				>.iconfont {
					display: none;
					font-size: 18px;
					color: $uni-color-primary;
				}
			}
		}

		&.first {
			flex-shrink: 0;
			width: 236rpx;
			background-color: #f0f0f0;

			.sub-menu {
				padding-left: 15px;

				&.on {
					color: $uni-color-primary;
					background-color: #fff;
				}
			}

			box-shadow: 5rpx 0 5rpx rgba($color: #000000, $alpha: 0.1);
		}

		&.alone {
			max-height: 345px;
			min-height: 170px;
			height: auto;

			.sub-menu {
				min-height: calc(44px - 1rpx);
				margin-left: 15px;
				border-bottom: solid 1rpx #e5e5e5;

				&.on {
					color: $uni-color-primary;

					>.menu-name {
						>.iconfont {
							display: block;
						}
					}
				}
			}
		}

		&.not-first {
			box-shadow: 5rpx 0 5rpx rgba($color: #000000, $alpha: 0.1);

			.sub-menu {
				min-height: calc(44px - 1rpx);
				margin-left: 15px;
				border-bottom: solid 1rpx #e5e5e5;

				>.menu-name {
					height: calc(44px - 1rpx);

					>.iconfont {
						display: none;
						font-size: 18px;
						color: $uni-color-primary;
					}
				}

				&.on {
					color: $uni-color-primary;

					>.menu-name {
						>.iconfont {
							display: block;
						}
					}
				}

				.more-sub-menu {
					flex-direction: row;
					flex-wrap: wrap;
					padding-bottom: 9px;

					>text {
						height: 30px;
						border-radius: 3px;
						background-color: #f5f5f5;
						color: #9b9b9b;
						margin-bottom: 6px;
						margin-right: 6px;
						text-align: center;
						line-height: 30px;
						border: solid #f5f5f5 1rpx;
						flex: 0 0 calc(33.33% - 6px);
						overflow: hidden;
						font-size: 12px;

						&:nth-child(3n) {
							margin-right: 0;
						}

						&.on {
							border-color: $uni-color-primary;
							color: $uni-color-primary;
						}

						.iconfont {
							color: #9b9b9b;
						}
					}
				}
			}
		}
	}

	.filter {
		width: 100%;
		height: 345px;
		display: flex;
		flex-direction: column;
		justify-content: space-between;
		align-items: center;

		.menu-box {
			width: 698rpx;
			height: calc(345px - 75px);
			flex-shrink: 1;

			.box {
				width: 100%;
				padding-top: 16px;
				flex-direction: column;

				.title {
					width: 100%;
					font-size: 13px;
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
						height: 30px;
						border: solid 1rpx #adadad;
						border-radius: 2px;
						margin-right: 30rpx;
						margin-top: 8px;
						font-size: 12px;
						flex-direction: row;
						justify-content: center;
						align-items: center;

						&:nth-child(4n) {
							margin-right: 0;
						}
					}
				}
			}
		}

		.btn-box {
			flex-shrink: 0;
			width: 698rpx;
			height: 75px;
			flex-direction: row !important;
			align-items: center;
			justify-content: space-between;

			>view {
				width: 320rpx;
				height: 40px;
				border-radius: 40px;
				border: solid 1rpx $uni-color-primary;
				align-items: center;
				justify-content: center;
			}

			.reset {
				color: $uni-color-primary;
			}

			.submit {
				color: #fff;
				background-color: $uni-color-primary;
			}
		}
	}

	.mask {
		z-index: 10;
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		background-color: rgba(0, 0, 0, 0);
		transition: background-color .15s linear;

		&.show {
			background-color: rgba(0, 0, 0, 0.5);
		}

		&.hide {
			display: none;
		}
	}

	/* 字体图标 */
	@font-face {
		font-family: "HM-FD-font";
		src: url('data:application/x-font-woff2;charset=utf-8;base64,d09GMgABAAAAAALAAAsAAAAABpQAAAJzAAEAAAAAAAAAAAAAAAAAAAAAAAAAAAAAHEIGVgCDBgp4gQIBNgIkAwwLCAAEIAWEbQc5G8sFERWMIbIfCbbzqA4hp7InSBibVsYGb4J42o82b3e/nJlHMw/NHbGOlwKJRCRpwzPtpAECCOZubdqxjYpQLMlVg+70/08edrgQOtx2ukpVyApZn+dyehPoQObHo3O85rYx9vOjXoBxQIHugW2yIkqIW2QXcScu4jwE8CSWbKSmrqUHFwOaJoCsLM5P4haSGIxRcRHshrUGucLCVcfqI3AZfV/+USguKCwNmtsxVztDxU/n55C+3W0Z4QQpEOTNFqCBbMCAjDUWB9CIwWk87aa70cYgqLkyd3dEmm+18R8eKATEBrV7A5CulBT8dKiWOYZk412XNcDdKSEKSGODnyKIDl+dmVt9/Dx4pu/xyeutkMlHISGPTsPCnoTNP9nOT6wTtDdlO6dPr47efvj942lkYuQzrhMKEjq9N6y98P3340gmlJ/RStUD6F31CAEEPtUW94/7rf+7XgaAz57X0ZHXAGsFFwVgw38yALuMb0IBbVyNamFYEw4oKMDTj3AHRQP5Pt4dci9VwSVkRNQh5r7CLskZadhsWHhRDBsXczk8ZYk3ewnCxmQeQKa3BOHvA8XXO2j+vqRhf7CE+sPmn4anvoL29JLa4qqaUQkmoK+QG2osCckq7txi2leK86aIPyJ3eQZ8xytXYmyQ51jQndJAxIJlqiGSLsOqImiZCjTiZCJt6Lq26U2OoXqwUo0hRaAE0K5AziANy/uLVeXzWyjVqyjcoeupjxDr5MMDn8MDkLG9Aenu5ZrOSSoghAUsRmogkkahSoWAtnlUARnCkY3It0Iu7mWhdmd9Z/19BwBP6GidEi0G56opckXTGZVSPxgAAAA=');
	}

	.iconfont {
		font-family: "HM-FD-font" !important;
		font-size: 13px;
		font-style: normal;
		color: #757575;

		&.triangle {
			&:before {
				content: "\e65a";
			}
		}

		&.selected {
			&:before {
				content: "\e607";
			}
		}
	}
</style>

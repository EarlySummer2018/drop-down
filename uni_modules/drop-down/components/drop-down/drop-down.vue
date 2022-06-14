<template>
	<view class="drop-down" :class="{'setDropdownBottom':maskVisibility}">
		<!-- 顶部插槽 -->
		<view class="header">
			<slot name="header"></slot>
		</view>
		<view class="main">
			<!-- 顶部菜单 -->
			<view class="nav">
				<block v-for="(item,index) in menu" :key="index">
					<view class="first-menu" :class="{'on':current==index}" @click="togglePage(index)">
						<text class="name">{{item[fileds]}}</text>
					</view>
				</block>
			</view>
			<!-- 遮罩层 -->
			<view class="mask" :class="{'show':isShowMask,'hide':maskVisibility!=true}" @click="hideMenu(false)"></view>
			<block v-for="(item,index) in menuData" :key="index">
				<view class="sub-menu-class" :class="{'show':current==index,'hide':!pageState[index]}">
					<block v-if="(item.type=='hierarchy'||item.type=='hierarchy-column')&& item[childName].length>0">
						<drop-item-menu :item="item" :firstScrollInto="firstScrollInto"
							:secondScrollInto="secondScrollInto" :thirdScrollInto="thirdScrollInto"
							:childName="childName" :fileds="fileds" @first="first" @second="second" @third="third"
							ref="itemMenu" style="width: 100%;">
						</drop-item-menu>
					</block>
					<!-- 多选筛选 -->
					<block v-if="item.type=='filter'">
						<view class="filter">
							<drop-item :item="item" @change="changeFilterItem" :childName="childName" :fileds="fileds">
							</drop-item>
						</view>
					</block>

					<!-- 单选筛选 -->
					<block v-if="item.type=='radio'">
						<view class="filter">
							<drop-item :item="item" type="radio" @change="changeRadioItem" :childName="childName"
								:fileds="fileds"></drop-item>
						</view>
					</block>
					<view class="btn-box" v-if="!autoStow||item.type=='filter'||item.type=='radio'">
						<view class="reset" @click="resetFilter()">重置</view>
						<view class="submit" @click="confirmFilter()">确定</view>
					</view>
				</view>
			</block>
		</view>
	</view>
</template>
<script>
	export default {
		name: 'DropDown',
		/**
		 * @property {Array}						filterData  		渲染的数组（默认 []）
		 * @property {String}						childName  			子菜单字段（默认 submenu）
		 * @property {String}						fileds  			显示字段	（默认 name）
		 * @property {Boolean}		  				isChild				返回值是否包含子菜单 （默认 false，可选 true）
		 * @property {Boolean}		  				autoStow			菜单类型为 hierarchy 或 hierarchy-column 时选择完成之后是否自动收起菜单并返回选中值，默认 true，选择所有子																		  菜单后再收起，如果有多级时，希望选择一级或二级菜单时就收起，需要设置为 false，点击确定收起
		 * @property {Boolean}						resetStow  			重置是否收起，默认 false，可选 true
		 * @property {Function() <array> {}}		confirm  			返回所有选中的对象
		 * */
		props: {
			filterData: {
				value: Array,
				default: []
			},
			childName: {
				type: String,
				default: 'submenu'
			},
			fileds: {
				type: String,
				default: "name"
			},
			isChild: {
				type: Boolean,
				default: false
			},
			autoStow: {
				type: Boolean,
				default: false
			},
			resetStow: {
				type: Boolean,
				default: false
			}
		},
		data() {
			return {
				menuData: [],
				menu: [], //顶部菜单数据
				current: -1, //菜单页面显示/隐藏动画控制
				pageState: [], //页面的状态
				isShowMask: false, //遮罩层显示/隐藏动画控制
				maskVisibility: false, //遮罩层显示/隐藏状态

				//滚动区域定位
				firstScrollInto: 0,
				secondScrollInto: 0,
				thirdScrollInto: 0,


				// 多列选择
				columns: [],

				// 单选/多选
				filter: [],
				radio: [],
			}
		},
		watch: {
			/**
			 * @ desc 由于筛选条件可能是深层对象且是通过请求过来的，所以采用 watch 进行深度监听，使其及时更新
			 * */
			filterData: {
				handler(newVal) {
					this.menuData = JSON.parse(JSON.stringify(newVal));
					this.init();
					this.addIdentity(this.menuData, 0)
				},
				immediate: true,
				deep: true
			},
		},
		created() {
			this.init();
			this.addIdentity(this.menuData, 0)
		},
		methods: {
			init() {
				let tmpMenu = [];
				for (let i = 0; i < this.menuData.length; i++) {
					let tmpitem = this.menuData[i];
					tmpMenu.push({
						[this.fileds]: tmpitem[this.fileds] || (tmpitem.type == "filter" ? "筛选" : tmpitem[this
							.childName] ? tmpitem[
							this.childName][0][this.fileds] : '筛选'),
						type: tmpitem.type
					});
					this.pageState.push(false);
				}

				// 提取 nav 
				this.menu = tmpMenu;
			},

			// 为每一项添加标识
			addIdentity(data, level = 0) {
				if (!data || !data.length) return
				data.forEach(item => {
					item.drop_item_identity = level
					if (item[this.childName] && item[this.childName].length) {
						this.addIdentity(item[this.childName], level + 1)
					}
				})
			},

			//写入结果，筛选
			confirmFilter() {
				const data = [...this.filter, ...this.radio, ...this.columns]
				console.log(data)
				this.$emit('confirm', data)
				this.closeMeun(true)
			},
			//重置结果和ui，筛选
			resetFilter(page_index) {
				const items = this.$refs.itemMenu
				items.forEach(item => {
					item.current = -1
					item.sec_current = -1
					item.sec2_current = -1
				})
				if (this.resetStow) this.closeMeun(this.resetStow)
				this.$emit('reset', [])
				this.closeMeun(true)
			},

			//菜单开关
			togglePage(index) {
				if (this.isToggleing) {
					return;
				}
				this.isToggleing = true;
				setTimeout(() => {
					this.isToggleing = false;
				}, 150)
				if (index == this.current) {
					this.hideMenu();
				} else {
					this.showMenu(index)
				}
			},


			// 隐藏菜单
			hideMenu(isTriggerConfirm) {
				this.hideMenuLayer(true);
				this.hideMaskLayer();
				this.current = -1;
			},

			// 显示菜单
			showMenu(index) {
				if (this.current > -1) {
					this.hideMenuLayer(false);
				}
				this.showMenuLayer(index);
				this.showMaskLayer();

			},

			//隐藏遮罩层
			hideMaskLayer() {
				this.isShowMask = false;
				setTimeout(() => {
					this.maskVisibility = false;
				}, 200);
			},

			//显示遮罩层
			showMaskLayer() {
				this.maskVisibility = true;
				this.$nextTick(() => {
					setTimeout(() => {
						this.isShowMask = true;
					}, 0)
				})
			},

			//隐藏菜单页
			hideMenuLayer(isAnimation) {
				let tmpIndex = this.current;
				if (isAnimation) {
					setTimeout(() => {
						this.pageState.splice(tmpIndex, 1, false);
					}, 200);
				} else {
					this.pageState.splice(tmpIndex, 1, false)
				}
				this.firstScrollInto = null;
				this.secondScrollInto = null;
			},

			//显示菜单页
			showMenuLayer(index) {
				this.pageState.splice(index, 1, true);
				this.$nextTick(() => {
					setTimeout(() => {
						this.current = index;
					}, 0)
				})
			},

			// 选中filter类型item中的每一项
			changeFilterItem(val) {
				this.filter = this.formatResult(val)
			},

			// 选中radio类型item中的每一项
			changeRadioItem(val) {
				this.radio = this.formatResult(val)
			},

			// 处理结果并返回
			formatResult(val) {
				const result = JSON.parse(JSON.stringify(val))
				result.forEach(el => {
					delete el.drop_item_key
					delete el.pKey
					delete el.drop_item_identity
					delete el.parent_type
				})

				// 判断是否需要返回子菜单
				if (!this.isChild) {
					result.forEach(el => {
						delete el[this.childName]
					})
				}
				return result
			},

			first(obj) {
				this.columns = []
				if (obj.item.drop_item_key) this.columns.push(obj.item)
				else this.columns.splice(0, 1)
				if (this.autoStow && obj.show) {
					this.confirmFilter()
				}
			},
			second(obj) {
				if (this.columns[2]) this.columns.splice(2, 1)
				if (obj.item.drop_item_key) this.columns.splice(1, 1, obj.item)
				else this.columns.splice(1, 1)
				if (this.autoStow && obj.show) {
					this.confirmFilter()
				}
			},
			third(obj) {
				if (obj.item.drop_item_key) this.columns.splice(2, 1, obj.item)
				else this.columns.splice(2, 1)
				if (this.autoStow && obj.show) {
					this.confirmFilter()
				}
			},

			closeMeun(show) {
				if (show) this.hideMenu(true);
			}
		}
	}
</script>
<style lang="scss" scoped>
	view {
		display: flex;
		flex-wrap: nowrap;
	}

	.drop-down {
		flex-shrink: 0;
		width: 100%;
		position: fixed;
		// position: sticky;
		z-index: 997;
		flex-wrap: nowrap;
		display: flex;
		flex-direction: row;
		top: var(--window-top);
		left: 0;
		// top:100px;
		overflow-y: hidden;
		flex-direction: column;

		.deg0 {
			transform: rotateZ(0deg);
		}

		.deg180 {
			transform: rotateZ(180deg);
		}

		&.setDropdownBottom {
			// height: 345px;
			bottom: 0;
		}

	}

	.header {
		z-index: 12;
		background-color: #fff;
	}

	.nav {
		width: 100%;
		height: 88upx;
		border-bottom: solid 2upx #eee;
		z-index: 12;
		background-color: #ffffff;
		flex-direction: row;

		.first-menu {
			width: 100%;
			font-size: 26upx;
			color: #757575;
			flex-direction: row;
			align-items: center;
			justify-content: center;
			transition: color .2s linear;

			&.on {
				color: $uni-color-primary;

				.iconfont {
					color: $uni-color-primary;
				}

				.name::before {
					background-color: $uni-color-primary;
					transform: translate(150%, -50%) rotateZ(40deg);
				}

				.name::after {
					background-color: $uni-color-primary;
					transform: translate(230%, -50%) rotateZ(-40deg);
				}
			}

			.name {
				position: relative;
				height: 40upx;
				text-align: center;
				text-overflow: clip;

				// overflow: hidden;
				&::after,
				&::before {
					content: "";
					position: absolute;
					right: 0;
					top: 50%;
					width: 10upx;
					height: 5upx;
					background-color: #757575;
					transition: transform 0.3s linear;
				}

				&::before {
					transform: translate(150%, -50%) rotateZ(-40deg);
				}

				&::after {
					transform: translate(230%, -50%) rotateZ(40deg);
				}
			}
		}
	}

	.sub-menu-class {
		width: 100%;
		position: absolute;
		left: 0;
		transform: translate3d(0, - 100%, 0);
		max-height: 690upx;
		background-color: #ffffff;
		z-index: 11;
		box-shadow: 0 10upx 10upx rgba(0, 0, 0, .1);
		overflow: hidden;
		align-items: center;
		flex-direction: column;
		transition: transform .15s linear;

		&.hide {
			display: none;
		}

		&.show {
			transform: translate3d(0, calc(88upx + 1upx), 0);
		}

		.btn-box {
			flex-shrink: 0;
			width: 698rpx;
			height: 120upx;
			flex-direction: row !important;
			align-items: center;
			justify-content: space-between;

			>view {
				width: 320rpx;
				height: 80upx;
				border-radius: 80upx;
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

	.filter {
		width: 100%;
		height: 600upx;
		display: flex;
		flex-direction: column;
		justify-content: space-between;
		align-items: center;

		.menu-box {
			width: 698rpx;
			height: calc(690upx - 120upx);
			flex-shrink: 1;

			.box {
				width: 100%;
				padding-top: 32upx;
				flex-direction: column;
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
</style>

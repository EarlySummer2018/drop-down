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
						<text class="txt" :class="{'checked-color': checkedColor(item)}">
							<slot name="title" :title="item">
								{{showTitle(item)}}
							</slot>
						</text>
						<text class="name"></text>
					</view>
				</block>
			</view>
			<!-- 遮罩层 -->
			<view class="mask" v-if="overlay" :class="{'show':isShowMask,'hide':maskVisibility!=true}"
				@click="hideMenu(true, 'mask')"></view>
			<block v-for="(item,index) in menuData" :key="index">
				<view class="sub-menu-class"
					:class="{'show':current==index,'hide':!pageState[index], 'show-shadow':shadow}">
					<block v-if="(item.type=='hierarchy'||item.type=='hierarchy-column')&& item[childName].length>0">
						<drop-item-menu :item="item" :currentIndex="index" :firstScrollInto="firstScrollInto"
							:secondScrollInto="secondScrollInto" :thirdScrollInto="thirdScrollInto"
							:childName="childName" :fileds="fileds" ref="itemMenu" style="width: 100%;"
							@close="emitColse" @change="changeMenuTitle($event, index)">
						</drop-item-menu>
					</block>
					<!-- 多选筛选 -->
					<block v-if="item.type=='filter'">
						<view class="filter">
							<drop-item :item="item" :childName="childName" :fileds="fileds" ref="dropItem">
							</drop-item>
						</view>
					</block>

					<!-- 单选筛选 -->
					<block v-if="item.type=='radio'">
						<view class="filter">
							<drop-item :item="item" type="radio" :childName="childName" :fileds="fileds" ref="dropItem">
							</drop-item>
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
		 * @property {Boolean}						overlay  			是否显示遮罩，默认 true，可选 false
		 * @property {Boolean}						closeOnClickOverlay 点击遮罩是否收起菜单，默认 true，可选 false
		 * @property {Boolean}						shadow  			是否显示菜单下阴影，默认 true，可选 false
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
				default: true
			},
			resetStow: {
				type: Boolean,
				default: false
			},
			overlay: {
				type: Boolean,
				default: true
			},
			closeOnClickOverlay: {
				type: Boolean,
				default: true
			},
			shadow: {
				type: Boolean,
				default: true
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

				result: [],
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
		computed: {
			showTitle() {
				return (item) => {
					return (item.checkedName && item.checkedName.length) ? item.checkedName.join('/') : item[this
						.fileds]
				}
			},
			checkedColor() {
				return (item) => {
					return item.checkedName && item.checkedName.length
				}
			}
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
						type: tmpitem.type,
						checkedName: []
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

			//写入结果，合并
			confirmFilter() {
				this.result = []
				this.getListValue()
				this.getFilterOrRadioValue()
				this.formatResult()
				this.hideMenu(false)
			},
			//重置结果和ui，筛选
			resetFilter(page_index) {
				const components1 = this.$refs.itemMenu
				if (!components1 || !components1.length) return
				components1.forEach(component => {
					component.current = -1
					component.sec_current = -1
					component.sec2_current = -1
					component.selectArr = []
				})
				const components2 = this.$refs.dropItem
				if (!components2 || !components2.length) return
				components2.forEach(component => {
					component.selectFilterArr = []
					component.selectRadioArr = []
					component.filter = []
					component.radio = []
				})
				if (this.resetStow) this.hideMenu(true)
				this.confirmFilter()
			},

			// 取类型为 filter 或 radio 的选中值
			getFilterOrRadioValue() {
				const components = this.$refs.dropItem
				if (!components || !components.length) return
				components.forEach(component => {
					this.result.push(...component.selectFilterArr, ...component.selectRadioArr)
				})
			},

			// 取类型为 列表筛选 的选中值
			getListValue() {
				const components = this.$refs.itemMenu
				if (!components || !components.length) return
				components.forEach(component => {
					this.result.push(...component.selectArr)
				})
			},

			//菜单开关
			togglePage(index) {
				if (this.isToggleing) return;
				this.isToggleing = true;
				setTimeout(() => {
					this.isToggleing = false;
				}, 150)
				if (index == this.current) {
					this.hideMenu(true);
				} else {
					this.showMenu(index)
				}
			},


			// 隐藏菜单
			hideMenu(hideReturn = false, type) {
				if (!this.closeOnClickOverlay && type === 'mask') return
				this.hideMenuLayer(true);
				this.hideMaskLayer();
				this.current = -1;
				if (hideReturn) this.confirmFilter()
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

			// 处理结果并返回
			formatResult() {
				const result = JSON.parse(JSON.stringify(this.result))
				const data = this.removeCustomAttributes(result)
				this.$emit('confirm', data)
			},

			emitColse(flag) {
				if (this.autoStow && flag) {
					this.confirmFilter()
				}
			},

			// 点击改变头部标题
			changeMenuTitle(data, index) {
				if (!data || !data.length) {
					return this.menu[index].checkedName = []
				}
				this.menu[index].checkedName = []
				data.forEach((el) => {
					this.menu[index].checkedName.push(el.name)
				})
			},

			// 移除组件自定义属性
			removeCustomAttributes(data) {
				if (!data || !data.length) return []
				data.forEach(el => {
					delete el.drop_item_key;
					delete el.pKey;
					delete el.drop_item_identity;
					delete el.parent_type;
					!this.isChild && delete el[this.childName];
					if (this.isChild && el[this.childName]) {
						this.removeCustomAttributes(el[this.childName])
					}
				})
				return data
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
		height: 88rpx;
		border-bottom: solid 2rpx #eee;
		z-index: 12;
		background-color: #ffffff;
		flex-direction: row;

		.first-menu {
			width: 100%;
			font-size: 26rpx;
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
					transform: translate(210%, -50%) rotateZ(-40deg);
				}
			}

			.name {
				position: relative;
				height: 40rpx;
				text-align: center;
				text-overflow: clip;

				// overflow: hidden;
				&::after,
				&::before {
					content: "";
					position: absolute;
					right: 0;
					top: 50%;
					width: 10rpx;
					height: 5rpx;
					background-color: #757575;
					transition: transform 0.3s linear;
				}

				&::before {
					transform: translate(150%, -50%) rotateZ(-40deg);
				}

				&::after {
					transform: translate(210%, -50%) rotateZ(40deg);
				}
			}

			.txt {
				display: inline-block;
				max-width: 80rpx;
				overflow: hidden;
				white-space: nowrap;
			}

			.checked-color {
				color: $uni-color-primary;
			}
		}
	}

	.show-shadow {
		box-shadow: 0 4rpx 20rpx rgba(#ccc, 1);
	}

	.sub-menu-class {
		width: 100%;
		position: absolute;
		left: 0;
		transform: translate3d(0, - 100%, 0);
		max-height: 690rpx;
		background-color: #ffffff;
		z-index: 11;

		overflow: hidden;
		align-items: center;
		flex-direction: column;
		transition: transform .15s linear;

		&.hide {
			display: none;
		}

		&.show {
			transform: translate3d(0, calc(88rpx + 1rpx), 0);
		}

		.btn-box {
			flex-shrink: 0;
			width: 698rpx;
			height: 120rpx;
			flex-direction: row !important;
			align-items: center;
			justify-content: space-between;

			>view {
				width: 320rpx;
				height: 80rpx;
				border-radius: 80rpx;
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
		height: 600rpx;
		display: flex;
		flex-direction: column;
		justify-content: space-between;
		align-items: center;

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

drop-down --下拉筛选菜单，多端测试通过
===========

## 参数
可选参数属性列表

|参数名|说明|类型|是否必填|默认值|
|----|----|----|----|----|
|filterData|筛选列表|Array|是|[]|
|childName|子级菜单字段名|String|否|submenu|
|fileds|显示字段|String|否|name|
|isChild|返回结果时是否一并返回子菜单|Boolean|否|false|
|autoStow|选择完成是否自动收起菜单，仅列表模式有效|Boolean|否|true|
|resetStow|重置参数后自动收起菜单|Boolean|否|false|
|areaClass|textarea类名|String|否|空|
|confirm|菜单收起时返回赛选结果|Function|否|[]|

## filterData 格式和默认值设置
在 filterData 筛选列表中为需要选中的项添加 checked: true 即可，checked 不存或 checked：false 则不选中
```js
const filterData = [{
	name: '价格',
	type: 'radio',
	submenu: [{
		name: '200-300',
	},{
		name: '500-600',
		checked: true //默认选中
	}]
}]
```

## 使用

>[从uniapp插件市场导入](https://ext.dcloud.net.cn/plugin?name=drop-down)

```html
<template>
	<view class="index">
		<drop-down :filterData="filterData" @confirm="confirm"></drop-down>
	</view>
</template>
```

```js
<script>
	import data from '@/common/data.js'; //筛选菜单数据
	export default {
		data() {
			return {
				filterData: [],
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
```

## 描述
组件内部使用scss进行书写样式，主题色是使用根目录下的uni.scss中的 $uni-color-primary，使用HBuildX创建uniapp项目之后根目录下会自动创建uni.scss文件，如需修改主题色可以使用快捷键 Ctrl+F 输入 $uni-color-primary 并选择 本插件目录名称 drop-dowm 全部替换即可。

## 参考

参考 [HM-filterDropdown 下拉式筛选菜单](https://ext.dcloud.net.cn/plugin?id=1078) 

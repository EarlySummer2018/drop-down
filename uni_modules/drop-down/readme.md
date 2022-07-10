
## drop-down --下拉筛选菜单，多平台测试通过，不支持的平台暂未测试


[![OSCS Status](https://www.oscs1024.com/platform/badge/EarlySummer2018/drop-down.svg?size=small)](https://www.oscs1024.com/project/EarlySummer2018/drop-down?ref=badge_small)

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
				// 返回值为一个数组
				console.log('eeee', e);
			},
		},
	}
</script>
```

## 描述
组件内部使用scss进行书写样式，主题色是使用根目录下的uni.scss中的 $uni-color-primary，使用HBuildX创建uniapp项目之后根目录下会自动创建uni.scss文件，如需修改主题色可以使用快捷键 Ctrl+F 输入 $uni-color-primary 并选择 本插件目录名称 drop-dowm 全部替换即可。

## 参数
可选参数属性列表

|参数名				|说明																																																						|类型	|是否必填	|默认值	|可选值	|
|----				|----																																																						|----	|----		|----	|----	|
|filterData			|筛选列表																																																					|Array	|是			|[]		|-		|
|childName			|子级菜单字段名																																																				|String	|否			|submenu|-		|
|fileds				|显示字段																																																					|String	|否			|name	|-		|
|isChild			|返回结果时是否一并返回子菜单																																																|Boolean|否			|false	|true	|
|autoStow			|菜单类型为 hierarchy 或 hierarchy-column 时选择完成之后是否自动收起菜单并返回选中值，默认 true，选择所有子  菜单后再收起，如果有多级时，希望选择一级或二级菜单时就收起，需要设置为 false，点击确定收起						|Boolean|否			|true	|false	|
|resetStow			|重置参数后自动收起菜单																																																		|Boolean|否			|false	|true	|
|overlay			|是否显示遮罩																																																				|Boolean|否			|true	|false	|
|closeOnClickOverlay|点击遮罩是否收起菜单																																																		|Boolean|否			|true	|false	|
|shadow				|是否显示菜单下阴影																																																			|Boolean|否			|true	|false	|


# Event 事件
|事件名	|说明						|类型	|回调参数	|
|----	|----						|----	|----		|
|confirm|菜单收起时返回的筛选结果	|emit	|array		|
## Slot 插槽

|名称	|说明									|参数	|
|----	|----									|----	|
|header	|自定义顶部内容，如，在顶部添加搜索栏	|无		|
|title	|自定义选中显示样式，如，修改显示顺序	|title	|


## OSCS
[![OSCS Status](https://www.oscs1024.com/platform/badge/EarlySummer2018/drop-down.svg?size=large)](https://www.oscs1024.com/project/EarlySummer2018/drop-down?ref=badge_large)

## 参考

参考 [HM-filterDropdown 下拉式筛选菜单](https://ext.dcloud.net.cn/plugin?id=1078) 

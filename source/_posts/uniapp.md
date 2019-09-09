---
title: uniapp的使用
date: 2018-07-12 11:43:08
tags: [前端]
---
##### 在app内通过浏览器打开网址

```
plus.runtime.openURL("https://github.com/F-loat/mpvue-echarts");
```

##### 左滑删除(不能设置在flex-direction: column;的子元素中)

```javascript
<template>
	<view style="flex-direction: column;">
		<view style="width: 750upx;">
			<view @touchstart="touchstart" @touchend="touchend" @tap="click" :class="move">	
			<view style="width: 750upx;height: 100upx;background-color: #007AFF;"></view>
			<view style="width: 200upx;height: 100upx;background-color: #FF0000;" v-show="show"></view>
			</view>
		</view>

	</view>

</template>

<script>
	export default {
		data(){
			return{
				show:false,
				sx:0,
				es:0,
				move:""
			}
		},
		methods:{
			touchend(e){
				console.log(e.mp.changedTouches[0].clientX)
				this.ex=e.mp.changedTouches[0].clientX
				if(this.ex-this.sx>30){
					this.move="move_before"
					this.show=false
					this.ex=0
					this.sx=0
				}
				if(this.ex-this.sx<-30){
					this.move="move_after"
					this.show=true
					this.ex=0
					this.sx=0
				}
			},
			touchstart(e){
				console.log(e.mp.changedTouches[0].clientX)
				this.sx=e.mp.changedTouches[0].clientX	
				
			},
			click(){
				console.log("点击")
			}
		}
	}
</script>

<style>
	.move_before{
		transform:translateX(0)
	}
	.move_after{
		transform:translateX(-200upx)
	}
</style>
```

##### 循环操作当前的item

```
:class="move==index?'move_left':'move_right'
```



##### css样式

```
1.设置元素的旋转:transform:rotate(45deg);
  transform 属性向元素应用 2D 或 3D 转换。该属性允许我们对元素进行旋转、缩放、移动或倾斜。
  元素移动:transform:translateX(-200upx)
2.设置背景占满全屏:min-height:100vh;
3.      position: fixed; //固定定位,设置上下左右为0.铺满全屏
		z-index: 998;//层级,数值越大,显示在最上面
		top: 0;
		right: 0;
		bottom: 0;
		left: 0;
		background-color: rgba(0, 0, 0, .3);//混合颜色,最后一位可视度
4.
background-size:auto;/* 默认值，不改变背景图片的高度和宽度 */
background-size:100px 50px;/* 第一个值为宽，第二个值为高，当设置一个值时，将其作为图片宽度来等比缩放 */
background-size:10%;/* 0％~100％之间的任何值，将背景图片宽高按百分比显示，当设置一个值的时候同也将其作为图片宽度来等比缩放 */
background-size:cover;/* 将背景图片等比缩放填满整个容器 */
background-size:contain;/* 将背景图片等比缩放至某一边紧贴容器边缘 */
5.height: calc(100% - 100upx);计算高度
6.box-sizing: border-box;写了这个padding不会改变组件的宽高
7.overflow:hidden;超过容器隐藏,overflow:scroll;超过容器滚动
```

事件

```
检测物理返回键:onBackPress
```

##### vuex的使用

store/index.js

```
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

const store = new Vuex.Store({
    state: {
        user:{
			username:"",
			nickname:"",
			phone:"",
			id:"",
		},
		token:""
    },
    mutations: {
		setUser(state,user){
			state.user.username=user.username
			state.user.nickname=user.nickname
			state.user.phone=user.phone
			state.user.id=user.id			
		},
		logout(state){
			state.token=""
		},
		setToken(state,token){
			state.token=token
		}
        
    }
})

export default store
```

使用

main.js

```
import store from './store'
Vue.prototype.$store = store
```

##### 使用scroll-view注意

```
当设置为x轴滑动

scroll-view的样式设置为:white-space: nowrap;规定段落中的文本不进行换行：

子view的样式设置为:display: inline-block;不独占一行的块级元素
```

##### 渐变

```
background: linear-gradient(direction, color-stop1, color-stop2, ...);
```

##### 使用scroll-view,Y轴滚动

```
主页:

.page {
		flex-direction: column;
		height: 100vh;
		width: 750upx;
		background-color: #F1F1F3;
		position: fixed;
		top: 0;
		left: 0;
	}

scroll-view,需放到一个view中,不单独使用,

<view style="width: 750upx;flex:1 ;padding-bottom: 100upx;">
	<scroll-view style="width: 750upx;height: 100%" scroll-y>
```

##### 键盘弹起来压缩输入框

```
flex-shrink:0;
```

##### 实现弹框的动画



```
隐藏前{
transition: all 0.3s ease;
transform: translateY(100%);
}
显示{
transform: translateY(0);
}
```



##### 实现返回列表页面

在收货地址的时候.编辑后返回地址列表,应使用返回,而不是用挑转

```
uni.navigateBack({
				delta:1
				})
```

##### 文字超出省略号

单行

```
display: block;
overflow: hidden;
text-overflow: ellipsis;
white-space: nowrap;
```

多行

```
display: -webkit-box;
-webkit-box-orient: vertical;
-webkit-line-clamp: 3;
overflow: hidden;
```


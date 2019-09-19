---
title: VUE的使用
date: 2018-07-12 11:43:08
tags: 
categories: 前端
---
#### 1.创建项目:

```
npm install --global vue-cli
```

​     

```
vue init webpack my-project
```


修改三处:

```
将config文件夹里面的index.js中的assetsPublicPath的值修改 为“./”，
webpack.prod.conf.js 中output添加参数publicPath:’./’ 
在webpack.base.conf.js里 publicPath: process.env.NODE_ENV === ‘production’ ? ‘./’ +config.build.assetsPublicPath : ‘./’ + config.dev.assetsPublicPath
```

```
//    ...(config.dev.useEslint ? [createLintingRule()] : []),
将下划线处注释掉，这样关闭代码规范验证就可以了，否则在vue-cli中会导致无法运行项目
```

#### 2.组件引用

```vue
<template>
  <div>
  <head-vue></head-vue>
  <bodyVue></bodyVue>
  <footVue></footVue>
  </div>
</template>

<script>
//导入组件
import headVue from './head.vue'
import bodyVue from './body.vue'
import footVue from './foot.vue'
export default {
  name:'HelloWorld',
  data () {
    return {
     
    }
  },
  //声明组件
  components:{
  headVue:headVue,
  bodyVue,
  footVue
  }
}
</script>


<style scoped>

</style>
```

#### 3.全局组件的使用

在main.js中

```vue
import headVue from './head.vue'
import bodyVue from './body.vue'
import footVue from './foot.vue'



Vue.component('headVue',headVue)
Vue.component('bodyVue',bodyVue)
Vue.component('footVue',footVue)

```

#### 4.组件传值

##### 父组件向子组件传值

在子组件中写

props:['传入的变量'],

props: {
​			disabled: {
​				type: Boolean,
​				default: false
​			}
​		},

在父组件中

<head-vue text="值"></head-vue>

或者

<head-vue :text="value"></head-vue>

在

data(){

return{

value:'值' }

}

##### 子组件向父组件传值

在子组件上的事件方法上使用this.$emit("名","值"):

```
this.$emit('send-message', {
						type: 'text',
						content: that.inputValue
					});
```

在父组件使用@send-message="getMessage",

```
getMessage(data){ var content = data.content }
```

##### 子组件父组件双向传值



在子组件

```
props: {
			value: {
				type: Boolean,
				default: false
			}
		},

watch: {
			value(newVal) {
				this.disabled = newVal;
			}
		},



panelShow() {
				this.disabled = false
				this.$emit('input', false);
			},
```

父组件

```
<dia-log v-model="disable" @getKuaidi="getKuaidi($event)"></dia-log>
```

#### v-model解释

v-model等同于 在子组件 prop:['value'],以及发送一个方法 this.$emit('input', newValue)



#### 5.路由:

    - 1:下载 `npm i vue-router -S`
    - 2:在main.js中引入 `import VueRouter from 'vue-router';`
    - 3:安装插件 `Vue.use(VueRouter);`
    - 4:创建路由对象并配置路由规则
        + `let router = new VueRouter({ routes:[ {path:'/home',component:Home}  ]   });`
    - 5:将其路由对象传递给Vue的实例，options中
        + options中加入 `router:router`
    - 6:在app.vue中留坑 ` <router-view></router-view>
```
 <router-link to="/beijing">去北京</router-link>
  <router-link :to="{name:'bj'}">去北京</router-link>
```

##### 路由带值:在create(){ 获取路由中的值 }

* 查询字符串
    - 1:去哪里 `<router-link :to="{name:'detail',query:{id:1}  } ">xxx</router-link>`
    - 2:导航(查询字符串path不用改) `{ name:'detail' , path:'/detail',组件}`
    - 3:去了干嘛,获取路由参数(要注意是query还是params和对应id名)
        + `this.$route.query.id`
* path方式
    - 1:去哪里 `<router-link :to="{name:'detail',params:{name:1}  } ">xxx</router-link>`
    - 2:导航(path方式需要在路由规则上加上/:xxx) 
    - `{ name:'detail' , path:'/detail/:name',组件}`可以在后面加多个值`{ name:'detail' , path:'/detail/:name/:id/:pass',组件}`
    - 3:去了干嘛,获取路由参数(要注意是query还是params和对应name名)
        + `this.$route.params.name`

##### 路由跳转

```
this.$router.go(1) 根据浏览器记录 前进1 后退-1

this.$router.push(/home)直接跳转到某个页面显示
```

##### 命名试图

在一个页面有多个router-view时,

```
<router-view name="a"></router-view>
<router-view name="b"></router-view>
```

在路由匹配中

```
var Foo = { template: '<div>foo</div>' }
var Bar = { template: '<div>bar</div>' }
var routes = [
        {
            path:"/one",
            name:"one",
            components:{
                a:Foo,
                b:Bar
            }
        },
    ]
```

##### 重定向和404

* 进入后，默认就是/
* 重定向 `{ path:'/' ,redirect:'/home'  }`
* 重定向 `{ path:'/' ,redirect:{name:'home'}  }`
* 404 : 在路由规则的最后的一个规则
    - 写一个很强大的匹配
    - `{ path:'*' , component:notFoundVue}



#### 6.执行子组件事件

在父组件

```
<child ref="childRef"></child>
this.$ref.childRef.childEvent();
```

在子组件

```
methods: {
    childEvent(){
        // xxxx
    }

}
```

#### 使用富文本

```
npm install @tinymce/tinymce-vue -S
```

```
npm install tinymce -S 
```

tinymce提供了很多的语言包，这里我们下载[中文语言包](<https://www.tiny.cloud/get-tiny/language-packages/>)

1、在`public`目录下新建`tinymce`，将上面下载的语言包解压到该目录
2、在`node_modules`里面找到`tinymce`,将`skins`目录复制到`public/tinymce`里面

最终形成以下目录形式：
![img](https://img2018.cnblogs.com/blog/1249006/201904/1249006-20190415142353815-177141399.jpg)

编写 tinymce-editor.vue 组件

```javascript
<template>
  <div class="tinymce-editor">
    <editor v-model="myValue"
      :init="init"
      :disabled="disabled"
      @onClick="onClick">
    </editor>
  </div>
</template>
<script>
import tinymce from 'tinymce/tinymce'
import Editor from '@tinymce/tinymce-vue'
import 'tinymce/themes/silver'
// 编辑器插件plugins
// 更多插件参考：https://www.tiny.cloud/docs/plugins/
import 'tinymce/plugins/image'// 插入上传图片插件
import 'tinymce/plugins/media'// 插入视频插件
import 'tinymce/plugins/table'// 插入表格插件
import 'tinymce/plugins/lists'// 列表插件
import 'tinymce/plugins/wordcount'// 字数统计插件
export default {
  components: {
    Editor
  },
  props: {
    value: {
      type: String,
      default: ''
    },
    disabled: {
      type: Boolean,
      default: false
    },
    plugins: {
      type: [String, Array],
      default: 'lists image media table wordcount'
    },
    toolbar: {
      type: [String, Array],
      default: 'undo redo |  formatselect | bold italic forecolor backcolor | alignleft aligncenter alignright alignjustify | bullist numlist outdent indent | lists image media table | removeformat'
    }
  },
  data () {
    return {
      init: {
        language_url: '/tinymce/langs/zh_CN.js',
        language: 'zh_CN',
        skin_url: '/tinymce/skins/ui/oxide',
        // skin_url: '/tinymce/skins/ui/oxide-dark',//暗色系
        height: 300,
        plugins: this.plugins,
        toolbar: this.toolbar,
        branding: false,
        menubar: false,
        // 此处为图片上传处理函数，这个直接用了base64的图片形式上传图片，
        // 如需ajax上传可参考https://www.tiny.cloud/docs/configure/file-image-upload/#images_upload_handler
        images_upload_handler: (blobInfo, success, failure) => {
          const img = 'data:image/jpeg;base64,' + blobInfo.base64()
          success(img)
        }
      },
      myValue: this.value
    }
  },
  mounted () {
    tinymce.init({})
  },
  methods: {
    // 添加相关的事件，可用的事件参照文档=> https://github.com/tinymce/tinymce-vue => All available events
    // 需要什么事件可以自己增加
    onClick (e) {
      this.$emit('onClick', e, tinymce)
    },
    // 可以添加一些自己的自定义事件，如清空内容
    clear () {
      this.myValue = ''
    }
  },
  watch: {
    value (newValue) {
      this.myValue = newValue
    },
    myValue (newValue) {
      this.$emit('input', newValue)
    }
  }
}

</script>
```

使用

```
import TinymceEditor from '@/components/tinymce'

components: {

 TinymceEditor

  },

<TinymceEditor v-model="msg"></TinymceEditor>
```



#### 使用element-ui

```
1. 安装 loader 模块：
    cnpm install style-loader -D
    cnpm install css-loader -D
    cnpm install file-loader -D
2. 安装 Element-UI 模块
   cnpm install element-ui --save 
   
3. 修改 webpack.base.conf.js 的配置，位置：如下图：
     {
        test: /\\\\\\\\.css$/,
        loader: "style!css"
    },
    {
        test: /\\\\\\\\.(eot|woff|woff2|ttf)([\\\\\\\\?]?.*)$/,
        loader: "file"
    } 
  4.打开项目：src/main.js,添加下面三条
   import ElementUI from 'element-ui'
   import 'element-ui/lib/theme-chalk/index.css'
   Vue.use(ElementUI)
```

在使用上传文件时,在钩子函数需要传另外的参数

```
:on-success="(response, file, fileList)=>uploadSuccessPic(response, file, fileList,index,index2)"

on-remove="(file, fileList)=>removePic(file, fileList,index,index2)"
```



#### 使用axios

```html
 npm install axios --save-dev
```

```html
import axios from 'axios'
```

```html
Vue.prototype.$http=axios
```

在组件中使用:

```

methods: {
      get(){
        this.$http({
          method:'get',
          url:'/url',
          data:{}
        }).then(function(res){
          console.log(res)
        }).catch(function(err){
          console.log(err)
        })
        
        this.$http.get('/url').then(function(res){
          console.log(res)
        }).catch(function(err){
          console.log(err)
        })
      }     
}
```

#### Vue2.0做的项目在IE下面打开一片空白解决办法

https://blog.csdn.net/lyn1772671980/article/details/80690490

#### VUE有默认边距

```
*{
    padding: 0;
    margin: 0;
  }
```

#### 函数calc() ,用于计算长度

```
height:calc(100vh - 100px);
```

#### 监听器

```
 watch: {
    // 如果 `question` 发生改变，这个函数就会运行
    question: function (newQuestion, oldQuestion) {
      this.answer = 'Waiting for you to stop typing...'
      this.debouncedGetAnswer()
    }
  },
```

#### 修改VUE的公共样式

```css
body, div, dl, dt, dd, ul, ol, li, h1, h2, h3, h4, h5, h6, pre, form, fieldset, legend, input, button, textarea, p, th, td {
  margin: 0;
  padding: 0;
  font-size: 12px;
}

img {
  border: none;
}

h1, h2, h3, h4, h5, h6 {
  font-size: 14px;
}

input, button, textarea, select, optgroup, option {
  font-family: inherit;
  font-size: inherit;
  font-style: inherit;
  font-weight: inherit;
}

input, button, textarea, select {
  *font-size: 100%;
}

ol, ul {
  list-style: none;
}

:link, :visited {
  text-decoration: none;
}

.clearfix:after {
  content: ".";
  display: block;
  height: 0;
  clear: both;
  visibility: hidden;
}

.clearfix {
  zoom: 1;
}
```

#### 快速实现块级元素上下居中

```css
top: 50%;
left: 50%;
position:absolute;
transform: translate(-50%, -50%);
```

#### 动态为对象的属性赋值

```javascript
var s={
name:"张三"
}
s.age=18
var p='sex'
s[p]='男'
最后s变为:
s{
name :"张三",
age :18,
sex :"男"
}
```

<<<<<<< HEAD
#### 双向绑定的坑

Vue 不能检测到对象属性的添加或删除。由于 Vue 会在初始化实例时对属性执行 getter/setter 转化过程，所以属性必须在 data 对象上存在才能让 Vue 转换它，这样才能让它是响应的。为对象在后面添加的属性不是响应的

#### 超出省略号

单行

```
overflow: hidden;
text-overflow:ellipsis;
white-space: nowrap;
```

多行

```
display: -webkit-box;
-webkit-box-orient: vertical;
-webkit-line-clamp: 3;
overflow: hidden;
```

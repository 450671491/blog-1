---
title: 一个弹窗内容引发的"血案"(下)
date: 2019-06-17
tags:
- Vue
- JavaScript
- Component
---

上回说到鄙人已经通过封装单个组件文件(`.vue`)的方式破解了这桩"血案"。但是却不是最"优美"的解决方式，于是便有了此文。

<!-- more -->

# 一、后续思考

前面说到`$createElement`可以接受一个组件选项对象作为参数，而且通过`import`组件的方式也成功拿到了目标组件的选项对象。那么除了该方法还能不能采用其他更好的解决方案呢？比如不采用封装单个组件文件的方式，而是直接构建一个局部组件获取它的选项对象。

首先介绍`Vue`中我所了解的两种构建组件的方法:

1. `Vue.extend(options)` 使用基础`Vue`构造器，创建一个"子类"。

    结果返回一个组件构造器。具体🌰代码如下:

    ```javascript
    // 获取构造器
    let myCom = Vue.extend({
      template: '<p>乐队的夏天</p>',
      data () {
        return {
          bandArr: ['面孔', '痛仰', '九连真人']
        };
      }
      // ...
    });
    // 实例化
    let vm = new myCom();
    ```

2. `Vue.component(id, [definition])` 注册或获取全局组件。

    结果同样返回一个构造器，🌰如下：

    ```javascript
    // 注册组件
    Vue.component('my-com', {
      template: '<p>乐队的夏天</p>',
      data () {
        return {
          bandArr: ['新裤子', 'Mr Woohoo', '海龟先生']
        };
      }
      // ...
    })
    // 获取注册的组件(返回构造器非实例)
    let myCom = Vue.component('my-com');
    ```

熟悉`Vue`的同学应该知道如果单纯只要得到一个构造器的话使用`Vue.extend`就够了，因为`Vue.component`的第二个参数就是`Vue.extend`的返回值。(会隐式调用`Vue.extend`，如果你传入一个对象的话)。

好，那么到目前思路很清晰了。我们可以通过`Vue.extend`得到目标组件的构造函数，然后通过`new`。。。一个。。。实例？。。。。。

细心的同学会发现我又本末倒置了，我本来就传入了组件的选项对象来生成构造器。所以最终的实现方式是这样的:

```javascript
const h = this.$createElement;
this.$confirm('', '提示', {
  message: h({
    name: 'myCom',
    template: '<p>乐队的夏天真好看</p>'
  }),
  closeOnClickModal: false,
  confirmButtonText: '发送',
  cancelButtonText: '取消'
}).then(() => {
}).catch(() => {
});
```

# 二、真相只有一个

爱折腾的我还是不满意现有的方式，原因有二: 1. 采用字符串模版的方式太死板不够灵活；2. 单纯地想搞事情。

于是我又找啊找，诶，找到了原来除了`template`之外还有`render`且官网说到如果`Vue`选项中若存在`render`函数则不会再通过`template`或`el`指定的挂载元素中提取出的 HTML 模板编译渲染函数。说到这个`render`函数它接受我们之前一直有用到过的方式(`$createElement`)作为参数来创建虚拟节点(`VNode`)。这有回到了故事的开头，写形如这样的:

```javascript
// DOM节点对象树
h('p', null, [
  h('span', null, '内容可以是 '),
  h('i', { style: 'color: teal' }, 'VNode')
])
```

但在这里要补充的是如果你不喜欢这种写法，那你可以写`JSX`呀(是不是很熟悉这段话)。就像在`React`里面那样，没错，是可以的，只要你安装了`babel`插件。

当然这个需求我肯定是不会用`JSX`去改造的啦，有点小题大做了。那么为什么我要引出这个`render`呢？是因为`Vue.compile`可以生成这个`render`函数。它接受的参数就是字符串模板。那么我们这一次的方式变成了这样:

```javascript
// 最终写法
let htmlStr = '<p>乐队的夏天真好看啊</p>';
let comRender = Vue.compile(htmlStr).render;
let staticRender = Vue.compile(htmlStr).staticRenderFns;
const h = this.$createElement;
this.$confirm('', '提示', {
  message: h({
    name: 'myCom',
    render: comRender,
    staticRenderFns: staticRender
  }),
  closeOnClickModal: false,
  confirmButtonText: '发送',
  cancelButtonText: '取消'
}).then(() => {
}).catch(() => {
});
```

其实这里就相当于我们主动调起`Vue`的编译流程，因为我们都知道从字符串模板到真实的`DOM`渲染会经历一个`toFunctions`的过程。`Vue.compile`的作用就是能让我们获取到编译的结果也就是`render functions`。有研究过的同学应该知道返回的结果对象包含两个属性`render`和`staticRenderFns`，前者就是普通的"渲染函数"，至于后者是啥，我们先来看看生成的`render`函数:

```javascript
function anonymous() {
  with(this) {
    return _c('div', [ // 创建一个div元素
      _m(0, false, false), // 静态节点 此处对应 staticRenderFns 数组索引为 0 的 render function
      _v(" "), // 创建空文本
      _c('el-popover', {
      attrs: {
        "placement": "right-start",
        "width": "245",
        "trigger": "hover"
      }
    },
    [_c('div', {
      staticStyle: {
        "font-size": "12px",
        "color": "#909399"
      }
    },
    [_c('div', [_v("绑定指的是家长登录相关H5，输入手机号及验证码后完成绑定")]), _v(" "), _c('div', {
      staticStyle: {
        "text-align": "center"
      }
    },
    [_c('img', {
      staticStyle: {
        "width": "60px"
      },
      attrs: {
        "src": "https://cdn.meishakeji.com/",
        "alt": "绑定示意图"
      }
    })]), _v(" "), _c('img', {
      staticStyle: {
        "width": "225px"
      },
      attrs: {
        "src": "/static/img/bindPic.6c59ab6.png",
        "alt": "绑定示意图"
      }
    })]), _v(" "), _c('span', {
      staticStyle: {
        "font-size": "12px",
        "color": "#909399"
      },
      attrs: {
        "slot": "reference"
      },
      slot: "reference"
    },
    [_c('i', {
      staticClass: "iconfont icon-ic_help",
      staticStyle: {
        "font-size": "14px"
      }
    }), _v(" "), _c('span', [_v("什么是绑定")])])])], 1)
  }
}
```

看过源码的同学对于上面的`_c, _v, _m`应该不陌生。

```javascript
// Vue源码
export function installRenderHelpers (target: any) {
  target._o = markOnce
  target._n = toNumber
  target._s = toString
  target._l = renderList
  target._t = renderSlot
  target._q = looseEqual
  target._i = looseIndexOf
  target._m = renderStatic // 渲染静态节点
  target._f = resolveFilter
  target._k = checkKeyCodes
  target._b = bindObjectProps
  target._v = createTextVNode // 创建虚拟文本节点
  target._e = createEmptyVNode
  target._u = resolveScopedSlots
  target._g = bindObjectListeners
  target._d = bindDynamicKeys
  target._p = prependModifier
}
```

`render`函数中的`_m(0, false, false)`就是调用`staticRenderFns`数组中的第一个函数。这个数组中的函数与"虚拟DOM"中的`diff`算法有关，会在编译阶段给后续不会发生变化的`VNode`打上static的为true的key。目前最新的`Vue`源码会将这类`VNode`加入到一个名为`cached`的数组当中。这也就是为什么在上面的🌰里我要加多一个`staticRenderFns`属性，不然会报`Cannot read property 'cached' of undefined`。

# 三、总结

好，这宗"血案"到这里就完美"结案"了。本来还想着给大家简单介绍下`Vue的组件化`和`Vue的编译`都到底干了些啥。但是写起来发现自己的认知还是那么的浅薄，所以在这就不"班门弄斧"了，下去沉淀一波再来！

***注:作者才疏学浅，如有不当的地方欢迎指出！***
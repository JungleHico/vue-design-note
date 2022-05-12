# 第3章 Vue.js 的设计思路

## 1. 声明式描述 UI

1. 模板语法（**编译时**）
   
   Vue.js 为用户提供模板语法来声明式描述 UI，例如：文本插值、指令、`v-bind` 或 `:` 来动态绑定指令、`v-on` 或 `@` 来绑定事件等，框架内部通过 `Complier` 编译成虚拟 DOM，这种方式对用户来说更加直观，这种方式编写的框架属于**编译时**框架。

2. 渲染函数（**运行时**）
   
   Vue.js 还为用户提供另一种描述 UI 的方式：渲染函数。这种方式通过使用 JavaScript 对象来描述 DOM 结构，也就是虚拟 DOM。使用 JavaScript 对象会比模板语法更加灵活，而且也不需要经过编译，这种方式编写的框架属于**运行时**框架。
   
   模板语法：
   
   ```html
   <div @click="handler">
       <span>text</span>
   </div>
   ```
   
   JavaScript 对象（虚拟 DOM）：
   
   ```js
   const title = {
       // 标签名称
       tag: 'div',
       // 标签属性
       props: {
           onClick: handler
       },
       // 子节点
       children: [
           { tag: 'span', children: 'text' }
       ]
   }
   ```
   
   Vue.js 中，组件的 `redner()` 函数就是用来描述虚拟 DOM 的，除此之外，Vue.js 还提供了 工具函数 `h()` ，用来简化虚拟 DOM 的书写：
   
   ```js
   import { h } from 'vue'
   export default {
       render() {
           return h('div', { onClick: handler }, { tag: 'span', children: 'text' })
       }
   }
   ```

## 2. 渲染器

渲染器的作用是将虚拟 DOM 转化为真实的 DOM 并渲染到页面中，无论我们使用模板语法，还是使用渲染函数编写代码，最终都需要通过渲染器渲染为真实 DOM。

假定我们有以下的虚拟 DOM：

```js
const vnode = {
     // 标签名称
     tag: 'h1',
     // 标签属性
     props: {
         class: 'title',
         onClick: () => alert('标题被点击')
     },
     // 子节点
     children: '标题'
```

为了将上面的虚拟 DOM 转化为真实 DOM，我们需要以下步骤：

1. 创建节点

2. 为节点添加属性和事件

3. 处理子节点

4. 挂载节点

```js
/**
 * @params vnode {VNode} 节点虚拟DOM
 * @params container {HTMLElement} 挂载的节点
 */
function renderer(vnode, container) {
    // 1. 创建节点
    const el = document.createElement(vnode.tag)

    // 2. 为节点添加属性和事件
    for (const key in vnode.props) {
        if (/^on/.test(key)) {
            // 以on开头，说明是事件
            el.addEventListener(
                key.substr(2).toLowerCase(),
                vnode.props[key]
            )
        } else {
            // 添加属性
            el.setAttribute(key, vnode.props[key])
        }
    }

    // 3. 处理子节点
    if (typeof vnode.children === 'string') {
        // 文本
        el.innerText = vnode.children
    } else if (Array.isArray(vnode.children)) {
        // 递归渲染子节点
        vnode.children.forEach(child => {
            renderer(child, el)
        })
    }

    // 4. 挂载节点
    container.appendChild(el)
}


renderer(vnode, document.body)
```

## 3. 编译器

编译器的作用是将模板编译成虚拟 DOM（渲染函数），最终交给渲染器渲染成真实 DOM。

以一个 .vue 文件为例：

```html
<template>
    <div @click="handler">
        click me
    </div>
</template>

<script>
export default {
    data() {
        return {}
    },
    methods: {
        handler() {}
    }
}
</script>
```

`<template>` 标签的内容就是模板内容，编译器会将模板内容编译成渲染函数并添加到组件对象中：

```js
export default {
    data() {
        return {}
    },
    methods: {
        handler() {}
    },
    render() {
        return h('div', { onClick: handler }, 'click me')
    }
}
```

最后，渲染器会将渲染函数返回的虚拟 DOM 转换为真实 DOM。

<div style="width: 100%; display: flex; justify-content: space-between;">
    <a href="https://github.com/JungleHico/vue-design-note/blob/master/docs/第2章%20框架（Vue.js）设计的核心要素.md">←第2章 框架（Vue.js）设计的核心要素</a>
    <span></span>
</div>
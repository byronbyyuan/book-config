---
title: vue 高级小技巧
date: 2020-02-22 22:16:33
tags: vue
categories: 
- 前端
- vue
top_img: https://cn.vuejs.org/images/logo.png
cover: https://cn.vuejs.org/images/logo.png
---

# vue 高级小技巧

学会这几个花里胡哨但小技巧，显得贼气（z）质(b) 😄😄

## hookEvent

---

小例子:
vue监听了滚动事件，一般我们都会定义 beforeDestroy 销毁周期中删除，有什么办法在一个函数完成监听操作呢？

```
export default {
  mounted() {
    // 监听窗口发生变化
    window.addEventListener('resize', this.handleResize)
    // 通过hook监听组件销毁钩子函数，并取消监听事件
    this.$once('hook:beforeDestroy', () => {
      window.removeEventListener('resize', this.handleResize)
    })
  },
  methods: {
    handleResize() {}
  }
}

```

写在一个函数里，可读性就比较好，换同学修改也不至于忘记啦，再也不怕 `code review `😁而且hook 不管只能监听自身组件的钩子哟，有时候咱们用到第三方组件，但是第三方组件确没有类似 change 😣这样的事件，那我们也可以通过hook来解决 😄

```
<template>
    // 绑定update钩子 触发函数操作
    <bk-select-user @hook:update='selectupdate'></bk-select-user>
</template>
export default {
  methods: {
    selectupdate() {}
  }
}
```

## watch

---

```
// 有没有在使用watch时，写过这种写法呢？
export default  {
    watch: {
        xxx () {
            this.getDetail()
        }
    }
    methods: {
        getDetail() {
            // ... 获取接口等等
        }
    }
    mounted () {
        this.getDetail()
    }
}
```

上面这种默认还需要自己去触发一次调用的场景时非常多的，而 `watch` 提供了 `immediate` 的配置项，我们只要将他设置为 `true` 就可以达成上面代码的效果


```
export default  {
    watch: {
        xxx {
            handler: this.getDetail,
            immediate: true // 默认触发一次
            deep： true // 深度监听
        }
    }
    methods: {
        getDetail() {
            // ... 获取接口等等
        }
    }
}

```

watch 还提供了 `deep` 是否深度监听的配置 与 `immediate` 使用一样配置了后 将会进行进行对每个值深度监听，常用于表单项的监控操作

## render

---

render 是 vue 提供的函数式组件

```
export default {
  // 配置functional属性指定组件为函数式组件
  functional: true,
  props: {
    text: {
      type: String
    }
  },
  /**
   * 渲染函数
   * @param {*} h
   * @param {*} context 等于当前组件的上下文 因为函数式组件没有this, props, slots等属性都只能通过context 访问
   */
  render(h, context) {
    const { props } = context
    return `<span>${context.props.text}</span>`
  }
}
```

### 什么情况下使用函数组件?

1. 因为函数式组件不需要实例化，无状态，没有生命周期，所以渲染性能要好于普通组件
2. 函数式组件结构比较简单，代码结构更清晰
3. 不使用 components 时可以作为动态组件展示的另一种方式

### 函数式组件与普通组件的区别?

1. 函数式组件需要在声明组件是指定functional
2. 数式组件不需要实例化，没有this,上下文通过第二个参数访问
3. 函数式组件没有生命周期钩子函数，不能使用计算属性，watch等等
4. 函数式组件不能通过$emit对外暴露事件，调用事件只能通过context.listeners.click的方式调用外部传入的事件因为函数式组件是没有实例化的，所以在外部通过ref去引用组件时，实际引用的是HTMLElement
5. 函数式组件的props可以不用显示声明，所以没有在props里面声明的属性都会被自动隐式解析为prop,而普通组件所有未声明的属性都被解析到$attrs里面，并自动挂载到组件根元素上面(可以通过inheritAttrs属性禁止)

## observable

在 vue 中使用数据状态管理需要引入 vuex 插件，但 vuex 只适用大型项目，就如它自身官网所说 ‘如果您不打算开发大型单页应用，使用 Vuex 可能是繁琐冗余的’, 在 vue **2.6** 版本更新了一个新的 API `VUe.observable`, 小型项目中可以使用他来实现vuex的功能,使用步骤为：

- 与 `vuex` 类似需要创建 `store`

```
import Vue from 'vue'

export const store = Vue.observable = {
    num: 0
}

export const mutations = {
    add () {
        store.num ++
    }
    ...
}

```

- 在组件中使用

```
<template>
  <div @click='add'>
    {{ num }}
  </div>
</template>
<script>
import { store, mutations } from './store'
export default {
  computed: {
    num() {
      return store.num
    }
  },
  methods: {
      add () {
          mutations.add()
      }
  }
}
</script>

```

---

后续内容整理中。。。
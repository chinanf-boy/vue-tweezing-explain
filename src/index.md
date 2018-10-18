## src/index.js

- [可以看看 库的readme先](../zh.md)

### Vue插件vue-tweezing

<details>

<summary> <b>一：Vue.use用例，下拉⬇️</b> </summary>

```js
import {Tweezing, tweezerHelper} from 'vue-tweezing';
// 导入 Tweezer 和 use ,作为 Tweening 引擎
import Tweezer from 'tweezer.js';

// 安装这个插件搭配，你想合作的任何引擎
// 使用 tweezerHelper 生成与Vue搭配的函数
// 因为需要 VueTweezing 控制 tweezing
Vue.use(Tweezing, {
  tweezer: tweezerHelper(Tweezer)
});
```
</details>

---

**二：流程**

- 0. **准备函数**的准备, by `tweezerHelper`函数

- 1. vue插件都需要`install`函数，给`Vue`统筹与指挥
    - 1.1 帮组件的全局变量`tweeners`拿个默认的**准备函数**

> 也就是`Vue.use(Tweezing, {`的时候, 👇下面是正常编写组件

- 2. 我们从用了组件（这个假设）开始，一开始是Vue组件的`props`+`data`+`method`...那一些定义
    - 2.1 `props`
    - 2.2 `data`

- 3. 然是`created`函数的运行，绑定作用域运行函数与用户赋值, 
    - 3.1 `watch.tween`确保`this.tweenFn`成为**准备函数**
    - 3.2 `watch.to` 使用**准备函数**，开始工作

> 期间`this.ensureValue`和`stopTweens`，一个验证兼赋当前值；一个停止`this.$tween`的运作

- 4. 渲染`render`，用的是默认插槽`this.$scopedSlots.default`

<details>

<summary id="4-1-use"> 4.1 组件插槽的使用 </summary>

将它用作组件:

```vue
<Tweezing ref="tweezing" :to="value" duration="500" @end="doSomething">
  <pre slot-scope="tweenedValue">
    target: {{ value }}
    val: {{ tweenedValue }}
  </pre>
</Tweezing>
```
</details>

<details>

<summary id="change-value"> 更改外部的value，达到控制tween的目的 </summary>

更改`value`你通常会这样做:

```js
const vm = new Vue({
  el: '#app',
  data: {
    value: 0
  }
});
// somewhere else
vm.value = 200;
```
</details>


---

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


  - [全局变量tweeners](#%E5%85%A8%E5%B1%80%E5%8F%98%E9%87%8Ftweeners)
  - [2. props + data](#2-props--data)
  - [3. created](#3-created)
  - [4. render](#4-render)
  - [methods](#methods)
  - [watch](#watch)
  - [1. install](#1-install)
  - [stopTweens](#stoptweens)
- [0. helper](#0-helper)
  - [tweezerHelper](#tweezerhelper)
  - [tweenjsHelper](#tweenjshelper)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


#### 全局变量tweeners

用于初始化，vue插件内部的**准备函数**
``` js
// 用于创建tween的函数集合
const tweeners = {}

```

#### 2. props + data
```js
export const Tweezing = {
  props: {
    to: [Number, Object, Array], // 三种类型
    tween: {
      type: [String, Function],
      default: () => 'default',
    },
  },

  data () {
    return {
      value: 0,
    }
  },

```

#### 3. created

组件被创建时，调用

```js
  created () {
    this.$options.watch.tween.call(this, this.tween) // 3.1
    this.value = this.to // 赋予当前值
    this.$options.watch.to.call(this, this.to) // 3.2
  },

```

#### 4. render

组件渲染时，调用

``` js
  render (h) { // <=== h --- createEelement
    const node = this.$scopedSlots.default(this.value) // 插槽的使用，node 就是一个VNode
    if (Array.isArray(node)) {
      // TODO 开发错误
      return node[0]
    }
    return node
  },

```

#### methods

因为作者给予了`this.to`三种类型`[Number, Object, Array]`,所以一直都在验证三种类型

```js
  methods: {
    ensureValue (val) { // 验证 + 赋当前值
      const targetType = typeof val
      const currentType = typeof this.value
      if (targetType === currentType) {
        if (Array.isArray(val)) {
          this.value.length = val.length
        }
        return
      }
      // 将初始值设置为当前值
      if (targetType === 'number') {
        this.value = val
      } else if (Array.isArray(val)) {
        this.value = val.slice()
      } else if (targetType === 'object') {
        this.value = Object.keys(val).reduce((values, name) => {
          values[name] = val[name]
          return values
        }, {})
      }
    },
  },

```

#### watch

用于观察组件的内部变量，若有变化就运行

- `tween` 一般都不怎么变，用来初始化`this.tweenFn`
- `to` ：也就是外部的`value`，内部的**value**由`this.ensureValue(to)`或tween引擎改变

> 这个`to`就有点疑惑了，若是单看代码的话, 请[结合4.1 插槽的使用例子来](#4-1-use)- 其中 - `:to="value"` 「to与value是双向绑定数据」

```js
  watch: {
    tween (tween) {
      if (typeof tween === 'function') {
        this.tweenFn = tween
      } else {
        // TODO警告，如果不是字符串或不存在
        this.tweenFn = tweeners[tween]
      }
    },
    // TODO应该很深
    to (to, old) {
      const type = typeof to
      this.ensureValue(to) // 赋予 to 到 this.value，若验证通过

      stopTweens(this.$tween)
      if (type === 'number') {
        this.$tween = this.tweenFn(this.value, to, {
          $setValue: v => { // tween引擎改变内部 value
            this.value = v
          },
          ...this.$attrs, // 注意这里可以让，除`tween`和`to`之外的props，都被视为`$tween`使用的选项
        }, this)
      } else if (Array.isArray(to)) {
        this.$tween = to.map((value, i) => {
          return this.tweenFn(this.value[i] || 0, value, {
            $setValue: v => {
              this.value.splice(i, 1, v)
            },
            ...this.$attrs,
          })
        })
      } else if (type === 'object') {
        this.$tween = Object.keys(to).reduce((tweens, name) => {
          tweens[name] = this.tweenFn(this.value[name], to[name], {
            $setValue: v => {
              this.value[name] = v
            },
            ...this.$attrs,
          })
          return tweens
        }, Object.create(null))
      }
    },
  },

```

#### 1. install

```js
  // 用作插件
  install (Vue, options = {}) {
    Vue.component(options.name || 'Tweezing', Tweezing)
    const defaultTweener = options.default
    // 本来可以使用{name，default，... options}但我真的不喜欢
    // bubel polyfills它的方式（即使在es）
    delete options.name
    delete options.default
    const tweenerNames = Object.keys(options)
    tweenerNames.forEach(tweener => {
      tweeners[tweener] = options[tweener]
    })
    // 使用第一个或用户提供的任何人   1.1
    tweeners.default = tweeners[defaultTweener || tweenerNames[0]]
  },
}

```

#### stopTweens

停止

```js
function stopTweens (tweens) {
  if (tweens) {
    // TODO使用更好的方法
    // 使用对象时原型为null
    if (Array.isArray(tweens)) {
      tweens.forEach(tween => tween.stop())
    } else if (!Object.getPrototypeOf(tweens)) {
      // TODO并非所有tween都有停止方法
      for (const key in tweens) tweens[key].stop()
    } else {
      tweens.stop()
    }
  }
}


```

### 0. helper

帮助本组件，统一各个引擎的接口与选项与生命周期，并返回一个**准备函数**

> **准备函数**的调用，才真的开始运行各个引擎

#### tweezerHelper

```js
// tweezer.js的助手
export function tweezerHelper (Tweezer) {
  return function (start, end, opts) { // 这里的opts，有一部分`...this.$attrs`
    // 取消之前的tween
    let started
    return new Tweezer({
      start,
      end,
      ...opts,
    }).on('tick', value => {
      if (!started) {
        started = true
        this.$emit('start')
      }
      opts.$setValue(value)
    }).on('done', () => this.$emit('end'))
      .begin()
  }
}
```

- `this.$emit('start')`+`this.$emit('end')`

组件的订阅当作tween引擎的生命周期，如`<Tweezing //... @end="doSomething">`


#### tweenjsHelper

```js
export function tweenjsHelper (TWEEN) {
  return function (value, end, opts) { // 这里的opts，有一部分`...this.$attrs`
    const container = { value }
    // 取消之前的tween
    return new TWEEN.Tween(container)
      .to({ value: end }, opts.duration)
      .interpolation(opts.interpolation || TWEEN.Interpolation.Linear)
      .easing(opts.easing || TWEEN.Easing.Quadratic.Out)
      .delay(+opts.delay || 0)
      // TODO也应该发出该物业的名称
      // 如果只提供一个值，则default可以是名称
      .onStart(() => this.$emit('start'))
      .onUpdate(() => {
        opts.$setValue(container.value)
      })
      .onComplete(() => this.$emit('end'))
      .start()
  }
}
```

- `this.$emit('start')`+`this.$emit('end')`

组件的订阅当作tween引擎的生命周期，如`<Tweezing //... @end="doSomething">`


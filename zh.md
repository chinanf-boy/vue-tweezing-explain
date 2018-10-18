
# VueTweezing [![Build Status](https://badgen.net/circleci/github/posva/vue-tweezing)](https://circleci.com/gh/posva/vue-tweezing) [![npm package](https://badgen.net/npm/v/vue-tweezing)](https://www.npmjs.com/package/vue-tweezing) [![coverage](https://badgen.net/codecov/c/github/posva/vue-tweezing)](https://codecov.io/github/posva/vue-tweezing) [![thanks](https://badgen.net/badge/thanks/♥/pink)](https://github.com/posva/thanks)

> 简单,可自定义和自动tweening,在范围内的插槽(scoped slots)中提供了很好的服务

VueTweezing 可与任何tweening引擎配合使用,并可轻松与某些引擎像[tween.js](https://github.com/tweenjs/tween.js)和[Tweezer](https://github.com/jaxgeller/tweezer.js/)集成

[演示](https://state-animations-amsterdam.surge.sh/polygon),[资源](https://github.com/posva/state-animation-demos/blob/master/pages/polygon.vue)

## 用法

将其安装为插件:

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

将它用作组件:

```vue
<Tweezing ref="tweezing" :to="value" duration="500" @end="doSomething">
  <pre slot-scope="tweenedValue">
    target: {{ value }}
    val: {{ tweenedValue }}
  </pre>
</Tweezing>
```

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

您可以通过访问属性来播放tween对象`$吐温`在里面`渐变`零件:

```js
// given the example above
vm.$refs.tweezing.$tween.stop();
```

## 传递tweening选项

任何传递给`Tweezing`，除`tween`和`to`之外的props，都被视为`$tween`使用的选项

## 支持的Tweeing引擎

WIP

### Tweezer

### Tween.js

### 添加自己的

WIP

您可以查看示例`src/index.js`，看看如何创建自己的助手.

## 执照

[MIT](http://opensource.org/licenses/MIT)

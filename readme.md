# posva/vue-tweezing [![explain]][source] [![translate-svg]][translate-list]

<!-- [![size-img]][size] -->

[explain]: http://llever.com/explain.svg
[source]: https://github.com/chinanf-boy/Source-Explain
[translate-svg]: http://llever.com/translate.svg
[translate-list]: https://github.com/chinanf-boy/chinese-translate-list
[size-img]: https://packagephobia.now.sh/badge?p=Name
[size]: https://packagephobia.now.sh/result?p=Name

「 VueTweezing 可与任何tweening引擎配合使用,并可轻松与某些引擎像[tween.js](https://github.com/tweenjs/tween.js)和[Tweezer](https://github.com/jaxgeller/tweezer.js/)集成 」

---

## explain 🀄️

<!-- doc-templite START generated -->
<!-- time = '2018-10-17' -->
<!-- name = 'posva' -->
<!-- repo = 'vue-tweezing' -->
<!-- commit = '5e8a0256bd910a58e21113494eeb4e4c653c1a17' -->

| 版本     | 与日期        | 最新更新   | 更多               |
| -------- | ------------- | ---------- | ------------------ |
| [commit] | ⏰ 2018-10-17 | ![version] | [源码解释][source] |

[commit]: https://github.com/posva/vue-tweezing/tree/5e8a0256bd910a58e21113494eeb4e4c653c1a17
[version]: https://img.shields.io/npm/v/vue-tweezing.svg

<!-- doc-templite END generated -->

### 中文

[readme](zh.md)


## 生活

[help me live , live need money 💰](https://github.com/chinanf-boy/live-need-money)

---

## package.json

``` json
  "main": "dist/vue-tweezing.cjs.js",
  "scripts": {
    "unit": "jest",
    "dev": "npm run unit -- --watchAll",
    "test": "npm run lint && npm run unit",
    "prepublishOnly": "rollit"
  },
```

> `rollit`是作者自己配置的集成`rollup`的打包工具，零配置

- `rollit` **src/index.js** =生成=> **dist/vue-tweezing.cjs.js**

## src/index.js

- [x] [explain](src/index.md)

## test

使用`jest`作为测试框架

> 其实，我主要想看看vue组件的单元测试

- [ ] [explain](test/)
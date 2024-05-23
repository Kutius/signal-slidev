---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://cover.sli.dev
# some information about your slides, markdown enabled
title: 浅聊Signal
# apply any unocss classes to the current slide
class: text-center
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# https://sli.dev/guide/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/guide/syntax#mdc-syntax
mdc: true
lineNumbers: true
fonts:
  mono: Input Mono
---

# 浅聊 Signal

从 `SolidJS` 中理解 `Signals`

<author>wzz</author>

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---
name: SolidJS
layout: center
---

<div class="grid grid-cols-[3fr_2fr] gap-4">
  <div class="text-center pb-4">
    <img class="h-50 inline-block" src="/solid.svg">
    <div class="op-80 mb-2">
      SolidJS · 反应式 JavaScript 库
    </div>
    <div class="text-center">
      <a class="!border-none" href="https://www.npmjs.com/package/solid-js" target="__blank"><img class="h-4 inline mx-0.5" src="https://img.shields.io/npm/v/solid-js?color=a1b858&label=" alt="NPM version"></a>
      <a class="!border-none" href="https://www.npmjs.com/package/solid-js" target="__blank"><img class="h-4 inline mx-0.5" alt="NPM Downloads" src="https://img.shields.io/npm/dm/solid-js?color=50a36f&label="></a>
      <br>
      <a class="!border-none" href="https://github.com/solidjs/solid" target="__blank"><img class="mt-2 h-4 inline mx-0.5" alt="GitHub stars" src="https://img.shields.io/github/stars/solidjs/solid?style=social"></a>
    </div>
  </div>
  <div class="border-l border-gray-400 border-opacity-25 !all:leading-12 !all:list-none my-auto">

  - 细粒度更新真实 `DOM`
  - 声明式数据
  - Render-once Mental Model
  - 自动依赖收集
  - 足够简单 极易上手

  </div>
</div>

---
layout: iframe
url: https://www.solidjs.com/
scale: 0.6
name: Solid Homepage
---

---
transition: slide-up
---

# 状态更新机制

仅触发一次渲染函数

一段 `JSX` 代码中，有多个变量被调用时，不会重新执行整段 `JSX` 代码，而是仅更新变量部分

<v-click>

````md magic-move
```jsx
const App = ({ value1, value2}) => (
  <div>
    value1: {console.log("value1", value1)}
    value2: {console.log("value2", value2)}
  </div>
)
```

```jsx {*|4-5}
// 当 value1 更新时，只会触发 value1 的更新
const App = ({ value1, value2}) => (
  <div>
    {/* Print: value1 */}
    value1: {console.log("value1", value1)}
    {/* No print */}
    value2: {console.log("value2", value2)}
  </div>
)
```
````

</v-click>

<br />
<br />
<v-click>

## 为什么 `SolidJS` 可以做到这一点？

</v-click>

---
layout: fact
---

# 面向状态驱动

---
transition: slide-up
---

# 响应式

以 `Signals` 方式的心智去理解

<div class="grid grid-cols-2 gap-4">
<div>
<h2>Solid</h2>

````md magic-move
```jsx
const [count, setCount] = createSignal(0);

setCount(count() + 1);
```

```jsx
import { createSignal } from "solid-js";

function Counter() {
  const [count, setCount] = createSignal(0);

  setInterval(() => setCount(count() + 1), 1000);

  return (
    <div>
      <p>Count: {count()}</p>
    </div>
  );
}
```

```jsx
import { createSignal } from "solid-js";

const [count, setCount] = createSignal(0);

function Counter() {
  // It still works.
  setInterval(() => setCount(count() + 1), 1000);

  return (
    <div>
      <p>Count: {count()}</p>
    </div>
  );
}
```
````

</div>
<div >
<h2>React</h2>

```jsx
const [count, setCount] = useState(0);

setCount(count + 1);
```

</div>
</div>

<br />
<v-click>
<p text-teal>纯粹的数据驱动，无论数据在哪定义，都可以建立绑定</p>
</v-click>

---

# 派生状态追踪

```jsx twoslash
const App = () => {
  const [count, setCount] = createSignal(0);
  const doubleCount = () => count() * 2;

  return (
    <button onClick={() => setCount(prev => prev + 1)}>
      {doubleCount()}
    </button>
  );
};
``` 

<br />
<br />
<v-clicks>

- `SolidJS` 收集所有用到了 `count()` 的依赖
- `doubleCount()` 用到了它
- 渲染函数用到了 `doubleCount()`
- 完成整条链路的依赖收集

</v-clicks>

---

# Effects

状态监听

```jsx
const App = () => {
  const [count, setCount] = createSignal(0);

  createEffect(() => {
    console.log(count()); // 在 count 更新时触发
  });
};
```

<br />
<v-clicks>

- 无 deps 声明
- 将监听与生命周期分开

</v-clicks>

---
layout: intro
---

# Why?

<v-click>

<p><span v-mark.red="3"> Signals </span></p>

</v-click>

---

# Hooks

<v-clicks>

- 不受顺序/条件分支影响
  - 逻辑在回调函数内部处理，`createSignal` 本身仅创建一个变量，`createEffect` 本身仅创建一个监听
  - 视图的绑定通过依赖收集完成
- 没有 `useEffect` 的闭包问题
  - 渲染函数仅执行一次，不会存在组件实例上多个闭包的情况
- **真正的响应式**，只关心数据的变化
  - `React` 相应的是组件树的变化，`SolidJS` 相应的是数据的变化，哪怕数据定义在申明之外

</v-clicks>

---
transition: fade-out
layout: intro
---

# 我不明白！

看起来 `count()` 和 `count` 没什么区别

---

# StateReference

`state-getter` vs `state-value`

```ts
useState() => value + setter
useSignal() => getter + setter
```

<br />

为了反应性，`Signal` 必须收集谁对 `Signal` 的值感兴趣

而这可以通过观察 `state-getter` 的调用来知晓

<br />

<div text-orange v-click
>
也就是说，调用 getter 会创建一个<b>订阅</b>
</div>

---

# Case

考虑一个购物车的实现如下：

<div class="grid grid-cols-2 gap-4">

```jsx
function App() {
  console.log("App rendered");
  const [cart, setCart] = useState([]);

  return (
    <div>
      <Main setCart={setCart} />
      <NavBar cart={cart} />
    </div>
  )
}

function Main({ setCart }) {
  console.log("Main rendered");
  // add to cart...
}
```

<div>

```jsx
function NavBar({ cart }) {
  console.log("NavBar rendered");
  return <Cart cart={cart} />
}
```

<div mt-4 />

```jsx
function Cart({ cart }) {
  console.log("Cart rendered");
  return <div>{cart}</div>
}
```

</div>

</div>

<br />
<div>
  <button v-click @click="$slidev.nav.next" class="px-2 py-1 rounded mr-4" hover="bg-white bg-op10">
    Add to Cart
  </button>
  <div class="text-gray inline-flex gap-4">
    <v-clicks>
      <span>App rendered</span>
      <span>Main rendered</span>
      <span>NavBar rendered</span>
      <span>Cart rendered</span>
    </v-clicks>
  </div>
</div>
<button v-click @click="$slidev.nav.next" class="px-2 py-1 rounded mr-4 text-blue-500" hover="bg-white bg-op10">
  Add to Cart
</button>
<div class="text-blue-300 inline-flex gap-4" v-click>
  <span>Cart rendered</span>
</div>

---
layout: center
---

# `Signals` 现状

---

# 历史
以及支持情况

支持 `Signals` 的流行框架如 <a font-mono op75 hover:op100 href="https://vuejs.org">Vue</a>、
<a font-mono op75 hover:op100 href="https://angular.io">Angular</a>、
<a font-mono op75 hover:op100 href="https://solidjs.com">Solid</a>、
<a font-mono op75 hover:op100 href="https://preactjs.com">Preact</a>、
<a font-mono op75 hover:op100 href="https://qwik.dev">Qwik</a> 等

<br />

事实上，`Signal` 并不是新概念，它可以追溯到十多年前的 `Knockout` 等实现。
`Vue` 的选项式 API 和 `React` 的状态管理库 `Mobx` 也是基于同样的原则。 

---
transition: fade-out
---

# 未来

`TC39` 已起草 `Signals` [标准提案](https://github.com/tc39/proposal-signals?tab=readme-ov-file)，即成为 `JavaScript` 的原生 `API`

<v-click>

使用标准 `Signals` 的原生代码：

```ts
const counter = new Signal.State(0);
const isEven = new Signal.Computed(() => (counter.get() & 1) == 0);
const parity = new Signal.Computed(() => isEven.get() ? "even" : "odd");

// Simulate external updates to counter...
setInterval(() => counter.set(counter.get() + 1), 1000);
```

</v-click>

<br />

<v-click>

### 动机

<p>提案的目标是<b text-yellow>完全解耦</b>响应式模型和渲染视图，使开发者能够无需重写非 UI 代码即可迁移到新的渲染技术，或在 JS 中开发共享的响应式模型用于不同的环境部署</p>

</v-click>

---
layout: end
---

## 谢谢！

---

# Clicks Animations

You can add `v-click` to elements to add a click animation.

<div v-click>

This shows up when you click the slide:

```html
<div v-click>This shows up when you click the slide.</div>
```

</div>

<br>

<v-click>

The <span v-mark.red="3"><code>v-mark</code> directive</span>
also allows you to add
<span v-mark.circle.orange="4">inline marks</span>
, powered by [Rough Notation](https://roughnotation.com/):

```html
<span v-mark.underline.orange>inline markers</span>
```

</v-click>

<div mt-20 v-click>

[Learn More](https://sli.dev/guide/animations#click-animations)

</div>

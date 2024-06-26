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
transition: fade-out
# enable MDC Syntax: https://sli.dev/guide/syntax#mdc-syntax
mdc: true
fonts:
  sans: nunito
  mono: Input Mono
---

# 浅聊响应式系统

从 `SolidJS` 中理解 `Signals`

<author>wzz</author>

---
layout: center
class: text-center
name: checkIn
---

<div class="font-serif text-2xl">Check In</div>

<br />

<div class="grid grid-cols-2 gap-50">
  <div>
    <span>签到</span>
    <img src="/sign.png" class="h-60 mt-6" />
  </div>
  <div>
    <span>评估</span>
    <img src="/evo.png" class="h-60 mt-6" />
  </div>
</div>

---
layout: center
name: Reactivity
---

# <span class="text-white">响应性 Reactivity </span>

<!--
从以前，JS直接修改html，到jQuery，再到React，Vue等框架，前端开发的方式发生了翻天覆地的变化。其核心目标都是将View层与Model层进行同步，使得数据的变化能够自动反映到视图上。这种机制就是响应式系统。现代前端框架中，响应式系统的实现方式有很多种，比如Vue的响应式对象，React的Hooks，Mobx等。而SolidJS则是基于Signal的响应式系统。我们将从SolidJS的角度来理解Signal。
-->

---

# 什么是响应性？

<v-clicks>

```ts {monaco-run}
let A0 = 1
let A1 = 2
let A2 = A0 + A1

console.log(A2) // 3

A0 = 2
console.log(A2) // 仍然是 3
```

<div class="mt-4">

为了重新运行计算的代码来更新 `A2`，我们需要将其包装成一个函数

</div>

</v-clicks>

<!--
假如有一个Excel表格，单元格A2的值通过公式A0+A1来定义的，当我们更改A0或A1的时候，A2也会自动更新。
但是JS默认不是这样的，如果我们用js写逻辑 [click]
当我们更改 A0 后，A2 不会自动更新
-->

---
name: 进入响应式系统
---

# 进入响应性世界

```ts {all|3|4|all}
let A2

function update() {
  A2 = A0 + A1
}
```

<v-click>

魔法函数：

```ts
whenDepsChange(update)
```

</v-click>

<v-clicks>

- 当一个变量被读取时进行追踪
- 如果一个变量在当前运行的副作用中被读取了，就将该副作用设为此变量的一个订阅者
- 探测一个变量的变化

</v-clicks>

<!--
我们需要定义几个术语: [click]
这个 update() 函数会产生一个副作用，或者就简称为作用 (effect)，因为它会更改程序里的状态。[click]
A0 和 A1 被视为这个作用的依赖 (dependency)，因为它们的值被用来执行这个作用。因此这次作用也可以被称作它的依赖的一个订阅者 (subscriber)。
[click] 我们需要一个魔法函数，能够在 A0 或 A1 (这两个依赖) 变化时调用 update() (产生作用)。[click] 它有这些任务：
[click] 例如我们执行了表达式 A0 + A1 的计算，则 A0 和 A1 都被读取到了。
[click] 例如由于 A0 和 A1 在 update() 执行时被访问到了，则 update() 需要在第一次调用之后成为 A0 和 A1 的订阅者。
[click] 例如当我们给 A0 赋了一个新的值后，应该通知其所有订阅了的副作用重新执行。
-->

---

# React

##### 函数式组件

````md magic-move
```tsx
function Counter() {
  // 定义一个状态变量和更新它的函数
  const [count, setCount] = useState(0);

  // 一个会更新状态的函数
  function increment() {
    setCount(count + 1);
  }

  return (
    <div>
      <button onClick={increment}>Click me</button>
      <p>You clicked {count} times</p>
    </div>
  );
}
```

```tsx
function DataFetcher({ url }) {
  const [data, setData] = useState(null);

  // 副作用
  useEffect(() => {
    fetchData(url).then(response => setData(response));
  }, [url]); // 声明依赖：当url变化时重新运行副作用

  if (!data) {
    return <div>Loading...</div>;
  }

  return (
    <div>
      <h1>Data</h1>
      <pre>{JSON.stringify(data, null, 2)}</pre>
    </div>
  );
}
```

```tsx
render(Component, container) {
  // 组件函数被调用，生成虚拟DOM
  const virtualDOM = Component();
  
  // 协调：比较新旧虚拟DOM，找出差异
  const updates = diff(virtualDOM, container);
  
  // 提交：将差异更新到真实DOM
  commit(updates);
}
```
````

<v-clicks>

- 当一个组件的状态变化时，React会检查该组件是否依赖了这个状态
- 调用组件函数，生成虚拟DOM
- 比较新旧虚拟DOM，找出差异

</v-clicks>

<!--
定义状态变量，我们称之为state
[click] 定义副作用，当依赖变化时重新执行
[click] 在协调阶段，React重新调用组件函数，并生成新的虚拟DOM，使用Diff算法来比较新旧虚拟DOM，找出需要更新的部分。这个时候，如果组件的状态更新导致了子组件的属性发生变化，React会递归的对子组件进行上述过程。
-->

---
layout: center
---

# 进入正题

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

<!--
无论内容是否拥有状态管理，也无论该组件是否接受来自父组件的 Props 透传，都仅触发一次渲染函数
-->

---
layout: fact
---

# 面向状态驱动

<!--
SolidJS 叫板 React 的核心理念：面向状态驱动而不是面向视图驱动。正因为这个差异，导致了渲染函数仅执行一次，也顺便衍生出变量更新粒度如此之细的结果，同时也是其高性能的基础，同时也解决了 React Hooks 不够直观的顽疾
-->

---
transition: slide-up
---

# 更完善的响应式

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

<!--
[click] 我们要完全以不一样心智理解这段代码，而不是 React 心智理解它，虽然它长得太像 Hooks 了。一个显著的不同是，将状态代码提到外层也完全能 Work。
[click] 这是最快理解 SolidJS 理念的方式，即 SolidJS 根本没有理 React 那套概念，SolidJS 理解的数据驱动是纯粹的数据驱动视图，无论数据在哪定义，视图在哪，都可以建立绑定。
[click] 这个设计自然也不依赖渲染函数执行多次，同时因为使用了依赖收集，也不需要手动申明 deps 数组，也完全可以将 createSignal 写在条件分支之后，因为不存在执行顺序的概念。
-->

---

# 派生状态追踪

```jsx twoslash
import { createSignal } from "solid-js";

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

<!--
为什么 SolidJS 这样看似简单的衍生状态写法可以生效。原因在于，SolidJS 收集所有用到了 count() 的依赖，而 doubleCount() 用到了它，而渲染函数用到了 doubleCount()，仅此而已，所以自然挂上了依赖关系，这个实现过程简单而稳定，没有 Magic。
-->

---

# Effects

状态监听

```jsx twoslash
import { createSignal, createEffect } from "solid-js";

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

<p><span v-mark.red="2"> Signals </span></p>

</v-click>

<!--
Angular、Qwik的作者MIŠKO HEVERY发文表示Signal是前端框架的未来，并考虑在Angular中实现它。其实还有个核心原因是在 SolidJS 增加的模板编译过程上，不过不在此处分享主题之内
-->

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

<!--
state耦合了对状态的引用以及对状态值的获取这两个含义。
getter是对状态的引用，getter()是对状态值的获取，也就是说，我们可以不必立刻获取状态的值，而是在需要的时候再获取，这么做有什么好处呢？如果我们在需要的时候再获取状态的值，就能感知当前的上下文环境。

signal的本质，是将对状态的引用以及对状态值的获取分离开，让数据驱动真的回归到数据上，而非与 UI 树绑定。
-->

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
      <span>"App rendered"</span>
      <span>"Main rendered"</span>
      <span>"NavBar rendered"</span>
      <span>"Cart rendered"</span>
    </v-clicks>
  </div>
</div>
<button v-click @click="$slidev.nav.next" class="px-2 py-1 rounded mr-4 text-blue-500" hover="bg-white bg-op10">
  Add to Cart
</button>
<div class="text-blue-300 inline-flex gap-4" v-click>
  <span>"Cart rendered"</span>
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

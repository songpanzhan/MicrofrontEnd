# 组合式 API：setup()

## 基本使用

`setup()` 钩子是在组件中使用组合式 API 的入口，通常只在以下情况下使用：

1. 需要在非单文件组件中使用组合式 API 时。

2. 需要在基于选项式 API 的组件中集成基于组合式 API 的代码时。

   > [!NOTE]
   >
   > 注意
   >
   > 对于结合单文件组件使用的组合式 API，推荐通过 [``](https://cn.vuejs.org/api/sfc-script-setup.html) 以获得更加简洁及符合人体工程学的语法。

   

我们可以使用[响应式 API](https://cn.vuejs.org/api/reactivity-core.html) 来声明响应式的状态，在 `setup()` 函数中返回的对象会暴露给模板和组件实例。其他的选项也可以通过组件实例来获取 `setup()` 暴露的属性：

```js
<script>
import { ref } from 'vue'

export default {
  setup() {
    const count = ref(0)

    // 返回值会暴露给模板和其他的选项式 API 钩子
    return {
      count
    }
  },

  mounted() {
    console.log(this.count) // 0
  }
}
</script>

<template>
  <button @click="count++">{{ count }}</button>
</template>
```

在模板中访问从 `setup` 返回的 [ref](https://cn.vuejs.org/api/reactivity-core.html#ref) 时，它会[自动浅层解包](https://cn.vuejs.org/guide/essentials/reactivity-fundamentals.html#deep-reactivity)，因此你无须再在模板中为它写 `.value`。当通过 `this` 访问时也会同样如此解包。

`setup()` 自身并不含对组件实例的访问权，即在 `setup()` 中访问 `this` 会是 `undefined`。你可以在选项式 API 中访问组合式 API 暴露的值，但反过来则不行。

`setup()` 应该*同步地*返回一个对象。唯一可以使用 `async setup()` 的情况是，该组件是 [Suspense](https://cn.vuejs.org/guide/built-ins/suspense.html) 组件的后裔。

## 访问 Props

`setup` 函数的第一个参数是组件的 `props`。和标准的组件一致，一个 `setup` 函数的 `props` 是响应式的，并且会在传入新的 props 时同步更新。

```js
export default {
  props: {
    title: String
  },
  setup(props) {
    console.log(props.title)
  }
}
```

请注意如果你解构了 `props` 对象，解构出的变量将会丢失响应性。因此我们推荐通过 `props.xxx` 的形式来使用其中的 props。

如果你确实需要解构 `props` 对象，或者需要将某个 prop 传到一个外部函数中并保持响应性，那么你可以使用 [toRefs()](https://cn.vuejs.org/api/reactivity-utilities.html#torefs) 和 [toRef()](https://cn.vuejs.org/api/reactivity-utilities.html#toref) 这两个工具函数：

```js
import { toRefs, toRef } from 'vue'

export default {
  setup(props) {
    // 将 `props` 转为一个其中全是 ref 的对象，然后解构
    const { title } = toRefs(props)
    // `title` 是一个追踪着 `props.title` 的 ref
    console.log(title.value)

    // 或者，将 `props` 的单个属性转为一个 ref
    const title = toRef(props, 'title')
  }
}
```

## Setup 上下文

传入 `setup` 函数的第二个参数是一个 **Setup 上下文**对象。上下文对象暴露了其他一些在 `setup` 中可能会用到的值：

```js
export default {
  setup(props, context) {
    // 透传 Attributes（非响应式的对象，等价于 $attrs）
    console.log(context.attrs)

    // 插槽（非响应式的对象，等价于 $slots）
    console.log(context.slots)

    // 触发事件（函数，等价于 $emit）
    console.log(context.emit)

    // 暴露公共属性（函数）
    console.log(context.expose)
  }
}
```

该上下文对象是非响应式的，可以安全地解构：

```js
export default {
  setup(props, { attrs, slots, emit, expose }) {
    ...
  }
}
```

`attrs` 和 `slots` 都是有状态的对象，它们总是会随着组件自身的更新而更新。这意味着你应当避免解构它们，并始终通过 `attrs.x` 或 `slots.x` 的形式使用其中的属性。此外还需注意，和 `props` 不同，`attrs` 和 `slots` 的属性都**不是**响应式的。如果你想要基于 `attrs` 或 `slots` 的改变来执行副作用，那么你应该在 `onBeforeUpdate` 生命周期钩子中编写相关逻辑。

### 暴露公共属性

`expose` 函数用于显式地限制该组件暴露出的属性，当父组件通过[模板引用](https://cn.vuejs.org/guide/essentials/template-refs.html#ref-on-component)访问该组件的实例时，将仅能访问 `expose` 函数暴露出的内容：

```js
export default {
  setup(props, { expose }) {
    // 让组件实例处于 “关闭状态”
    // 即不向父组件暴露任何东西
    expose()

    const publicCount = ref(0)
    const privateCount = ref(0)
    // 有选择地暴露局部状态
    expose({ count: publicCount })
  }
}
```

## 与渲染函数一起使用

`setup` 也可以返回一个[渲染函数](https://cn.vuejs.org/guide/extras/render-function.html)，此时在渲染函数中可以直接使用在同一作用域下声明的响应式状态：

```js
import { h, ref } from 'vue'

export default {
  setup() {
    const count = ref(0)
    return () => h('div', count.value)
  }
}
```

如果不使用 `<script setup>`，需确保从 `setup()` 返回 ref：

```js
export default {
  setup() {
    const input = ref(null)
    // ...
    return {
      input
    }
  }
}
```

注意，你只可以**在组件挂载后**才能访问模板引用。如果你想在模板中的表达式上访问 `input`，在初次渲染时会是 `null`。这是因为在初次渲染前这个元素还不存在呢！

如果你需要侦听一个模板引用 ref 的变化，确保考虑到其值为 `null` 的情况：

```js
watchEffect(() => {
  if (input.value) {
    input.value.focus()
  } else {
    // 此时还未挂载，或此元素已经被卸载（例如通过 v-if 控制）
  }
})
```

## `v-for` 中的模板引用

> 需要 v3.2.25 及以上版本

当在 `v-for` 中使用模板引用时，对应的 ref 中包含的值是一个数组，它将在元素被挂载后包含对应整个列表的所有元素：

```vue
<script setup>
import { ref, onMounted } from 'vue'

const list = ref([
  /* ... */
])

const itemRefs = ref([])

onMounted(() => console.log(itemRefs.value))
</script>

<template>
  <ul>
    <li v-for="item in list" ref="itemRefs">
      {{ item }}
    </li>
  </ul>
</template>
```

应该注意的是，ref 数组**并不**保证与源数组相同的顺序。

## 函数模板引用

除了使用字符串值作名字，`ref` attribute 还可以绑定为一个函数，会在每次组件更新时都被调用。该函数会收到元素引用作为其第一个参数：

```html
<input :ref="(el) => { /* 将 el 赋值给一个数据属性或 ref 变量 */ }">
```

注意我们这里需要使用动态的 `:ref` 绑定才能够传入一个函数。当绑定的元素被卸载时，函数也会被调用一次，此时的 `el` 参数会是 `null`。你当然也可以绑定一个组件方法而不是内联函数。

## 组件上的 ref

模板引用也可以被用在一个子组件上。这种情况下引用中获得的值是组件实例：

```vue
<script setup>
import { ref, onMounted } from 'vue'
import Child from './Child.vue'

const child = ref(null)

onMounted(() => {
  // child.value 是 <Child /> 组件的实例
})
</script>

<template>
  <Child ref="child" />
</template>
```


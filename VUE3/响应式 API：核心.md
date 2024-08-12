# 响应式 API：核心

> [!TIP]
>
> 要更好地了解响应式 API，推荐阅读下面几个指南中的章节：
>
> - [响应式基础](https://cn.vuejs.org/guide/essentials/reactivity-fundamentals.html) (with the API preference set to Composition API)
> - [深入响应式系统](https://cn.vuejs.org/guide/extras/reactivity-in-depth.html)

## ref()

接受一个内部值，返回一个响应式的、可更改的 ref 对象，此对象只有一个指向其内部值的属性 `.value`。

- **类型**

```js
function ref<T>(value: T): Ref<UnwrapRef<T>>

interface Ref<T> {
  value: T
}
```

- **详细信息**

ref 对象是可更改的，也就是说你可以为 `.value` 赋予新的值。它也是响应式的，即所有对 `.value` 的操作都将被追踪，并且写操作会触发与之相关的副作用。

如果将一个对象赋值给 ref，那么这个对象将通过 [reactive()](https://cn.vuejs.org/api/reactivity-core.html#reactive) 转为具有深层次响应式的对象。这也意味着如果对象中包含了嵌套的 ref，它们将被深层地解包。

若要避免这种深层次的转换，请使用 [`shallowRef()`](https://cn.vuejs.org/api/reactivity-advanced.html#shallowref) 来替代。

- **示例**

```js
const count = ref(0)
console.log(count.value) // 0

count.value = 1
console.log(count.value) // 1
```


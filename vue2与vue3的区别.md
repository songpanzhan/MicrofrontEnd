## vue2 VS vu3



| 对比     | vue2                  | vue3                       |
| -------- | --------------------- | -------------------------- |
| 脚手架   | vue-cli(命令式）      | 可视化创建脚手架           |
| 组件通信 |                       |                            |
| 数据监听 | watch,computed        | watch,watchEffect,computed |
| 双向绑定 | Object.defineProperty | ProxyAPI                   |
| 生命周期 |                       |                            |
| api      | 选项式                | 组件式                     |

## 双向绑定更新

1. **vue2** 的双向数据绑定是利⽤ES5 的⼀个 API ，Object.defineProperty()对数据进⾏劫持 结合 发布订阅模式的⽅式来实现的。

2. **vue3** 中使⽤了 ES6 的 ProxyAPI 对数据代理，通过 reactive() 函数给每⼀个对象都包⼀层 Proxy，通过 Proxy 监听属性的变化，从⽽ 实现对数据的监控。

   > 这⾥是相⽐于vue2版本，使⽤proxy的优势如下
   >
   > 1.defineProperty只能监听某个属性，不能对全对象监听 可以省去for in、闭包等内容来提升效率（直接绑定整个对象即可）
   >
   > 2.可以监听数组，不⽤再去单独的对数组做特异性操作,通过Proxy可以直接拦截所有对象类型数据的操作，完美⽀持对数组的监听

## 实例化

> 1、Vue2.x中new出的实例对象，所有的东西都在这个vue对象上，这样其实⽆论你⽤到还是没⽤到，都会跑⼀遍，这样不仅提⾼了性能消耗，也⽆疑增加了⽤户加载时间。
>
> 2、⽽vue3.0中可以⽤ES module imports按需引⼊，如：keep-alive内置组件、v-model指令，等等，不仅我们开发起来更加的便捷，减少 了内存消耗，也同时减少了⽤户加载时间，优化⽤户体验。

## 生命周期

![e8256adeaa1d4ab79df2cea118b8b38e.png](https://i-blog.csdnimg.cn/blog_migrate/b157711b9e29bd144399c0c25120ac29.png)

### 获取props
vue2在script代码块可以直接获取props，vue3通过setup指令传递

```js
vue2：console.log(‘props’,this.xxx)
vue3：setup(props,context){ console.log(‘props’,props) }
```

### 数据和方法的定义
Vue2使⽤的是选项类型API（Options API），Vue3使⽤的是合成型API（Composition API）

Vue2：

```
data() { return {}; }, methods:{ }
```

Vue3：

数据和⽅法都定义在setup中，并统⼀进⾏return{}

### 给父组件传值emit
```
vue2：this.$emit()
vue3：setup(props,context){context.emit()}
```

### watchEffect

> Vue3中除了watch，还引入了副作用监听函数watchEffect，用过之后我发现它和React中的useEffect很像，只不过watchEffect不需要传入依赖项。
>
> 那么什么是watchEffect呢？
>
> watchEffect它会立即执行传入的一个函数，同时响应式追踪其依赖，并在其依赖变更时重新运行该函数。
>
> computed和watch所依赖的数据必须是响应式的。Vue3引入了watchEffect,watchEffect 相当于将 watch 的依赖源和回调函数合并，当任何你有用到的响应式依赖更新时，该回调函数便会重新执行。不同于 watch的是watchEffect的回调函数会被立即执行，即（{ immediate: true }）。
> 

## 组件通信

![0244e04dc98042a88b8947ed634dd910.png](https://i-blog.csdnimg.cn/blog_migrate/9dd38dc0261649b6aff7bb27d126bf2e.png)

#### 注意
> props中数据流是单项的，即子组件不可改变父组件传来的值
>
> 在组合式API中，如果想在子组件中用其它变量接收props的值时需要使用toRef将props中的属性转为响应式。
>
> attrs和listeners
> 子组件使用$attrs可以获得父组件除了props传递的属性和特性绑定属性 (class和 style)之外的所有属性。
>
> 子组件使用$listeners可以获得父组件(不含.native修饰器的)所有v-on事件监听器，在Vue3中已经不再使用；但是Vue3中的attrs不仅可以获得父组件传来的属性也可以获得父组件v-on事件监听器

### 路由

#### vue3和vue2路由常用功能只是写法上有些区别：

vue3的beforeRouteEnter作为路由守卫的示例是因为它在setup语法糖中是无法使用的；大家都知道setup中组件实例已经创建，是能够获取到组件实例的。而beforeRouteEnter是再进入路由前触发的，此时组件还未创建，所以是无法用在setup中的；如果想在setup语法糖中使用则需要再写一个script 如下：

```js
<script>
export default {
  beforeRouteEnter(to, from, next) {
    // 在渲染该组件的对应路由被 confirm 前调用
    next()
  },
};
</script>
```

vue3路由写法：

```js
<script>
import { defineComponent } from 'vue'
import { useRoute, useRouter } from 'vue-router'
export default defineComponent({
  beforeRouteEnter (to, from, next) {
    // 在渲染该组件的对应路由被 confirm 前调用
    next()
  },
  beforeRouteLeave ((to, from, next)=>{//离开当前的组件，触发
    next()       
  }),
  beforeRouteLeave((to, from, next)=>{//离开当前的组件，触发
    next()      
  }),
  setup() {
    const router = useRouter()
    const route = useRoute()
    const toPage = () => {
      router.push(xxx)
    }
 
    //获取params 注意是route
    route.params
    //获取query
    route.query
    return {
      toPage
    }
  },
});
</script>
```

vue2写法：

```js
<script>
export default {
  beforeRouteEnter (to, from, next) {
    // 在渲染该组件的对应路由被 confirm 前调用
    next()
  },
  beforeRouteEnter (to, from, next) {
    // 在渲染该组件的对应路由被 confirm 前调用
    next()
  },
  beforeRouteLeave ((to, from, next)=>{//离开当前的组件，触发
    next()       
  }),
  beforeRouteLeave((to, from, next)=>{//离开当前的组件，触发
    next()      
  }),
  methods:{
    toPage(){
      //路由跳转
      this.$router.push(xxx)
    }
  },
  created(){
    //获取params
    this.$route.params
    //获取query
    this.$route.query
  }
}
</script>
```

### vue3的优点

> setup带来的改变：
>
> 1.解决了vue2的data和methods方法相距太远，无法组件之间复用
>
> 2.提供了script标签引入共同业务逻辑的代码块，顺序执行
>
> 3.script变成setup函数，默认暴露给模版
>
> 4.组件直接挂载，无需注册
>
> 5.自定义的指令也可以在模版中自动获得
>
> 6.this不再是这个活跃实例的引用
>
> 7.带来的大量全新api，比如defineProps，defineEmits，withDefault，toRef，toRefs
>
> ref带来的改变：
>
> Vue 提供了一个 ref() 方法来允许我们创建可以使用任何值类型的响应式数据
>
> Ref作TS的类型标注
>
> reactive带来的改变：
>
> 可以使用 reactive() 函数创建一个响应式对象或数组
>
> reactive可以隐式地从它的参数中推导类型
>
> 使用interface进行类型标注
>
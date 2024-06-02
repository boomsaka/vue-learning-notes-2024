# Vue3

## 1. Vue3的简介

### 1.1 性能的提升

- 打包大小减少41%
- 初次渲染快55%，更新渲染快133%
- 内存减少54%

### 1.2 源码的升级
- 使用Proxy代替defineProperty实现响应式
- 重写虚拟DOM的实现和Tree-Shaking

### 1.3 拥抱TypeScript
- Vue3可以更好的支持TypeScript

### 1.4 新的特性
1.  Composition API（组合API）
    - setup
    - ref与reactive
    - computed与watch
    ...
2. 新的内置组件
    - Fragment
    - Teleport
    - Suspense
    ...
3. 其他改变：
    - 新的生命周期钩子
    - data选项应始终被声明为一个函数
    - 一出keyCode支持作为v-on的修饰符
    ...

## 2. 创建Vue3工程

### 2.2基于vite创建 - 和webpack等价
vite是新一代前端构建工具
- 轻量快速的热承载（HMR），能实现快速的服务启动
- 对TypeScript、JSX、CSS登实现极速的服务器动
- 真正的按需编译，不在等待整个应用编译完成
- 用webpack架构的时候，当路由特别多的时候，项目启动非常慢
- 用vite的时候，等待用户，用户看什么，再去处理什么，而不像webpack不管你看什么，全部都处理

## 3. Vue3核心语法

### 3.1 【Options API】与【CompositionAPI】
- `Vue2`的`API`设计是`Options`（配置）风格的
- `Vue3`的`API`设计是`Composition`（组合）风格的

Options API 的弊端

Options类型的API，数据、方法、计算属性等，是分散在：`data`、`methods`、`computed`中的，若想新增或者修改一个需求，就需要分别修改：`data`、`methods`、`computed`，不便于维护和复用。

Composition API的优势

可以用函数的方式，更优雅的组织代码，让相关功能的代码更加有序的组织在一起。

### 3.2 setup

setup与Options API的关系
- Vue2的配置（data、methods...）中可以访问到setup中的属性、方法
- 但在setup中不能访问到Vue2的配置（data、methods...）
- 如果与Vue2冲突，则setup优先

setup语法糖
setup函数有一个语法糖，这个语法糖可以让我们把setup独立出去，代码如下

```vue
<template>
    <div class="person">
        <h2>姓名：{{name}}</h2>
        <h2>年龄：{{age}}</h2>
        <button @click="changeName">修改名字</button>
        <button @click="changeAge">年龄+1</button>
        <button @click="showTel">点击查看联系方式</button>
    </div>
</template>

<script lang="ts">
    export default {
        name: 'Person',
    }
</script>

<!-- 下面的写法是setup语法糖 -->
<script setup lang='ts'>
    console.log(this) //undefined

    // 数据（注意：此时的name、age、tel都不是响应式数据
    let name = '张三'
    let age = 18
    let tel = '1388888889'

    // 方法
    function changeName() {
        name = '李四'
    }
    function changeAge() {
        age += 1
    }
    function showTel() {
        alert(tel)
    }
</script>
```

扩展：上述代码，还需要编写一个不写setup的标签，去指定组件名字，比较麻烦，我们可以借助vite中的插件简化
1. 第一步：`npm i vite-plugin-vue-setup-extend -D`
2. 第二步：`vite.config.ts`

```ts
import (defineConfig) from 'vite'
import VueSetupExtend from 'vite-plugin-vue-setup-extend'

export default defineConfig({
    plugins: [VueSetupExtend()]
})
```

### 3.3 ref创建：基本类型的响应式数据

- 作用：定义响应式变量
- 语法：`let xxx = ref(初始值)`
- 返回值：一个`RefImpl`的实例对象，简称`ref`对象或`ref`，`ref`对象的`value`属性是响应式的
- 注意点：
    - `JS`中操作数据需要：`xxx.value`，但模板中不需要`.value`，直接使用即可
    - 对于`let name = ref('张三')`来说，`name`不是响应式的，`name.value`是响应式的

```vue
<template>
    <div class="person">
        <h2>姓名：{{name}}</h2>
        <h2>年龄：{{age}}</h2>
        <button @click="changeName">修改名字</button>
        <button @click="changeAge">年龄+1</button>
        <button @click="showTel">点击查看联系方式</button>
    </div>
</template>

<script lang="ts">
    export default {
        name: 'Person',
    }
</script>

<!-- 下面的写法是setup语法糖 -->
<script setup lang='ts'>
    import {ref} from 'vue'

    // name和age是一个RefImpl的实例对象，简称ref对象，它们的value属性是响应式的
    let name = ref('张三')
    let age = ref(18)
    let tel = '1388888889'

    // 方法
    function changeName() {
        name.value = '李四'
    }
    function changeAge() {
        age.value += 1
    }
    function showTel() {
        alert(tel)
    }
</script>
```

### 3.4 reactive只能创建对象类型的响应式数据：

```vue
<template>
    <div class="person">
        <h2>一辆{{car.brand}}车，价值{{car.price}}万</h2>
        <button @click="changePrice">修改汽车价格</button>
        <br>
        <h2>游戏列表：</h2>
        <ul>
            <li v-for="g in games" :key="g.id">{{g.name}}</li>
        </ul>
        <button @click="changeFirstGame">修改第一个游戏的名字</button>
    </div>
</template>

<script lang="ts" setup name="Person">
    import {reactive} from 'vue'

    // 数据
    let car = reactive({brand:"奔驰", price: 100})
    let games = reactive([
        {id: 'dscjnsdkj1', name: '王者荣耀'},
        {id: 'dscjnsdkj2', name: '原神'},
        {id: 'dscjnsdkj3', name: '三国志'},
    ])

    // 方法
    function changePrice() {
        car.price += 10
    }
    function changeFirstGame() {
        games[0].name = '马里奥'
    }
</script>

<style scoped>
</style>
```

### 3.5 ref创建：对象类型的响应式数据

```vue
<template>
    <div class="person">
        <h2>一辆{{car.brand}}车，价值{{car.price}}万</h2>
        <button @click="changePrice">修改汽车价格</button>
        <br>
        <h2>游戏列表：</h2>
        <ul>
            <li v-for="g in games" :key="g.id">{{g.name}}</li>
        </ul>
        <button @click="changeFirstGame">修改第一个游戏的名字</button>
    </div>
</template>

<script lang="ts" setup name="Person">
    import {ref} from 'vue'

    // 数据
    let car = ref({brand:"奔驰", price: 100})
    let games = ref([
        {id: 'dscjnsdkj1', name: '王者荣耀'},
        {id: 'dscjnsdkj2', name: '原神'},
        {id: 'dscjnsdkj3', name: '三国志'},
    ])

    // 方法
    function changePrice() {
        car.value.price += 10
    }
    function changeFirstGame() {
        games.value[0].name = '马里奥'
    }
</script>

<style scoped>

</style>
```

### 3.6 ref对比reactive

宏观角度看：
1. `ref`用来定义：基本类型数据、对象类型数据
2. `reactive`用来定义：对象类型数据

- 区别：
    1. `ref`创建的变量必须使用`.value`（可以使用`volar`插件自动添加`.value`）
    2. reactive重新分配了一个新对象，会失去响应式（可以使用Object.assign去整体替换）
    ```ts

    let car = reactive({brand: '宝马', price: 100})

    function changeCar() {
        // car = {brand: '奥拓', price: 1} // 这个写页面不更新
        // car - reactive({brand: '奥拓', price: 1}) //这么写页面不更新

        // 下面这个写法页面可以更新
        Object.assign(car, {brand: '奥拓', price: 1})
    }

    ```
- 使用原则：
    1. 若需要一个基本类型的响应式数据，必须使用`ref`
    2. 若需要一个响应式对象，层级不深，`ref`、`reactive`都可以
    3. 若需要一个响应式对象，且层级较深，推荐使用`reactive`

### 3.7 toRefs与toRef

- 作用：将一个响应式对象中的每一个属性，转换为ref对象
- 备注：toRefs与toRef功能一致，但toRefs可以批量转换
- 语法如下：
    ```vue
    <template>
        <div class="person">
            <h2>姓名：{{name}}</h2>
            <h2>年龄：{{age}}</h2>
            <button @click="changeName">修改名字</button>
            <button @click="changeAge">修改年龄</button>
        </div>
    </template>

    <script lang="ts" setup name="Person">
        import {reactive, toRefs, toRef} from 'vue'

        let person = reactive({
            name: '张三',
            age: 18
        })

        // 解构
        let {name, age} = toRefs(person)
        let ageRef = toRef(person, 'age')

        function changeName() {
            name.value += '~'
        }
        function changeAge() {
            age.value += 1
        }
    </script>

    <style scoped></style>
    ```

### 3.8 Computed

作用：根据已有数据计算出新数据

```vue
<template>
    <div class="person">
        姓：<input type="text" v-model="firstName">
        名：<input type="text" v-model="lastName">
        全名：<span>{{fullName}}</span>
        <button @click="changeFullName">修改全名</button>
    </div>
</template>

<script lang="ts" setup name="Person">
    import {ref} from 'vue'

    let firstName = ref('张')
    let lastName = ref('三')

    // 这么定义的fullName是一个计算属性，且是只读的
    // let fullName = computed(()=>{
    //     return firstName.value.slice(0,1).toUpperCase() + firstName.value.slice(1) + '-' + lastName.value
    // })
    
    // 这么定义的fullName是一个计算属性，可读可写
    let fullName = computed({
        get(){
            return firstName.value.slice(0,1).toUpperCase() + firstName.value.slice(1) + '-' + lastName.value
        },
        set(val) {
            const [str1, str2] = val.split('-')
            firstName.value = str1
            lastName.value = str2
        }
    })
    function changeFullName() {
        fullName.value = 'li-si'
    }



</script>

<style scoped></style>

```

### 3.9 watch

- 作用：监视数据的变化
- 特点：Vue3中的watch只能监视以下四中数据：
    1. ref定义的数据
    2. reactive定义的数据
    3. 函数返回一个值
    4. 一个包含上述内容的数组

#### 情况一

监视ref定义的”基本类型“数据：直接写数据名即可，监视的是其的value值的改变

```vue
<template>
    <div class="person">
        <h2>当前求和为：{{sum}}</h2>
        <button @click="changeSum">点我sum+1</button>
    </div>
</template>

<script lang="ts" setup name="Person">
    import {ref, watch} from 'vue'

    // 数据
    let sum = ref(0)

    // 方法
    function changeSum() {
        sum.value += 1
    }

    // 监视：情况一：监视【ref】定义的【基本类型】数据
    watch(sum, (newValue, oldValue)=>{
        console.log('sum变化了')
        if (newValue >= 10) {
            stopWatch()
        }
    })

</script>

<style scoped></style>

```

#### 情况二

监视ref定义的【对象类型】数据：直接写数据名、监视的是对象的【地址值】，若想监视对象内部的数据，要手动开启深度监视。

```vue
<template>
    <div class="person">
        <h2>姓名：{{person.name}}</h2>
        <h2>年龄：{{person.age}}</h2>
        <button @click="changeNanme"修改名字></button>
        <button @click="changeAge">修改年龄</button>
    </div>
</template>

<script lang="ts" setup name="Person">
    import {ref, watch} from 'vue'


</script>

<style scoped></style>

```

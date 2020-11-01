# 一、简答题

### 1、简述 Vue 首次渲染的过程

- 渲染之前，首先进行 Vue 初始化，初始化实例成员以及静态成员
- 初始化结束后，调用 Vue 的构造函数`new Vue()`
- 在构造函数中调用 `this_init()`，\_init() 方法相当于整个 Vue 的入口
- 最终在 \_init() 方法中调用 `vm.$mount()`,其中有两个 \$mount() 方法
- 第一个 \$mount() 在 src\platforms\web\entry-runtime-with-compile.js 文件

  - 核心作用是帮我们把模板编译成 render() 函数
  - 首先判断是否传入 render() 函数，如果没有会判断是否传入 template 选项，如果 template 也没有会把 el 中内容作为模板，再把模板编译成 render() 函数
  - 该方法通过 compileToFunctions() 将模板编译成 render() 函数
  - 最后将 render() 函数存入 options 中 `options.render = render`

- 第二个 \$mount() 在 src\platforms\runtime\index.js 文件

  - 在这个方法中会重新获取 el，因为如果是运行时版本是不会执行这个入口，在这个入口获取 el，运行时版本会在 runtime\index.js \$mount 中重新获取 el

- 接着调用 mountComponent(this, el)，该方法在 src\core\instance\lifecycle.js 中

  - 该方法首先判断是否有 render 选项，如果没有但是传入了模板，并且当前是开发环境的话会发送警告（判断的目的：如果当前使用的是运行时版本的 Vue，而且没有传入 render，而是传入了模板，此时发送一个警告，告诉我们运行时版本不支持编译器）
  - 接着触发 beforeMount 生命周期钩子函数
  - 接着定义 updateComponent，在这个函数调用了 \_render() 和 \_update() 方法

    - vm.\_render() 调用实例化时 Vue 传入的 render() 或者编译 template 生成的 render()，生成虚拟 DOM 返回
    - vm.\_update() 调用 \_\_patch\_\_(vm.\$el, vnode) 把虚拟 DOM 转换为真实 DOM，并且挂载到页面上，最后将真实 DOM 设置到 vm.\$el 中

  - 接着创建 Watcher 实例，创建的时候传入 updateComponent 函数，并在 Watcher 内部调用
    - Watcher 会在里面调用 get() 方法，最后在 get() 方法里面调用 updateComponent() 方法
  - 当 Watcher 实例创建完成后，触发生命周期函数 mounted，挂载完毕返回 Vue 实例 `return vm`

### 2、简述 Vue 响应式原理

- Vue 响应式原理的核心主要是通过 Object.defineProperty 方法来实现的
- 当一个 Vue 实例创建时，vue 会遍历 data 选项的属性，通过 Object.defineProperty 将 data 属性转换成 getter/setter，并且在内部收集依赖，当属性被访问和修改时触发通知重新渲染视图

#### 3、简述虚拟 DOM 中 Key 的作用和好处

- 作用：在 upateChildren 中比较子节点的时候，因为 oldVnode 的子节点的 b,c,d 和 newVnode 的 x,b,c 的 key 相同，所以只做比较，没有更新 DOM 的操作，当遍历完毕后，会再把 x 插入到 DOM 上 DOM 操作只有一次插入操作
- 好处：为了高效的更新虚拟 DOM，其原理是 vue 在 patch 过程中通过 key 可以精准判断两个节点是否是同一个，从而避免频繁更新不同元素，使得整个 patch 过程更加高效，减少 DOM 操作量，提高性能。

#### 4、请简述 Vue 中模板编译的过程。

- 首先通过入口函数 compileToFunctions(template,...) 先从缓存中加载编译好的 render 函数，如果缓存中没有 render 函数，调用 complie(template,options) 开始编译
- 在 compile(template, options) 首先合并 options，然后调用 baseCompile(template, trim(), finalOptions) 编译模板，compile 的核心是合并 options
- 在 baseCompile(template.trim(), finalOptions) 内，通过 parse() 把 template 转换成 AST tree，接着调用 optimize() 对抽象语法树进行优化，标记 AST tree 中的静态 sub trees，静态子树不需要每次重新渲染，patch 的过程中会掉过静态子树，最后调用 generate() 把 AST tree 生成字符串形式的 js 代码
- 当 compile 执行完后，回到入口函数 compileToFunctions ，通过调用 createFunction()，继续把上一步中生成的字符串形式 js 代码转换为函数，当 render 和 staticRenderFns 创建完毕，挂载到 Vue 实例的 options 对应的属性中

# React

标签： React

---

### React优化
1. 渲染数组时加上有意义的key(virtual DOM 优化)
2. 渲染函数中不使用匿名函数(jsx-no-lambda)
3. 使用React.PureComponent 或 @pureComponent 或 shouldComponentUpdate(避免相同props state引发的render)
4. render无关数据不放在state中 可以以class的attribute形式储存
5. props 和 state 的值没有改变的时候不生成新的引用（使用 Array 的方法基本上都会生成新的数组引用）

### 如何废弃componentWillReceiveProps
1.使用props直接渲染，避免复制props到state的行为
2.使用getDerivedStateFromProps方法 (函数式编程思想 **View=f(state)** 的体现)
3.有需要引用到实例this的地方(通常为有副作用)使用componentDidUpdate处理
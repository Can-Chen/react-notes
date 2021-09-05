<!-- 老版本架构（ v15）
分为两层：
Reconciler（协调器）
Renderer（渲染器）

React15架构的缺点：
在Reconciler中，组件初次挂载时会调用mountComponent，更新时会调用updateComponent。这两个方法都会递归更新组件。

官网的解释（https://zh-hans.reactjs.org/docs/implementation-notes.html）。 -->

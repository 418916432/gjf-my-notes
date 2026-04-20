## Doc
### basic
```
next.js是一个基于react的全栈web开发框架

react只是一个UI库，只解决界面怎么画的问题；
Next.js是一个框架，解决应用怎么跑的问题：

// 纯 React 需要做的事：
// 1. 安装 react-router-dom
// 2. 配置路由
// 3. 自己搭 Node 服务器做 SSR
// 4. 配置 webpack
// 5. 处理代码分割
// 6. ... 几十个配置项

// Next.js 只需要：
// 1. 创建文件即可自动路由
// 2. 自带 SSR
// 3. 零配置
```

### next dev
```

```
### route
```
app/page.tsx                    →    /
app/about/page.tsx              →    /about

In Next.js, each folder under `app/` is a URL route. The `page.tsx` inside renders at that URL.
这个规则是Next.js APP Router的核心设置：文件系统及路由
```
### "use client";
```
ext.js 的一个指令（directive）用来告诉 Next.js：这个组件必须在浏览器（客户端）运行，而不是在服务器上。
```
### export default
```
export default function HomePage()
next.js会自动执行这个带有export default的方法，加载当前页面
```
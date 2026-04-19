### Basic
```
传统的web开发：HTML + CSS + JS 三个文件分开
React 开发：JSX 让 HTML 和 JS 写在一起（组件化），它不写html，它写的是JSX，一种看起来像 HTML 的 JavaScript 语法扩展，它会被编译成Java script

React 通过"声明式"和"组件化"，将复杂的 UI 开发变成了清晰、可预测、可复用的组装过程，极大提升了大型前端应用的开发效率和维护性
```
### quick start sample
```
-----不用react
<!-- index.html -->
<div id="app">
  <button onclick="handleClick()">点击我</button>
</div>

<!-- style.css -->
<style>
  .btn { color: red; }
</style>

<!-- script.js -->
<script>
  function handleClick() { alert('clicked'); }
</script>

-----使用react
// Button.jsx - 一个文件包含结构+样式+逻辑
function Button() {
  const handleClick = () => alert('clicked');
  
  return (
    <button style={{ color: 'red' }} onClick={handleClick}>
      点击我
    </button>
  );
}
```
### 声明式（场景：一个计数器，显示数字，奇数红色，偶数蓝色）
#### 原生，你需要操心每一步
```
<div>
  <span id="countDisplay">0</span>
  <button id="incrementBtn">+1</button>
</div>

<script>
  let count = 0;
  const display = document.getElementById('countDisplay');
  const btn = document.getElementById('incrementBtn');
  
  function updateUI() {
    // 😫 每次都要手动告诉浏览器做什么
    display.textContent = count;
    
    // 😫 还要手动处理样式逻辑
    if (count % 2 === 0) {
      display.style.color = 'blue';
    } else {
      display.style.color = 'red';
    }
  }
  
  btn.addEventListener('click', () => {
    count++;           // 😫 改数据
    updateUI();        // 😫 手动调用更新UI（容易忘！）
  });
  
  updateUI(); // 初始化
</script>
```
#### 你只需描述"长什么样"
```
function Counter() {
  const [count, setCount] = useState(0);
  
  // 😊 直接声明：UI 就是 count 的函数
  return (
    <div>
      <span style={{ color: count % 2 === 0 ? 'blue' : 'red' }}>
        {count}
      </span>
      <button onClick={() => setCount(count + 1)}>
        +1
      </button>
    </div>
  );
}
```
### 组件化(一个用户卡片列表，每个卡片有头像、名字、邮箱)
#### 原生 - 代码重复、难以维护
```
<div class="user-list">
  <!-- 用户1 -->
  <div class="card">
    <img src="avatar1.jpg" />
    <h3>张三</h3>
    <p>zhangsan@example.com</p>
  </div>
  
  <!-- 用户2 -->
  <div class="card">
    <img src="avatar2.jpg" />
    <h3>李四</h3>
    <p>lisi@example.com</p>
  </div>
  
  <!-- 用户3、4、5... 要复制粘贴100次？😫 -->
</div>

<script>
  // 如果要动态添加用户，需要手动创建DOM
  function addUser(name, email, avatar) {
    const card = document.createElement('div');
    card.className = 'card';
    card.innerHTML = `
      <img src="${avatar}" />
      <h3>${name}</h3>
      <p>${email}</p>
    `;
    document.querySelector('.user-list').appendChild(card);
  }
  // 但样式、结构、逻辑分散在 HTML 和 JS 中，很难复用
</script>
```
#### 组件化 - 封装、复用、独立
```
// 1. 定义一次组件（封装了结构+样式+逻辑）
function UserCard({ name, email, avatar }) {
  const [isFollowing, setIsFollowing] = useState(false);
  
  return (
    <div className="card">
      <img src={avatar} alt={name} />
      <h3>{name}</h3>
      <p>{email}</p>
      <button onClick={() => setIsFollowing(!isFollowing)}>
        {isFollowing ? '✓ 已关注' : '+ 关注'}
      </button>
    </div>
  );
}

// 2. 到处复用（像搭积木一样）
function UserList() {
  const users = [
    { name: '张三', email: 'zs@example.com', avatar: '1.jpg' },
    { name: '李四', email: 'ls@example.com', avatar: '2.jpg' },
    { name: '王五', email: 'ww@example.com', avatar: '3.jpg' },
  ];
  
  return (
    <div className="user-list">
      {users.map(user => (
        <UserCard {...user} />  // 复用！清晰！
      ))}
    </div>
  );
}
```

### Hook
```
Hook 是你和 React 数据之间的"连接线"，它把数据和操作数据的方法打包成一个JSON对象返回。所以这个对象就是hook。useState 这根线帮你把数据绑在组件上，数据变了页面就自动更新。
Hook是以use开头的function

普通变量
function Counter() {
  let count = 0;  // 普通变量
  
  function handleClick() {
    count = count + 1;
    console.log(count);  // 控制台：1, 2, 3...
    // 但页面上的数字不会变！
  }
  
  return (
    <button onClick={handleClick}>
      点击次数：{count}  {/* 这里永远是 0 */}
    </button>
  );
}

Hook
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);  // ← 这是 Hook
  
  function handleClick() {
    setCount(count + 1);  // 改数字，页面自动更新
  }
  
  return (
    <button onClick={handleClick}>
      点击次数：{count}  {/* 点击后会变：1, 2, 3... */}
    </button>
  );
}

useState 让 React 记住 count 这个值
```
### State Hook
```
State Hook是react提供的一个特殊功能，让函数组件能够记住数据，并且当数据变化时自动更新界面

const [isFollowing, setIsFollowing] = useState(false);
这是 React 的 State Hook，用来让函数组件"记住"数据变化，并自动触发 UI 更新。
isFollowing: 状态变量当前值
setIsFollowing: 更新函数
useState(false): Hook，参数是初始值

```
### useAuth
```
const { isAuthenticated, refreshUser } = useAuth();
//        ↑                    ↑
//     一个变量             一个函数
// 但 useAuth 实际返回了更多，只是你只取了这两个
```
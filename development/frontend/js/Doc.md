## basic
### compare
#### js
```
js是基础

最原始的javascript文件，浏览器可以直接运行，不需要编译

// app.js
function greet(name) {
    return "Hello, " + name;
}

const user = { name: "Alice", age: 25 };
console.log(greet(user.name));
```
#### ts
```
js 加上类型，最终需编译成js，这里多了类型比如string，有类型检查，可以提前发现错误

// utils.ts
function greet(name: string): string {
    return "Hello, " + name;
}

interface User {
    name: string;
    age: number;
}

const user: User = { name: "Alice", age: 25 };
console.log(greet(user.name));
```
#### tsx
```
ts加上react组件语法，，最终需编译成js

// Button.tsx
interface ButtonProps {
    label: string;
    onClick: () => void;
    disabled?: boolean;  // ? 表示可选
}

const Button = ({ label, onClick, disabled = false }: ButtonProps) => {
    return (
        <button 
            onClick={onClick} 
            disabled={disabled}
            className="btn"
        >
            {label}
        </button>
    );
};

export default Button;
```
### const
```
`const` 是 ES6（2015 年发布的 JavaScript 版本 引入的关键字，用来声明常量

// 数组/对象
const users = ['Alice', 'Bob'];
const config = { theme: 'dark' };

const: 不能重新赋值，块级作用于（花括号以内），不能先使用后声明，不能重复声明，必须初始化
let: 可以重新赋值，块级作用于（花括号以内），不能先使用后声明，不能重复声明，不强制初始化值是 undefined
var: 可以重新赋值，函数级，不受块限制，可以先使用后声明（没声明前是undefined），可以重复声明，后面的覆盖前面的，不强制初始化值是 undefined
```
### Export default
```
export default 是 ES6 模块语法，用于把这个文件中的"主要东西"暴露出去，让其他文件可以导入。

例子: 
// file: HomePage.tsx
export default function HomePage() { ... }

// file: 其他文件（比如 app/page.tsx）
import HomePage from './HomePage';  // 导入上面导出的组件
//     ↑ 名字可以随便起，因为它是"默认"导出

export default: 一个文件只能有一个export default，这样导入import X from './file'
export: 但可以有多个export，这样导入import { X } from './file'

例子
// ✅ 一个文件只能有一个 export default
export default function HomePage() { ... }

// ✅ 但可以有多个命名导出
export const API_URL = '/api';
export const MAX_RETRY = 3;
export function helperFunction() { ... }

// 导入时：
import HomePage, { API_URL, MAX_RETRY } from './HomePage';
//      ↑ 默认导入（名字自定）     ↑ 命名导入（必须用原名字）
```
## basic
### js
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
### ts
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
### tsx
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
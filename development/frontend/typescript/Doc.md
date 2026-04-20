## basic
### interface vs class
```
interface: 主要用来定义类型，定义结构
class: 主要用来造实例，写逻辑

```
### interface
```
interface AuthState {
  user: string | null;
  login: (name: string) => void;
}

没有逻辑细节，比如login: (name: string) => void;
永远不会不能创建实例，不能new
它编译完就不见了，它的作用只在你写代码的时候：
- 提醒你变量类型
- 防止你拼写错误
- 防止你传错参数
- 给你代码提示
```
### class
```
class AuthState {
  user: string | null = null;

  login(name: string) {
    this.user = name; // 这里是真实运行的代码
  }
}

// 必须 new 才能用
const auth = new AuthState();
auth.login("张三");

class 里面包含逻辑细节
```

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
### type
```
export type Translations = typeof EN;
create a typescript type called Translations.

Example:
const EN = { welcome: "Hello", button: "Click me", description: "A cool app" };
export type Translations = typeof EN;
automatically become this type:
type Translations = { welcome: string; button: string; description: string; };


export type TranslationKey = keyof typeof EN;
This type is a list of all the KEY NAMES in your EN translation file.

how to use:
import type { Translations } from "@/lib/i18n";

it imports a typescript type definition named Translations.
```
### zod
```
z means zod, it's a validation library from TypeScript.

export type GenerationFormValues = z.infer<typeof generationSchema>;
create a schema from real objects

example:
const generationSchema = z.object({
  prompt: z.string(),
  tone: z.string(),
  length: z.number(),
});

export type GenerationFormValues = z.infer<typeof generationSchema>;

automatically created:
type GenerationFormValues = {
  prompt: string;
  tone: string;
  length: number;
};

```

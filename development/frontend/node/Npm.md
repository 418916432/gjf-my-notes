## Common
### definition
```
NPM = Node Package Manager（Node 包管理器），管理包或者运行命令

Java 用 Maven/Gradle 管理依赖（jar 包）
JavaScript/TypeScript 用 NPM 管理依赖（npm 包）

npm install express     # 安装包
npm uninstall react     # 卸载包
npm update              # 更新包
npm list                # 查看已安装的包

npm run dev             # 运行 dev 脚本
npm run build           # 运行 build 脚本
npm start               # 运行 start 脚本（简写）
npm test                # 运行 test 脚本（简写）
```
### npm install
```
npm install --legacy-peer-deps: 

npm install: 安装 `package.json` 中定义的所有依赖
--legacy-peer-deps: 对于有peer dependency的包，可以安装，但运行时可能出错。
例子：以下安装了16的版本。
# package.json
{
  "dependencies": {
    "react": "^16.14.0",      # React 16
    "react-dom": "^16.14.0",
    "next": "^13.0.0"          # Next.js 13 需要 React 18
  }
}

# npm v7+ 安装时报错：
npm install
# ERESOLVE unable to resolve dependency tree
# next@13.0.0 requires react@^18.2.0
# but react@16.14.0 is installed

# 使用 --legacy-peer-deps 强制安装：
npm install --legacy-peer-deps
# 成功安装，但可能运行时出错

```
### npm run
```
npm run dev: 运行你在 `package.json` 里定义的 `dev` 这个脚本命令
{
  "name": "myapp",
  "scripts": {
    "dev": "next dev",           // ← 定义 dev 命令
    "start": "next start",
    "build": "next build"
  }
}

把你的 React/Next.js 代码编译成 HTML/JS/CSS
启动一个 Node.js 服务器
这个服务器等着浏览器来"取"这些文件
浏览器访问时，服务器把文件发给浏览器

你输入: npm run dev
    ↓
npm 读取: "dev": "next dev --turbopack"
    ↓
npm 执行: next dev --turbopack
    ↓
系统发现 next 是个 Node.js 脚本
    ↓
系统调用: node node_modules/.bin/next dev --turbopack
    ↓
Node.js 运行 next 的代码
    ↓
next 代码里启动了 HTTP 服务器
    ↓
服务器开始运行在 localhost:3000

npm run build: 将源代码（TypeScript/JSX）转换成浏览器可以运行的静态文件（HTML/JS/CSS)
```
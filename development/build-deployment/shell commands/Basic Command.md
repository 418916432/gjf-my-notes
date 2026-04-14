## Common
```
touch: create file

nano: edit file. 保存并退出Ctrl + O → 回车 → Ctrl + X

cp: copy file. cp .env.example .env

export: export IMAGE_TAG=$(date +%Y%m%d-%H%M) 设置环境变量，方便后面这样使用比如: echo $IMAGE_TAG

$(): $(command)执行扩号里的命令。

${RED} 和 $RED: 都可以取到这个变量

(): arr = (a b c) 创建数组

&: 所有输出，1是正常输出，2是错误输出

0: 标准输入

date: 显示或者设置系统时间的命令

%: 格式转义符，类似%d,%s等

+: 在date中的命令，后面跟着的是自定义格式

apt-get: Ubuntu/Debian包管理工具

pip install: python的包管理工具

addgroup: 创建新组的工具
addgroup --system --gid 1001 nodejs 创建系统组，组id是1001，名字是nodejs

useradd: 创建用户的工具
useradd -m -u 1000 appuser  -m创建home目录 -u创建用户的id

chown: change owner

echo -n 'value' | base64  base64 encode

ruff: python代码检查工具，检查代码规范，是否有语法错误等

mypy: python类型检查工具，检查你的 Python 代码是否正确使用了类型注解

>: 输出重定向，覆盖文件

>>: 把内容添加到文件里面，追加

/dev/null: linux里的黑洞

echo -e: enable escape(开启转义)

[[]]: [[ ... ]] Shell 里的条件判断，相当于如果

-z: -z "$PROJECT_ID" 判断这个变量是不是空的

command -v xxx: 检查XXX命令能否找到

|: 把前面的结果传给后面


```
### chown
```
chown -R appuser:appuser /app

chown: change owner（改变文件拥有者）命令
-R: recursive（递归），处理目录下的所有文件和子目录
appuser:appuser: 用户名:组名，将拥有者和组都设为 appuser
app: 要修改的目标目录
```
### grep
```
| grep -q ".": 

grep ".": 
只要有内容就算匹配

-q: 
quiet, 安静模式，不输出任何东西，只返回成功 / 失败

```
### sed
```
sed -i "s/PROJECT_ID/${PROJECT_ID}/g" k8s/service-account.yaml

sed
文本替换工具

-i
直接修改文件本身

s/PROJECT_ID/${PROJECT_ID}/g
把文件的PROJECT_ID替换成这个变量的值PROJECT_ID
```
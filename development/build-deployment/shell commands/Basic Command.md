## Common
```
touch: create file

nano: edit file. 保存并退出Ctrl + O → 回车 → Ctrl + X

cp: copy file. cp .env.example .env

export: export IMAGE_TAG=$(date +%Y%m%d-%H%M) 设置环境变量，方便后面这样使用比如: echo $IMAGE_TAG

$: $(command)执行扩号里的命令。

(): arr = (a b c) 创建数组

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
```
### chown
```
chown -R appuser:appuser /app

chown: change owner（改变文件拥有者）命令
-R: recursive（递归），处理目录下的所有文件和子目录
appuser:appuser: 用户名:组名，将拥有者和组都设为 appuser
app: 要修改的目标目录
```
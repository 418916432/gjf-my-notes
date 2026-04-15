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
### openssl
```
openssl: 是一个开源的命令行工具和加密库，用于处理 SSL/TLS 证书、加密、解密、生成随机数等安全相关的操作

openssl rand -base64 32
生成以base64遍吗的长度为32个字节的随机字符串

```
### tr
```
tr 是替换或者删除命令

tr -d '/=+'
-d 表示删除，会删除/ = +这三种符号

```
### head

```
head: Linux/Unix命令，用于显示文件或数据流的开头部分

head -c 32
-c: bytes，表示取前N个字节

```
### find
```
find k8s/ -name "*.yaml"
在k8s目录下查找所有.yaml文件
```
### while
```
while read -r f; do
  sed -i "s|PROJECT_ID|${PROJECT_ID}|g;s|REGION|${REGION}|g;s|IMAGE_TAG|${IMAGE_TAG}|g" "$f" \
    2>/dev/null || \
  sed -i '' "s|PROJECT_ID|${PROJECT_ID}|g;s|REGION|${REGION}|g;s|IMAGE_TAG|${IMAGE_TAG}|g" "$f"
done

逐行读取，每行存到变量 `f`，变量f存储的是文件路径/名字
```

### for
```
for cmd in gcloud kubectl docker; do
    command -v "$cmd" &>/dev/null || log_error "'$cmd' not found. Please install it."
  done
  
done for循环完成
```
### $
```
./deploy.sh arg1 arg2 arg3
```
### case
```
case语句，根据变量不同的值，执行不同的命令

case "$SERVICE" in
  backend)  deploy_backend ;;   # 如果 SERVICE=backend，执行 deploy_backend
  frontend) deploy_frontend ;;  # 如果 SERVICE=frontend，执行 deploy_frontend
  all)      deploy_backend; deploy_frontend ;;  # 如果 SERVICE=all，执行两个
  *) echo "Unknown service: $SERVICE"; exit 1 ;;  # 其他情况，报错
esac

```
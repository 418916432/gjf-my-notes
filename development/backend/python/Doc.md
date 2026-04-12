## Common
## venv
```
python3.12 -m venv .venv: 创建虚拟环境，隔离环境 
# 项目 A
cd project_a
python3.12 -m venv .venv
source .venv/bin/activate
pip install django==3.2   # 只安装在 project_a/.venv/

# 项目 B
cd project_b
python3.12 -m venv .venv
source .venv/bin/activate
pip install django==4.2   # 只安装在 project_b/.venv/
# 互不干扰！

source .venv/bin/activate: 激活虚拟环境
(.venv) $ deactivate: 推出虚拟环境


```
### install
```
pip install -r requirements-dev.txt 'pydantic[email]'
```
### pytest
```
python的测试框架

# 详细模式（最常用）
pytest tests/ -v
```
### .env
```
python环境变量配置文件
```
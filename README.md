# # Django 学习项目（my）

一个基于 Django 的练手项目，涵盖路由、视图、模板语法、ORM、表单提交、静态文件加载、第三方 API 调用等基础用法。

---

## 项目简介

- 项目名：`my`
- 应用名：`app01`
- 数据库：MySQL（库名 `mysite2`）
- 前端：Bootstrap 3.4.1 + jQuery 3.6.0（本地静态文件）
- 主要演示：
  - 函数视图与 `path` 路由
  - Django 模板语法（变量、循环、字典、列表、对象）
  - `UserInfo` / `Department` 模型的增删查改
  - HTML 表单 + CSRF + POST 提交
  - 调用 Open-Meteo 天气 API 并在页面展示
  - 静态文件加载（`{% static %}`）

---

## 环境要求

| 依赖        | 版本（参考）          |
| ----------- | --------------------- |
| Python      | 3.12                  |
| Django      | 6.1.1（settings 中声明） |
| MySQL       | 5.7 / 8.x             |
| requests    | 最新即可              |
| mysqlclient | 最新即可（或 PyMySQL）|

---

## 快速开始

### 1. 安装依赖

```bash
pip install django requests mysqlclient
如果 mysqlclient 安装失败，可改用 PyMySQL：

bash
pip install pymysql
并在 my/__init__.py 中添加：

python
import pymysql
pymysql.install_as_MySQLdb()
2. 创建数据库
sql
CREATE DATABASE mysite2 DEFAULT CHARACTER SET utf8mb4;
3. 配置数据库
编辑 my/settings.py 中的 DATABASES：

python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'mysite2',
        'USER': 'root',
        'PASSWORD': '123456',
        'HOST': '127.0.0.1',
        'PORT': 3306,
    }
}
4. 执行迁移
bash
python manage.py makemigrations
python manage.py migrate
5. 启动开发服务器
bash
python manage.py runserver
浏览器访问：http://127.0.0.1:8000/

### 目录结构

text
my/
├── manage.py
├── my/                          # 项目配置
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py
│   └── wsgi.py
└── app01/                       # 应用
    ├── admin.py
    ├── apps.py
    ├── models.py
    ├── tests.py
    ├── views.py
    ├── migrations/
    │   ├── 0001_initial.py
    │   ├── 0002_role.py
    │   ├── 0003_department.py
    │   ├── 0004_rename_defg_department_department.py
    │   ├── 0005_delete_role_department_age_alter_userinfo_age.py
    │   └── 0006_rename_department_department_title.py
    └── templates/
        ├── info_add.html
        ├── info_list.html
        ├── login.html
        ├── tpl.html
        ├── user_add.html
        ├── user_list.html
        └── weather.html

static/
├── js/
│   └── jquery-3.6.0.min.js
├── img/
│   └── 1.png
└── plugins/
    └── bootstrap-3.4.1/
        ├── css/
        │   ├── bootstrap.css
        │   ├── bootstrap-theme.css
        │   └── ...
        └── js/
            └── bootstrap.js
### 路由与功能说明

## 路由	视图	说明

/index/	index	返回欢迎字符串 huanyingshiyong
/user/list/	user_list	渲染 user_list.html，演示静态文件加载
/user/add/	user_add	渲染 user_add.html（空模板）
/tpl/	tpl	模板语法练习：变量、列表、字典、对象列表
/weather/	weather	请求 Open-Meteo API，获取成都实时天气
/something/	something	打印请求方法与参数，重定向到百度
/login/	login	GET 显示登录页；POST 校验 root / 123
/orm/	orm	ORM 增删改查演示，返回“成功”
/info/list/	info_list	展示 UserInfo 列表
/info/add/	info_add	GET 显示表单；POST 新增用户并跳转列表
/info/delete/	info_delete	根据 ?nid=id 删除用户并跳转列表
admin/ 路由在 urls.py 中被注释掉了，如需使用请取消注释。

### 数据模型

## UserInfo
字段	类型	说明
name	CharField(32)	姓名
password	CharField(64)	密码
age	IntegerField(default=2)	年龄
Department
字段	类型	说明
title	CharField(16)	部门名称（由 defg → department → title 演变而来）
age	IntegerField(default=2)	备用字段
历史上的 Role 模型已在迁移 0005 中删除，models.py 里也已注释。

静态文件
STATIC_URL = 'static/'

模板中通过 {% load static %} + {% static '...' %} 引用

已内置：jQuery 3.6.0、Bootstrap 3.4.1（CSS/JS）

已知问题与注意事项
weather 视图与模板变量不一致

视图传入的是 {"data": data_list}，而 weather.html 中判断的是 weather_list，页面会显示“数据加载失败”。

建议将视图改为 {"weather_list": data_list}，或修改模板变量名。

user_list.html 中 Bootstrap CSS 引用缺少后缀

当前写法：{% static 'plugins/bootstrap-3.4.1/css/bootstrap' %}

应改为：{% static 'plugins/bootstrap-3.4.1/css/bootstrap.css' %}

info_add.html 表单标签拼写错误

<from> 应为 <form>，否则 CSRF 与提交行为可能异常。

views.py 中 tpl 函数被定义了两次

后定义的会覆盖前者，实际生效的是带 n1 ~ n4 上下文的那一个。

urls.py 中多余的导入

from django.contrib.sitemaps.views import index 未被使用，可删除。

登录逻辑为明文校验

仅用于教学演示，生产环境请使用 Django 内置认证系统并加密存储密码。

数据库密码明文写在 settings.py

生产环境建议使用环境变量或配置文件管理敏感信息。

DEBUG = True 与空 ALLOWED_HOSTS

仅适用于开发环境，部署前请调整。

常用命令速查
bash
# 启动开发服务器
python manage.py runserver

# 生成迁移文件
python manage.py makemigrations

# 应用迁移
python manage.py migrate

# 创建超级用户
python manage.py createsuperuser

# 进入 Django shell
python manage.py shell

## 使用蓝图构建项目

蓝图是 flask 提供的帮助我们隔离不同模块路由的功能。  

### 项目目录结构

```python
'''
app/
    admin/
        __init__.py
        views.py
    home/
        __init__.py
        views.py
    __init__.py
manage.py
'''
```

### 使用蓝图创建路由

```python
# app/home/views.py
from . import home
@home.route("/")
def index():
    return "<h1 style='color: green'>this is home</h1>"
    

# app/home/__init__.py
from flask import Blueprint
home = Blueprint("home", __name__)
import app.home.views
```

### 注册路由

```python
# app/__init__.py
from flask import Flask

app = Flask(__name__)
app.debug = True

from app.home import home as home_blueprint
from app.admin import admin as admin_blueprint

app.register_blueprint(home_blueprint)
app.register_blueprint(admin_blueprint, url_prefix="/admin")
```

### 启动项目

```python
from app import app

if __name__ == "__main__":
    app.run()
```


## 创建表模型

首先需要在虚拟环境中安装 `Flask-SQLAlchemy`  

```python
# app/models.py
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
import datetime # 用于获取当前时间

app = Flask(__name__)
# movie 这个数据库需要手动创建之后才能使用
app.config["SQLALCHEMY_DATABASE_URI"] = "mysql://root:root@127.0.0.1:3306/movie"
app.config["SQLALCHEMY_TRACK_MODIFICATIONS"] = True

db = SQLAlchemy(app)


# 会员数据模型
class User(db.Model):
    __tablename__ = "user"
    id = db.Column(db.Integer, primary_key=True)  # 编号
    name = db.Column(db.String(100), unique=True)  # 昵称
    pwd = db.Column(db.String(100))  # 密码
    email = db.Column(db.String(100), unique=True)  # 邮箱
    phone = db.Column(db.String(11), unique=True)  # 手机
    info = db.Column(db.Text)  # 个性简介
    face = db.Column(db.String(255), unique=True)  # 头像
    addtime = db.Column(db.DateTime, index=True, default=datetime.utcnow)  # 注册时间
    uuid = db.Column(db.String(255), unique=True)  # 唯一标志符
    userlogs = db.relationship('Userlog', backref='User')  # 会员日志外键关系关联

    def __repr__(self):
        return "<User %r>" % self.name


class Userlog(db.Model):
    __tablename__ = "userlog"
    id = db.Column(db.Integer, primary_key=True)  # 编号
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'))  # 所属会员
    ip = db.Column(db.String(100))  # 登陆IP
    addtime = db.Column(db.DateTime, index=True, default=datetime.utcnow)  # 登陆时间

    def __repr__(self):
        return "<Userlog %r>" % self.id
```
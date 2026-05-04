# Flask 教程

Flask 是一个轻量级的Python Web框架，非常适合初学者学习和快速开发。

## 目录
1. [安装Flask](#安装flask)
2. [第一个Flask应用](#第一个flask应用)
3. [路由和视图函数](#路由和视图函数)
4. [模板渲染](#模板渲染)
5. [静态文件](#静态文件)
6. [请求和响应](#请求和响应)
7. [表单处理](#表单处理)
8. [文件上传](#文件上传)
9. [会话管理](#会话管理)
10. [错误处理](#错误处理)

## 安装Flask

```bash
# 使用pip安装Flask
pip install Flask

# 或者创建虚拟环境
python -m venv venv
source venv/bin/activate  # Linux/Mac
# 或
venv\Scripts\activate     # Windows
pip install Flask
```

## 第一个Flask应用

创建一个简单的Flask应用：

```python
# app.py
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello_world():
    return '<h1>Hello, World!</h1>'

if __name__ == '__main__':
    app.run(debug=True)
```

运行应用：
```bash
python app.py
```

访问 http://127.0.0.1:5000 查看结果。

## 路由和视图函数

Flask使用路由装饰器来映射URL到视图函数：

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
    return '这是首页'

@app.route('/about')
def about():
    return '关于我们'

@app.route('/user/<username>')
def show_user_profile(username):
    return f'用户名: {username}'

@app.route('/post/<int:post_id>')
def show_post(post_id):
    return f'文章ID: {post_id}'

@app.route('/path/<path:subpath>')
def show_subpath(subpath):
    return f'子路径: {subpath}'

if __name__ == '__main__':
    app.run(debug=True)
```

## 模板渲染

使用Jinja2模板引擎渲染HTML：

```python
# app.py
from flask import Flask, render_template

app = Flask(__name__)

@app.route('/')
def home():
    return render_template('index.html')

@app.route('/user/<name>')
def user(name):
    return render_template('user.html', name=name)

if __name__ == '__main__':
    app.run(debug=True)
```

创建templates目录并添加模板文件：

```html
<!-- templates/index.html -->
<!DOCTYPE html>
<html>
<head>
    <title>首页</title>
</head>
<body>
    <h1>欢迎来到我的网站！</h1>
    <p>这是一个使用Flask创建的简单网站。</p>
</body>
</html>
```

```html
<!-- templates/user.html -->
<!DOCTYPE html>
<html>
<head>
    <title>用户页面</title>
</head>
<body>
    <h1>你好, {{ name }}!</h1>
    <p>欢迎访问你的个人页面。</p>
</body>
</html>
```

## 静态文件

在static目录中存放CSS、JavaScript、图片等静态文件：

```
your_project/
├── app.py
├── static/
│   ├── css/
│   │   └── style.css
│   ├── js/
│   │   └── script.js
│   └── images/
│       └── logo.png
└── templates/
    └── index.html
```

```python
# app.py
from flask import Flask, render_template

app = Flask(__name__)

@app.route('/')
def home():
    return render_template('index.html')

if __name__ == '__main__':
    app.run(debug=True)
```

```html
<!-- templates/index.html -->
<!DOCTYPE html>
<html>
<head>
    <title>静态文件示例</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}">
</head>
<body>
    <h1>静态文件示例</h1>
    <img src="{{ url_for('static', filename='images/logo.png') }}" alt="Logo">
    <script src="{{ url_for('static', filename='js/script.js') }}"></script>
</body>
</html>
```

```css
/* static/css/style.css */
body {
    font-family: Arial, sans-serif;
    margin: 0;
    padding: 20px;
    background-color: #f0f0f0;
}

h1 {
    color: #333;
}
```

## 请求和响应

处理HTTP请求和响应：

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        if username == 'admin' and password == 'password':
            return jsonify({'status': 'success', 'message': '登录成功'})
        else:
            return jsonify({'status': 'error', 'message': '用户名或密码错误'})
    return '''
        <form method="post">
            用户名: <input type="text" name="username"><br>
            密码: <input type="password" name="password"><br>
            <input type="submit" value="登录">
        </form>
    '''

@app.route('/api/users', methods=['GET'])
def get_users():
    users = [
        {'id': 1, 'name': '张三'},
        {'id': 2, 'name': '李四'},
        {'id': 3, 'name': '王五'}
    ]
    return jsonify(users)

if __name__ == '__main__':
    app.run(debug=True)
```

## 表单处理

处理表单数据：

```python
from flask import Flask, render_template, request, redirect, url_for

app = Flask(__name__)

@app.route('/form')
def form():
    return render_template('form.html')

@app.route('/submit', methods=['POST'])
def submit():
    name = request.form['name']
    email = request.form['email']
    message = request.form['message']
    
    # 这里可以处理表单数据，比如保存到数据库
    print(f"收到表单: 姓名={name}, 邮箱={email}, 消息={message}")
    
    return redirect(url_for('success'))

@app.route('/success')
def success():
    return '表单提交成功！'

if __name__ == '__main__':
    app.run(debug=True)
```

```html
<!-- templates/form.html -->
<!DOCTYPE html>
<html>
<head>
    <title>表单示例</title>
</head>
<body>
    <h1>联系我们</h1>
    <form method="post" action="/submit">
        <label for="name">姓名:</label><br>
        <input type="text" id="name" name="name" required><br><br>
        
        <label for="email">邮箱:</label><br>
        <input type="email" id="email" name="email" required><br><br>
        
        <label for="message">留言:</label><br>
        <textarea id="message" name="message" rows="4" required></textarea><br><br>
        
        <input type="submit" value="提交">
    </form>
</body>
</html>
```

## 文件上传

处理文件上传：

```python
from flask import Flask, render_template, request, redirect, url_for
import os

app = Flask(__name__)
app.config['UPLOAD_FOLDER'] = 'uploads'
app.config['ALLOWED_EXTENSIONS'] = {'txt', 'pdf', 'png', 'jpg', 'jpeg', 'gif'}

def allowed_file(filename):
    return '.' in filename and \
           filename.rsplit('.', 1)[1].lower() in app.config['ALLOWED_EXTENSIONS']

@app.route('/upload')
def upload_form():
    return render_template('upload.html')

@app.route('/upload_file', methods=['POST'])
def upload_file():
    if 'file' not in request.files:
        return '没有文件部分', 400
    
    file = request.files['file']
    
    if file.filename == '':
        return '没有选择文件', 400
    
    if file and allowed_file(file.filename):
        filename = file.filename
        file.save(os.path.join(app.config['UPLOAD_FOLDER'], filename))
        return f'文件 {filename} 上传成功！'
    
    return '不允许的文件类型', 400

if __name__ == '__main__':
    # 确保上传目录存在
    os.makedirs(app.config['UPLOAD_FOLDER'], exist_ok=True)
    app.run(debug=True)
```

```html
<!-- templates/upload.html -->
<!DOCTYPE html>
<html>
<head>
    <title>文件上传</title>
</head>
<body>
    <h1>文件上传</h1>
    <form method="post" enctype="multipart/form-data" action="/upload_file">
        <input type="file" name="file" required><br><br>
        <input type="submit" value="上传">
    </form>
</body>
</html>
```

## 会话管理

使用Flask-Extension进行会话管理：

```bash
pip install Flask-Session
```

```python
from flask import Flask, session, redirect, url_for, request, render_template
from flask_session import Session

app = Flask(__name__)
app.config['SECRET_KEY'] = 'your-secret-key-here'
app.config['SESSION_TYPE'] = 'filesystem'

Session(app)

@app.route('/')
def index():
    if 'username' in session:
        return f'你好, {session["username"]}! <a href="/logout">登出</a>'
    return '<a href="/login">登录</a>'

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form['username']
        session['username'] = username
        return redirect(url_for('index'))
    return '''
        <form method="post">
            用户名: <input type="text" name="username"><br>
            <input type="submit" value="登录">
        </form>
    '''

@app.route('/logout')
def logout():
    session.pop('username', None)
    return redirect(url_for('index'))

if __name__ == '__main__':
    app.run(debug=True)
```

## 错误处理

自定义错误页面：

```python
from flask import Flask, render_template

app = Flask(__name__)

@app.errorhandler(404)
def page_not_found(e):
    return render_template('404.html'), 404

@app.errorhandler(500)
def internal_server_error(e):
    return render_template('500.html'), 500

@app.route('/')
def home():
    return '首页'

@app.route('/trigger_error')
def trigger_error():
    # 这个页面会触发500错误
    1 / 0  # 故意制造错误

if __name__ == '__main__':
    app.run(debug=True)
```

```html
<!-- templates/404.html -->
<!DOCTYPE html>
<html>
<head>
    <title>404 - 页面未找到</title>
</head>
<body>
    <h1>404 - 页面未找到</h1>
    <p>抱歉，您访问的页面不存在。</p>
    <a href="/">返回首页</a>
</body>
</html>
```

```html
<!-- templates/500.html -->
<!DOCTYPE html>
<html>
<head>
    <title>500 - 服务器错误</title>
</head>
<body>
    <h1>500 - 服务器内部错误</h1>
    <p>抱歉，服务器出现了问题。</p>
    <a href="/">返回首页</a>
</body>
</html>
```

## 完整示例：简单的博客应用

```python
# app.py
from flask import Flask, render_template, request, redirect, url_for
import datetime

app = Flask(__name__)

# 模拟数据库
posts = []
next_id = 1

@app.route('/')
def index():
    return render_template('index.html', posts=posts)

@app.route('/post/<int:post_id>')
def post(post_id):
    post = next((p for p in posts if p['id'] == post_id), None)
    if post is None:
        return '文章不存在', 404
    return render_template('post.html', post=post)

@app.route('/new', methods=['GET', 'POST'])
def new_post():
    if request.method == 'POST':
        title = request.form['title']
        content = request.form['content']
        post = {
            'id': next_id,
            'title': title,
            'content': content,
            'date': datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')
        }
        posts.append(post)
        next_id += 1
        return redirect(url_for('index'))
    return render_template('new_post.html')

if __name__ == '__main__':
    app.run(debug=True)
```

```html
<!-- templates/index.html -->
<!DOCTYPE html>
<html>
<head>
    <title>简单博客</title>
</head>
<body>
    <h1>我的博客</h1>
    <a href="{{ url_for('new_post') }}">写新文章</a>
    <hr>
    {% for post in posts %}
        <article>
            <h2><a href="{{ url_for('post', post_id=post.id) }}">{{ post.title }}</a></h2>
            <p>{{ post.date }}</p>
            <p>{{ post.content[:100] }}...</p>
        </article>
        <hr>
    {% else %}
        <p>还没有文章。</p>
    {% endfor %}
</body>
</html>
```

```html
<!-- templates/post.html -->
<!DOCTYPE html>
<html>
<head>
    <title>{{ post.title }}</title>
</head>
<body>
    <h1>{{ post.title }}</h1>
    <p>{{ post.date }}</p>
    <hr>
    <p>{{ post.content }}</p>
    <hr>
    <a href="{{ url_for('index') }}">返回首页</a>
</body>
</html>
```

```html
<!-- templates/new_post.html -->
<!DOCTYPE html>
<html>
<head>
    <title>写新文章</title>
</head>
<body>
    <h1>写新文章</h1>
    <form method="post">
        <label for="title">标题:</label><br>
        <input type="text" id="title" name="title" required><br><br>
        
        <label for="content">内容:</label><br>
        <textarea id="content" name="content" rows="10" required></textarea><br><br>
        
        <input type="submit" value="发布">
    </form>
</body>
</html>
```

## 总结

这个Flask教程涵盖了以下主要内容：

1. **基础概念**：安装、创建应用、运行服务器
2. **路由系统**：不同类型的路由参数和HTTP方法
3. **模板引擎**：使用Jinja2渲染动态HTML
4. **静态文件**：CSS、JavaScript、图片等资源管理
5. **请求处理**：GET、POST请求的处理和响应
6. **表单处理**：用户输入数据的收集和处理
7. **文件上传**：文件上传功能实现
8. **会话管理**：用户登录状态的维护
9. **错误处理**：自定义错误页面
10. **完整示例**：一个简单的博客应用

通过学习这些内容，您应该能够使用Flask创建基本的Web应用程序。建议继续学习Flask的更多高级特性，如数据库集成、用户认证、RESTful API开发等。
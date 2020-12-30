# Flask 介绍

# 第一个 flask 程序

## Hello World

```python
from flask import Flask

app = Flask(__name__)

@app.route("/hello")
def hello():
    return "Hello"

if __name__ == '__main__':
    app.run(debug=True, port=8000)
```

运行这个程序：`python xxx.py`

也可以不在代码中run这个app：

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!'
```

这样的就可以使用 `flask run` 命令运行

这种方式在运行应用之前，需要在终端里导出 `FLASK_APP` 环境变量

还可以通过导出 `FLASK_DEBUG=1` 来单独控制调试模式的开关

## 程序基本结构说明

# DEBUG模式

## 为什么需要DEBUG模式

1. 如果开启了DEBUG模式，那么在代码中如果抛出了异常，在浏览器的页面中可以看到具体的错误信息，以及具体的错误代码位置。方便开发者调试。
2. 如果开启了DEBUG模式，那么以后在`Python`代码中修改了任何代码，只要按`ctrl+s`，`flask`就会自动的重新记载整个网站。不需要手动点击重新运行。

## 配置DEBUG模式的四种方式

1. 在`app.run()`中传递一个参数`debug=True`就可以开启`DEBUG`模式。
2. 给`app.deubg=True`也可以开启`debug`模式。
3. 通过配置参数的形式设置DEBUG模式：`app.config.update(DEBUG=True)`。
4. 通过配置文件的形式设置DEBUG模式：`app.config.from_object(config)`。

## PIN码

如果要在网页上调试代码，会需要输入一个PIN码，在启动的记录会显示

默认会有8小时的缓存时间（存在Cookie中）

# 上下文全局变量

在Flask中有两种上下文：程序上下文 和 请求上下文

![image-20200422050304451](images/image-20200422050304451.png)

Flask 在分发请求之前激活（或推送）程序和请求上下文，请求处理完成后再将其删除。

程序上下文被推送后， 就可以在线程中使用 current_app 和 g 变量。

类似地，请求上下文被推送后，就可以使用 request 和 session 变量。

如果使用这些变量时我们没有激活程序上下文或请求上下文， 就会导致错误。  

# 请求调度

程序收到客户端发来的请求时，要找到处理该请求的视图函数。

为了完成这个任务， Flask会在程序的 URL 映射中查找请求的 URL。 URL 映射是 URL 和视图函数之间的对应关系。

Flask 使用 app.route 修饰器或者非修饰器形式的 app.add_url_rule() 生成映射  

URL 映射中的 HEAD、 Options、 GET 是请求方法，由路由进行处理。

Flask 为每个路由都指定了请求方法， 这样不同的请求方法发送到相同的 URL 上时，会使用不同的视图函数进行处理。

HEAD 和 OPTIONS 方法由 Flask 自动处理  

# 请求钩子

有时在处理请求之前或之后执行代码会很有用。

例如，在请求开始时，我们可能需要创建数据库连接或者认证发起请求的用户。 

为了避免在每个视图函数中都使用重复的代码，Flask 提供了注册通用函数的功能， 

注册的函数可在请求被分发到视图函数之前或之后调用  

Flask支持4种钩子：

- `before_first_request`：注册一个函数，在处理第一个请求之前执行
- `before_request`：注册一个函数，在每次请求之前执行
- `after_request`：注册一个函数，如果没有未处理的异常抛出，在每次请求之后执行
- `teardown_request`：注册一个函数，即使有未处理的异常抛出，也在每次请求之后执行

在请求钩子函数和视图函数之间共享数据一般使用上下文全局变量 `g`。

例如， `before_request` 处理程序可以从数据库中加载已登录用户，并将其保存到 `g.user` 中。

随后调用视图函数时，视图函数再使用 `g.user` 获取用户  

# 路由

## 基本路由写法

```python
@app.route('/')
def hello_world():
    return 'Hello, World!'

@app.route('/index')
def index():
    return 'index'


```

## 路径变量

1. 如果没有指定具体的数据类型，那么默认就是使用`string`数据类型。
2. `int`数据类型只能传递`int`类型。
3. `float`数据类型只能传递`float`类型。
4. `path`数据类型和`string`有点类似，都是可以接收任意的字符串，但是`path`可以接收路径，也就是说可以包含斜杠。
5. `uuid`数据类型只能接收符合`uuid`的字符串。`uuid`是一个全宇宙都唯一的字符串，一般可以用来作为表的主键。
6. `any`数据类型可以在一个`url`中指定多个路径。例如：
    ```python
    @app.route('/<any(blog,article):url_path>/<id>/')
    def detail(url_path,id):
        if url_path == 'blog':
            return '博客详情：%s' % id
        else:
            return '博客详情：%s' % id
    ```

```python
@app.route('/user/<username>')
def show_user_profile(username):
    # show the user profile for that user
    return 'User %s' % escape(username)

@app.route('/post/<int:post_id>')
def show_post(post_id):
    # show the post with the given id, the id is an integer
    return 'Post %d' % post_id

@app.route('/path/<path:subpath>')
def show_subpath(subpath):
    # show the subpath after /path/
    return 'Subpath %s' % escape(subpath)
```

路径变量的转换器主要有以下几种类型：

![image-20200421011017879](images/image-20200421011017879.png)

1. 第一种：使用path的形式（将参数嵌入到路径中），就是上面讲的。
2. 第二种：使用查询字符串的方式，就是通过`?key=value`的形式传递的。
    ```python
    @app.route('/d/')
    def d():
        wd = request.args.get('wd')
        return '您通过查询字符串的方式传递的参数是：%s' % wd
    ```
3. 如果你的这个页面的想要做`SEO`优化，就是被搜索引擎搜索到，那么推荐使用第一种形式（path的形式）。如果不在乎搜索引擎优化，那么就可以使用第二种（查询字符串的形式）。

## any 转换方式

可以指定多种类型 `<any(a,b,c):xxx>`


## 自定义转换方式

```python
class TelephoneConveter(BaseConverter):
    regex = r'1[85734]\d{9}'

app.url_map.converters['tel'] = TelephoneConveter

# 然后就可以使用了
@app.route('/tel/<tel:tel>/')
def print_tel(tel):
    return tel
```

1. 实现一个类，继承自`BaseConverter`。
2. 在自定义的类中，重写`regex`，也就是这个变量的正则表达式。
3. 将自定义的类，映射到`app.url_map.converters`上。比如：
    ```python
    app.url_map.converters['tel'] = TelephoneConverter
    ```

## url_for

这个函数的第一个参数传入一个视图函数名，后面可以传入一堆参数（关键字参数）

每个关键字参数对应路径中的变量

然后可以通过这个函数得到指定函数对应的URL

`url_for`第一个参数，应该是视图函数的名字的字符串。后面的参数就是传递给`url`。

如果传递的参数之前在`url`中已经定义了，那么这个参数就会被当成`path`的形式给 `url`。

如果这个参数之前没有在`url`中定义，那么将变成查询字符串的形式放到`url`中。

```python
@app.route('/post/list/<page>/')
def my_list(page):
    return 'my list'

print(url_for('my_list',page=1,count=2))
# 构建出来的url：/my_list/1/?count=2
```

## 为什么需要 url_for

1. 将来如果修改了`URL`，但没有修改该URL对应的函数名，就不用到处去替换URL了。
2. `url_for`会自动的处理那些特殊的字符，不需要手动去处理。
    ```python
    url = url_for('login',next='/')
    # 会自动的将/编码，不需要手动去处理。
    # url=/login/?next=%2F
    ```



## 静态文件

只要在你的包或模块旁边创建一个名为 `static` 的文件夹就行了。 静态文件位于应用的 `/static` 中。

```python
url_for('static', filename='style.css')
```

这个静态文件在文件系统中的位置应该是 `static/style.css` 。

# 请求

## 获取请求方式

from flask import request

然后在视图函数内部使用 `request.method` 获取请求的方式

# Response 对象

## response 可以返回哪些值

1. 可以返回字符串：返回的字符串其实底层将这个字符串包装成了一个`Response`对象。
2. 可以返回元组：元组的形式是(响应体,状态码,头部信息)，也不一定三个都要写，写两个也是可以的。返回的元组，其实在底层也是包装成了一个`Response`对象。
3. 可以返回`Response`及其子类。

## 自定义一个response对象

1. 继承自`Response`类。
2. 实现方法`force_type(cls,rv,environ=None)`。
3. 指定`app.response_class`为你自定义的`Response`对象。
4. 如果视图函数返回的数据，不是字符串，也不是元组，也不是Response对象，那么就会将返回值传给`force_type`，然后再将`force_type`的返回值返回给前端。



## JSON 格式的 API

如果从视图 返回一个 `dict` ，那么它会被转换为一个 JSON 响应。

```python
@app.route("/me")
def me_api():
    user = get_current_user()
    return {
        "username": user.username,
        "theme": user.theme,
        "image": url_for("user_image", filename=user.image),
    }
```

## 重定向

重定向分为永久性重定向和暂时性重定向，在页面上体现的操作就是浏览器会从一个页面自动跳转到另外一个页面。比如用户访问了一个需要权限的页面，但是该用户当前并没有登录，因此我们应该给他重定向到登录页面。

* 永久性重定向：`http`的状态码是`301`，多用于旧网址被废弃了要转到一个新的网址确保用户的访问，最经典的就是京东网站，你输入`www.jingdong.com`的时候，会被重定向到`www.jd.com`，因为`jingdong.com`这个网址已经被废弃了，被改成`jd.com`，所以这种情况下应该用永久重定向。

* 暂时性重定向：`http`的状态码是`302`，表示页面的暂时性跳转。比如访问一个需要权限的网址，如果当前用户没有登录，应该重定向到登录页面，这种情况下，应该用暂时性重定向。



`flask`中有一个函数叫做`redirect`，可以重定向到指定的页面。示例代码如下：

```python
from flask import Flask,request,redirect,url_for

app = Flask(__name__)

@app.route('/login/')
def login():
    return '这是登录页面'

@app.route('/profile/')
def profile():
    if request.args.get('name'):
        return '个人中心页面'
    else:
        # redirect 重定向
        return redirect(url_for('login'))
```



# session 对象

 `session` 对象，允许你在不同请求 之间储存信息。

这个对象相当于用密钥签名加密的 cookie ，

即用户可以查看你的 cookie ，但是如果没有密钥就无法修改它

```python
from flask import Flask, session, redirect, url_for, escape, request

app = Flask(__name__)

# Set the secret key to some random bytes. Keep this really secret!
app.secret_key = b'_5#y2L"F4Q8z\n\xec]/'

@app.route('/')
def index():
    if 'username' in session:
        return 'Logged in as %s' % escape(session['username'])
    return 'You are not logged in'

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        session['username'] = request.form['username']
        return redirect(url_for('index'))
    return '''
        <form method="post">
            <p><input type=text name=username>
            <p><input type=submit value=Login>
        </form>
    '''

@app.route('/logout')
def logout():
    # remove the username from the session if it's there
    session.pop('username', None)
    return redirect(url_for('index'))
```



# Jinjia2 Template

## 渲染模板

Flask 自动为你配置 [Jinja2](http://jinja.pocoo.org/) 模板引擎

使用 `render_template()` 方法可以渲染模板，你只要提供模板名称和需要 作为参数传递给模板的变量就行了

```python
from flask import render_template

@app.route('/hello/')
@app.route('/hello/<name>')
def hello(name=None):
    return render_template('hello.html', name=name)
```

Flask 会在 `templates` 文件夹内寻找模板。

因此，如果你的应用是一个模块， 那么模板文件夹应该在模块旁边；如果是一个包，那么就应该在包里面：

1. 在使用`render_template`渲染模版的时候，可以传递关键字参数。以后直接在模版中使用就可以了。
2. 如果你的参数过多，那么可以将所有的参数放到一个字典中，然后在传这个字典参数的时候，使用两个星号，将字典打散成关键参数。

## 模板中使用 url_for 获取链接

模版中的`url_for`跟我们后台视图函数中的`url_for`使用起来基本是一模一样的。也是传递视图函数的名字，也可以传递参数。

使用的时候，需要在`url_for`左右两边加上一个`{{ url_for('func') }}`

## 过滤器

### 什么是过滤器

1. 有时候我们想要在模版中对一些变量进行处理，那么就必须需要类似于Python中的函数一样，可以将这个值传到函数中，然后做一些操作。在模版中，过滤器相当于是一个函数，把当前的变量传入到过滤器中，然后过滤器根据自己的功能，再返回相应的值，之后再将结果渲染到页面中。
2. 基本语法：`{{ variable|过滤器名字 }}`。使用管道符号`|`进行组合。

### default 过滤器

使用方式`{{ value|default('默认值') }}`。如果value这个`key`不存在，那么就会使用`default`过滤器提供的默认值。如果你想使用类似于`python`中判断一个值是否为False（例如：None、空字符串、空列表、空字典等），那么就必须要传递另外一个参数`{{ value|default('默认值',boolean=True) }}`。
可以使用`or`来替代`default('默认值',boolean=True)`。例如：`{{ signature or '此人很懒，没有留下任何说明' }}`。

### 自动转义过滤器

1. `safe`过滤器：可以关闭一个字符串的自动转义。
2. `escape`过滤器：对某一个字符串进行转义。
3. `autoescape`标签，可以对他里面的代码块关闭或开启自动转义。
    ```jinja
    {% autoescape off/on %}
        ...代码块
    {% endautoescape %}
    ```

### 常用过滤器

1. first(value)：返回一个序列的第一个元素。names|first。
format(value,*arags,**kwargs)：格式化字符串。例如以下代码：
    ```
    {{ "%s" - "%s"|format('Hello?',"Foo!") }}
    ```
将输出：`Helloo? - Foo!`

2. last(value)：返回一个序列的最后一个元素。示例：names|last。

3. length(value)：返回一个序列或者字典的长度。示例：names|length。

4. join(value,d=u'')：将一个序列用d这个参数的值拼接成字符串。

5. safe(value)：如果开启了全局转义，那么safe过滤器会将变量关掉转义。示例：content_html|safe。

6. int(value)：将值转换为int类型。

7. float(value)：将值转换为float类型。

8. lower(value)：将字符串转换为小写。

9. upper(value)：将字符串转换为小写。

10. replace(value,old,new)： 替换将old替换为new的字符串。

11. truncate(value,length=255,killwords=False)：截取length长度的字符串。

12. striptags(value)：删除字符串中所有的HTML标签，如果出现多个空格，将替换成一个空格。

13. trim：截取字符串前面和后面的空白字符。

14. string(value)：将变量转换成字符串。

15. wordcount(s)：计算一个长字符串中单词的个数。

### 自定义模板过滤器

过滤器本质上就是一个函数。如果在模版中调用这个过滤器，那么就会将这个变量的值作为第一个参数传给过滤器这个函数，然后函数的返回值会作为这个过滤器的返回值。需要使用到一个装饰器：`@app.template_filter('cut')`
```python
@app.template_filter('cut')
def cut(value):
    value = value.replace("hello",'')
    return value
```
```html
<p>{{ article|cut }}</p>
```

## if 判断

`if`条件判断语句必须放在`{% if statement %}`中间，并且还必须有结束的标签`{% endif %}`。

和`python`中的类似，可以使用`>，<，<=，>=，==，!=`来进行判断，也可以通过`and，or，not，()`来进行逻辑合并操作。

## for 

在`jinja2`中的`for`循环，跟`python`中的`for`循环基本上是一模一样的。

也是`for...in...`的形式。并且也可以遍历所有的序列以及迭代器。

但是唯一不同的是，`jinja2`中的`for`循环没有`break`和`continue`语句。

## macro 宏

模板中的宏跟python中的函数类似，可以传递参数，但是不能有返回值，可以将一些经常用到的代码片段放到宏中，然后把一些不固定的值抽取出来当成一个变量。
使用宏的时候，参数可以为默认值。相关示例代码如下：

1. 定义宏：
    ```html
    {% macro input(name, value='', type='text') %}
    <input type="{{ type }}" name="{{ name }}" value="{{
    value }}">
    {% endmacro %}
    ```
2. 使用宏：
    ```html
    <p>{{ input('username') }}</p>
    <p>{{ input('password', type='password') }}</p>
    ```



导入宏：

1. `import "宏文件的路径" as xxx`。
2. `from '宏文件的路径' import 宏的名字 [as xxx]`。
3. 宏文件路径，不要以相对路径去寻找，都要以`templates`作为绝对路径去找。
4. 如果想要在导入宏的时候，就把当前模版的一些参数传给宏所在的模版，那么就应该在导入的时候使用`with context`。示例：`from 'xxx.html' import input with context`。

## include

1. 这个标签相当于是直接将指定的模版中的代码复制粘贴到当前位置。
2. `include`标签，如果想要使用父模版中的变量，直接用就可以了，不需要使用`with context`。
3. `include`的路径，也是跟`import`一样，直接从`templates`根目录下去找，不要以相对路径去找。

## set 

在模版中，可以使用`set`语句来定义变量。示例如下：
```html
{% set username='知了课堂' %}
<p>用户名：{{ username }}</p>
```
一旦定义了这个变量，那么在后面的代码中，都可以使用这个变量，就类似于Python的变量定义是一样的。



## with

`with`语句定义的变量，只能在`with`语句块中使用，超过了这个代码块，就不能再使用了。示例代码如下：

```html
{% with classroom = 'zhiliao1班' %}
<p>班级：{{ classroom }}</p>
{% endwith %}
```
`with`语句也不一定要跟一个变量，可以定义一个空的`with`语句，以后在`with`块中通过`set`定义的变量，就只能在这个`with`块中使用了：
```html
{% with %}
    {% set classroom = 'zhiliao1班' %}
    <p>班级：{{ classroom }}</p>
{% endwith %}
```

## 静态文件

加载静态文件使用的是`url_for`函数。然后第一个参数需要为`static`，第二个参数需要为一个关键字参数`filename='路径'`。示例：
    ```html
    {{ url_for("static",filename='xxx') }}
    ```
路径查找，要以当前项目的`static`目录作为根目录。

## 模板继承

### 为什么需要模版继承：
模版继承可以把一些公用的代码单独抽取出来放到一个父模板中。以后子模板直接继承就可以使用了。这样可以重复性的代码，并且以后修改起来也比较方便。

### 模版继承语法：
使用`extends`语句，来指明继承的父模板。父模板的路径，也是相对于`templates`文件夹下的绝对路径。示例代码如下：
`{% extends "base.html" %}`。

### block语法：
一般在父模版中，定义一些公共的代码。子模板可能要根据具体的需求实现不同的代码。这时候父模版就应该有能力提供一个接口，让父模板来实现。从而实现具体业务需求的功能。
在父模板中：
```html
{% block block的名字 %}
{% endblock %}
```
在子模板中：
```html
{% block block的名字 %}
子模板中的代码
{% endblock %}
```

### 调用父模版代码block中的代码：
默认情况下，子模板如果实现了父模版定义的block。那么子模板block中的代码就会覆盖掉父模板中的代码。如果想要在子模板中仍然保持父模板中的代码，那么可以使用`{{ super() }}`来实现。示例如下：
父模板：
```html
{% block body_block %}
        <p style="background: red;">这是父模板中的代码</p>
    {% endblock %}
```
子模板：
```html
{% block body_block %}
    {{ super() }}
    <p style="background: green;">我是子模板中的代码</p>
{% endblock %}
```

### 调用另外一个block中的代码：
如果想要在另外一个模版中使用其他模版中的代码。那么可以通过`{{ self.其他block名字() }}`就可以了。示例代码如下：
```html
{% block title %}
    知了课堂首页
{% endblock %}

{% block body_block %}
    {{ self.title() }}
    <p style="background: green;">我是子模板中的代码</p>
{% endblock %}
```

### 其他注意事项：
1. 子模板中的代码，第一行，应该是`extends`。
2. 子模板中，如果要实现自己的代码，应该放到block中。如果放到其他地方，那么就不会被渲染。

# 视图高级

## `add_url_rule(rule,endpoint=None,view_func=None)`

这个方法用来添加url与视图函数的映射。如果没有填写`endpoint`，那么默认会使用`view_func`的名字作为`endpoint`。

以后在使用`url_for`的时候，就要看在映射的时候有没有传递`endpoint`参数，

如果传递了，那么就应该使用`endpoint`指定的字符串，如果没有传递，那么就应该使用`view_func`的名字。

## `app.route(rule,**options)`装饰器

这个装饰器底层，其实也是使用`add_url_rule`来实现url与视图函数映射的。

## 类视图

### 标准类视图：
1. 标准类视图，必须继承自`flask.views.View`.
2. 必须实现`dipatch_request`方法，以后请求过来后，都会执行这个方法。这个方法的返回值就相当于是之前的函数视图一样。也必须返回`Response`或者子类的对象，或者是字符串，或者是元组。
3. 必须通过`app.add_url_rule(rule,endpoint,view_func)`来做url与视图的映射。`view_func`这个参数，需要使用类视图下的`as_view`类方法类转换：`ListView.as_view('list')`。
4. 如果指定了`endpoint`，那么在使用`url_for`反转的时候就必须使用`endpoint`指定的那个值。如果没有指定`endpoint`，那么就可以使用`as_view(视图名字)`中指定的视图名字来作为反转。
5. 类视图有以下好处：可以继承，把一些共性的东西抽取出来放到父视图中，子视图直接拿来用就可以了。但是也不是说所有的视图都要使用类视图，这个要根据情况而定。

### 基于请求方法的类视图：
1. 基于方法的类视图，是根据请求的`method`来执行不同的方法的。如果用户是发送的`get`请求，那么将会执行这个类的`get`方法。如果用户发送的是`post`请求，那么将会执行这个类的`post`方法。其他的method类似，比如`delete`、`put`。
2. 这种方式，可以让代码更加简洁。所有和`get`请求相关的代码都放在`get`方法中，所有和`post`请求相关的代码都放在`post`方法中。就不需要跟之前的函数一样，通过`request.method == 'GET'`。

### 类视图中的装饰器：
1. 如果使用的是函数视图，那么自己定义的装饰器必须放在`app.route`下面。否则这个装饰器就起不到任何作用。
2. 类视图的装饰器，需要重写类视图的一个类属性`decorators`，这个类属性是一个列表或者元组都可以，里面装的就是所有的装饰器。

## 蓝图

1. 蓝图的作用就是让我们的Flask项目更加模块化，结构更加清晰。可以将相同模块的视图函数放在同一个蓝图下，同一个文件中，方便管理。
2. 基本语法：
    * 在蓝图文件中导入Blueprint：
        ```python
        from flask import Blueprint
        user_bp = Blueprint('user',__name__)
        ​```。
        ```
    * 在主app文件中注册蓝图：
        ```python
        from blueprints.user import user_bp
        app.regist_blueprint(user_bp)
        ```
3. 如果想要某个蓝图下的所有url都有一个url前缀，那么可以在定义蓝图的时候，指定url_prefix参数：
    ```python
    user_bp = Blueprint('user',__name__,url_prefix='/user/')
    ```
    在定义url_prefix的时候，要注意后面的斜杠，如果给了，那么以后在定义url与视图函数的时候，就不要再在url前面加斜杠了。

4. 蓝图模版文件的查找：
    * 如果项目中的templates文件夹中有相应的模版文件，就直接使用了。
    * 如果项目中的templates文件夹中没有相应的模版文件，那么就到在定义蓝图的时候指定的路径中寻找。并且蓝图中指定的路径可以为相对路径，相对的是当前这个蓝图文件所在的目录。比如：
        ```python
        news_bp = Blueprint('news',__name__,url_prefix='/news',template_folder='zhiliao')
        ```
        因为这个蓝图文件是在blueprints/news.py，那么就会到blueprints这个文件夹下的zhiliao文件夹中寻找模版文件。

5. 蓝图中静态文件的查找规则：
    * 在模版文件中，加载静态文件，如果使用url_for('static')，那么就只会在app指定的静态文件夹目录下查找静态文件。
    * 如果在加载静态文件的时候，指定的蓝图的名字，比如`news.static`，那么就会到这个蓝图指定的static_folder下查找静态文件。

6. url_for反转蓝图中的视图函数为url：
    * 如果使用蓝图，那么以后想要反转蓝图中的视图函数为url，那么就应该在使用url_for的时候指定这个蓝图。比如`news.news_list`。否则就找不到这个endpoint。在模版中的url_for同样也是要满足这个条件，就是指定蓝图的名字。
    * 即使在同一个蓝图中反转视图函数，也要指定蓝图的名字。

## 蓝图实现子域名

1. 使用蓝图技术。
2. 在创建蓝图对象的时候，需要传递一个`subdomain`参数，来指定这个子域名的前缀。例如：`cms_bp = Blueprint('cms',__name__,subdomain='cms')`。
3. 需要在主app文件中，需要配置app.config的SERVER_NAME参数。例如：
    ```python
    app.config['SERVER_NAME'] = 'jd.com:5000'
    ```
    * ip地址不能有子域名。
    * localhost也不能有子域名。
4. 在`C:\Windows\System32\drivers\etc`下，找到hosts文件，然后添加域名与本机的映射。例如：
    ```python
    127.0.0.1   jd.com
    127.0.0.1   cms.jd.com
    ```
    域名和子域名都需要做映射。



# SQLALchemy

## 环境安装

### 安装
```shell
pip install flask-sqlalchemy
```

### 数据库连接
1. 跟sqlalchemy一样，定义好数据库连接字符串DB_URI。
2. 将这个定义好的数据库连接字符串DB_URI，通过`SQLALCHEMY_DATABASE_URI`这个键放到`app.config`中。示例代码：`app.config["SQLALCHEMY_DATABASE_URI"] = DB_URI`.
3. 使用`flask_sqlalchemy.SQLAlchemy`这个类定义一个对象，并将`app`传入进去。示例代码：`db = SQLAlchemy(app)`。

### 创建ORM模型
1. 还是跟使用sqlalchemy一样，定义模型。现在不再是需要使用`delarative_base`来创建一个基类。而是使用`db.Model`来作为基类。
2. 在模型类中，`Column`、`String`、`Integer`以及`relationship`等，都不需要导入了，直接使用`db`下面相应的属性名就可以了。
3. 在定义模型的时候，可以不写`__tablename__`，那么`flask_sqlalchemy`会默认使用当前的模型的名字转换成小写来作为表的名字，并且如果这个模型的名字使用了多个单词并且使用了驼峰命名法，那么会在多个单词之间使用下划线来进行连接。**虽然flask_sqlalchemy给我们提供了这个特性，但是不推荐使用。因为明言胜于暗喻**

### 将ORM模型映射到数据库
1. db.drop_all()
2. db.create_all()

### 使用session
以后session也不需要使用`sessionmaker`来创建了。直接使用`db.session`就可以了。操作这个session的时候就跟之前的`sqlalchemy`的`session`是iyimoyiyang的。

### 查询数据
如果查找数据只是查找一个模型上的数据，那么可以通过`模型.query`的方式进行查找。`query`就跟之前的sqlalchemy中的query方法是一样用的。示例代码如下：
```python
users = User.query.order_by(User.id.desc()).all()
print(users)
```

## 使用SQLAlchemy去连接数据库

使用SQLALchemy去连接数据库，需要使用一些配置信息，然后将他们组合成满足条件的字符串：
```python
HOSTNAME = '127.0.0.1'
PORT = '3306'
DATABASE = 'first_sqlalchemy'
USERNAME = 'root'
PASSWORD = 'root'
```



```
连接的写法：
dialect+driver://username:password@host:port/database

定义：
DB_URI = "mysql+pymysql://{username}:{password}@{host}:{port}/{db}?charset=utf8".format(username=USERNAME,password=PASSWORD,host=HOSTNAME,port=PORT,db=DATABASE)
```

然后使用`create_engine`创建一个引擎`engine`，然后再调用这个引擎的`connect`方法，就可以得到这个对象，然后就可以通过这个对象对数据库进行操作了：
```python
engine = create_engine(DB_URI)

# 判断是否连接成功
conn = engine.connect()
result = conn.execute('select 1')
print(result.fetchone())
```

## ORM 介绍

1. ORM：Object Relationship Mapping
2. 大白话：对象模型与数据库表的映射

### 将ORM模型映射到数据库中
1. 用`declarative_base`根据`engine`创建一个ORM基类。
    ```python
    from sqlalchemy.ext.declarative import declarative_base
    engine = create_engine(DB_URI)
    Base = declarative_base(engine)
    ```
2. 用这个`Base`类作为基类来写自己的ORM类。要定义`__tablename__`类属性，来指定这个模型映射到数据库中的表名。
    ```python
    class Person(Base):
        __tablename__ = 'person'
    ```
3. 创建属性来映射到表中的字段，所有需要映射到表中的属性都应该为Column类型：
    ```python
    class Person(Base):
        __tablename__ = 'person'
        # 2. 在这个ORM模型中创建一些属性，来跟表中的字段进行一一映射。这些属性必须是sqlalchemy给我们提供好的数据类型。
        id = Column(Integer,primary_key=True,autoincrement=True)
        name = Column(String(50))
        age = Column(Integer)
    ```
4. 使用`Base.metadata.create_all()`来将模型映射到数据库中。
5. 一旦使用`Base.metadata.create_all()`将模型映射到数据库中后，即使改变了模型的字段，也不会重新映射了。

### 用session做数据的增删改查操作
1. 构建session对象：所有和数据库的ORM操作必须通过一个叫做`session`的会话对象来实现，通过以下代码来获取会话对象：
    ```python
    from sqlalchemy.orm import sessionmaker

    engine = create_engine(DB_URI)
    session = sessionmaker(engine)()
    ```
2. 添加对象：
    * 创建对象，也即创建一条数据：
        ```python
        p = Person(name='zhiliao',age=18,country='china')
        ```
    * 将这个对象添加到`session`会话对象中：
        ```python
        session.add(p)
        ```
    * 将session中的对象做commit操作（提交）：
        ```python
        session.commit()
        ```
    * 一次性添加多条数据：
        ```python
        p1 = Person(name='zhiliao1',age=19,country='china')
        p2 = Person(name='zhiliao2',age=20,country='china')
        session.add_all([p1,p2])
        session.commit()
        ```
3. 查找对象：
    ```python
    # 查找某个模型对应的那个表中所有的数据：
    all_person = session.query(Person).all()
    # 使用filter_by来做条件查询
    all_person = session.query(Person).filter_by(name='zhiliao').all()
    # 使用filter来做条件查询
    all_person = session.query(Person).filter(Person.name=='zhiliao').all()
    # 使用get方法查找数据，get方法是根据id来查找的，只会返回一条数据或者None
    person = session.query(Person).get(primary_key)
    # 使用first方法获取结果集中的第一条数据
    person = session.query(Person).first()
    ```
4. 修改对象：首先从数据库中查找对象，然后将这条数据修改为你想要的数据，最后做commit操作就可以修改数据了。
    ```python
    person = session.query(Person).first()
    person.name = 'ketang'
    session.commit()
    ```
5. 删除对象：将需要删除的数据从数据库中查找出来，然后使用`session.delete`方法将这条数据从session中删除，最后做commit操作就可以了。
    ```python
    person = session.query(Person).first()
    session.delete(person)
    session.commit()
    ```



## 常用数据类型

1. Integer：整形，映射到数据库中是int类型。
2. Float：浮点类型，映射到数据库中是float类型。他占据的32位。
3. Double：双精度浮点类型，映射到数据库中是double类型，占据64位。
4. String：可变字符类型，映射到数据库中是varchar类型.
5. Boolean：布尔类型，映射到数据库中的是tinyint类型。
6. DECIMAL：定点类型。是专门为了解决浮点类型精度丢失的问题的。在存储钱相关的字段的时候建议大家都使用这个数据类型。并且这个类型使用的时候需要传递两个参数，第一个参数是用来标记这个字段总能能存储多少个数字，第二个参数表示小数点后有多少位。
7. Enum：枚举类型。指定某个字段只能是枚举中指定的几个值，不能为其他值。在ORM模型中，使用Enum来作为枚举，示例代码如下：
    ```python
    class Article(Base):
        __tablename__ = 'article'
        id = Column(Integer,primary_key=True,autoincrement=True)
        tag = Column(Enum("python",'flask','django'))
    ```
    在Python3中，已经内置了enum这个枚举的模块，我们也可以使用这个模块去定义相关的字段。示例代码如下：
    ```python
    class TagEnum(enum.Enum):
        python = "python"
        flask = "flask"
        django = "django"

    class Article(Base):
        __tablename__ = 'article'
        id = Column(Integer,primary_key=True,autoincrement=True)
        tag = Column(Enum(TagEnum))

    article = Article(tag=TagEnum.flask)
    ```
8. Date：存储时间，只能存储年月日。映射到数据库中是date类型。在Python代码中，可以使用`datetime.date`来指定。示例代码如下：
    ```python
    class Article(Base):
        __tablename__ = 'article'
        id = Column(Integer,primary_key=True,autoincrement=True)
        create_time = Column(Date)

    article = Article(create_time=date(2017,10,10))
    ```
9. DateTime：存储时间，可以存储年月日时分秒毫秒等。映射到数据库中也是datetime类型。在Python代码中，可以使用`datetime.datetime`来指定。示例代码如下：
    ```python
    class Article(Base):
        __tablename__ = 'article'
        id = Column(Integer,primary_key=True,autoincrement=True)
        create_time = Column(DateTime)

    article = Article(create_time=datetime(2011,11,11,11,11,11))
    ```
10. Time：存储时间，可以存储时分秒。映射到数据库中也是time类型。在Python代码中，可以使用`datetime.time`来至此那个。示例代码如下：
    ```python
    class Article(Base):
        __tablename__ = 'article'
        id = Column(Integer,primary_key=True,autoincrement=True)
        create_time = Column(Time)

    article = Article(create_time=time(hour=11,minute=11,second=11))
    ```
11. Text：存储长字符串。一般可以存储6W多个字符。如果超出了这个范围，可以使用LONGTEXT类型。映射到数据库中就是text类型。
12. LONGTEXT：长文本类型，映射到数据库中是longtext类型。

## Column 常用参数

1. primary_key：设置某个字段为主键。
2. autoincrement：设置这个字段为自动增长的。
3. default：设置某个字段的默认值。在发表时间这些字段上面经常用。
4. nullable：指定某个字段是否为空。默认值是True，就是可以为空。
5. unique：指定某个字段的值是否唯一。默认是False。
6. onupdate：在数据更新的时候会调用这个参数指定的值或者函数。在第一次插入这条数据的时候，不会用onupdate的值，只会使用default的值。常用的就是`update_time`（每次更新数据的时候都要更新的值）。
7. name：指定ORM模型中某个属性映射到表中的字段名。如果不指定，那么会使用这个属性的名字来作为字段名。如果指定了，就会使用指定的这个值作为参数。这个参数也可以当作位置参数，在第1个参数来指定。
    ```python
    title = Column(String(50),name='title',nullable=False)
    title = Column('my_title',String(50),nullable=False)
    ```

## query 可用参数

1. 模型对象。指定查找这个模型中所有的对象。
2. 模型中的属性。可以指定只查找某个模型的其中几个属性。
3. 聚合函数。
    * func.count：统计行的数量。
    * func.avg：求平均值。
    * func.max：求最大值。
    * func.min：求最小值。
    * func.sum：求和。
    `func`上，其实没有任何聚合函数。但是因为他底层做了一些魔术，只要mysql中有的聚合函数，都可以通过func调用。

## filter 过滤条件

过滤是数据提取的一个很重要的功能，以下对一些常用的过滤条件进行解释，并且这些过滤条件都是只能通过filter方法实现的：
1. equals：
    ```python
    article = session.query(Article).filter(Article.title == "title0").first()
    print(article)
    ```
2. not equals:
    ```python
    query.filter(User.name != 'ed')
    ```
2. like：
    ```python
    query.filter(User.name.like('%ed%'))
    ```

3. in：
    ```python
    query.filter(User.name.in_(['ed','wendy','jack']))
    # 同时，in也可以作用于一个Query
    query.filter(User.name.in_(session.query(User.name).filter(User.name.like('%ed%'))))
    ```

4. not in：
    ```python
    query.filter(~User.name.in_(['ed','wendy','jack']))
    ```
5.  is null：
    ```python
    query.filter(User.name==None)
    # 或者是
    query.filter(User.name.is_(None))
    ```

6. is not null:
    ```python
    query.filter(User.name != None)
    # 或者是
    query.filter(User.name.isnot(None))
    ```

7. and：
    ```python
    from sqlalchemy import and_
    query.filter(and_(User.name=='ed',User.fullname=='Ed Jones'))
    # 或者是传递多个参数
    query.filter(User.name=='ed',User.fullname=='Ed Jones')
    # 或者是通过多次filter操作
    query.filter(User.name=='ed').filter(User.fullname=='Ed Jones')
    ```

8. or：
    ```python
    from sqlalchemy import or_  query.filter(or_(User.name=='ed',User.name=='wendy'))
    ```

如果想要查看orm底层转换的sql语句，可以在filter方法后面不要再执行任何方法直接打印就可以看到了。比如：
    ```python
        articles = session.query(Article).filter(or_(Article.title=='abc',Article.content=='abc'))
        print(articles)
    ```

## 外键

使用SQLAlchemy创建外键非常简单。在从表中增加一个字段，指定这个字段外键的是哪个表的哪个字段就可以了。从表中外键的字段，必须和父表的主键字段类型保持一致。
示例代码如下：
```python
class User(Base):
    __tablename__ = 'user'
    id = Column(Integer,primary_key=True,autoincrement=True)
    username = Column(String(50),nullable=False)

class Article(Base):
    __tablename__ = 'article'
    id = Column(Integer,primary_key=True,autoincrement=True)
    title = Column(String(50),nullable=False)
    content = Column(Text,nullable=False)

    uid = Column(Integer,ForeignKey("user.id"))
```
外键约束有以下几项： 
1. RESTRICT：父表数据被删除，会阻止删除。默认就是这一项。 
2. NO ACTION：在MySQL中，同RESTRICT。 
3. 3. CASCADE：级联删除。 
4. 4. SET NULL：父表数据被删除，子表数据会设置为NULL。

## ORM的关系及一对多

mysql级别的外键，还不够ORM，必须拿到一个表的外键，然后通过这个外键再去另外一张表中查找，这样太麻烦了。SQLAlchemy提供了一个`relationship`，这个类可以定义属性，以后在访问相关联的表的时候就直接可以通过属性访问的方式就可以访问得到了。示例代码：
```python
class User(Base):
    __tablename__ = 'user'
    id = Column(Integer,primary_key=True,autoincrement=True)
    username = Column(String(50),nullable=False)

    # articles = relationship("Article")

    def __repr__(self):
        return "<User(username:%s)>" % self.username

class Article(Base):
    __tablename__ = 'article'
    id = Column(Integer,primary_key=True,autoincrement=True)
    title = Column(String(50),nullable=False)
    content = Column(Text,nullable=False)
    uid = Column(Integer,ForeignKey("user.id"))

    author = relationship("User",backref="articles")
```
另外，可以通过`backref`来指定反向访问的属性名称。articles是有多个。他们之间的关系是一个一对多的关系。

## 一对一

在sqlalchemy中，如果想要将两个模型映射成一对一的关系，那么应该在父模型中，指定引用的时候，要传递一个`uselist=False`这个参数进去。就是告诉父模型，以后引用这个从模型的时候，不再是一个列表了，而是一个对象了。示例代码如下：
```python
class User(Base):
    __tablename__ = 'user'
    id = Column(Integer,primary_key=True,autoincrement=True)
    username = Column(String(50),nullable=False)

    extend = relationship("UserExtend",uselist=False)

    def __repr__(self):
        return "<User(username:%s)>" % self.username

class UserExtend(Base):
    __tablename__ = 'user_extend'
    id = Column(Integer, primary_key=True, autoincrement=True)
    school = Column(String(50))
    uid = Column(Integer,ForeignKey("user.id"))

    user = relationship("User",backref="extend")
```
当然，也可以借助`sqlalchemy.orm.backref`来简化代码：
```python
class User(Base):
    __tablename__ = 'user'
    id = Column(Integer,primary_key=True,autoincrement=True)
    username = Column(String(50),nullable=False)

    # extend = relationship("UserExtend",uselist=False)

    def __repr__(self):
        return "<User(username:%s)>" % self.username

class UserExtend(Base):
    __tablename__ = 'user_extend'
    id = Column(Integer, primary_key=True, autoincrement=True)
    school = Column(String(50))
    uid = Column(Integer,ForeignKey("user.id"))

    user = relationship("User",backref=backref("extend",uselist=False))
```

## 多对多

1. 多对多的关系需要通过一张中间表来绑定他们之间的关系。
2. 先把两个需要做多对多的模型定义出来
3. 使用Table定义一个中间表，中间表一般就是包含两个模型的外键字段就可以了，并且让他们两个来作为一个“复合主键”。
4. 在两个需要做多对多的模型中随便选择一个模型，定义一个relationship属性，来绑定三者之间的关系，在使用relationship的时候，需要传入一个secondary=中间表。

## ORM 层面删除

ORM层面删除数据，会无视mysql级别的外键约束。直接会将对应的数据删除，然后将从表中的那个外键设置为NULL。如果想要避免这种行为，应该将从表中的外键的`nullable=False`。
在SQLAlchemy，只要将一个数据添加到session中，和他相关联的数据都可以一起存入到数据库中了。这些是怎么设置的呢？其实是通过relationship的时候，有一个关键字参数cascade可以设置这些属性： 
1. save-update：默认选项。在添加一条数据的时候，会把其他和他相关联的数据都添加到数据库中。这种行为就是save-update属性影响的。 
2. delete：表示当删除某一个模型中的数据的时候，是否也删掉使用relationship和他关联的数据。
3. delete-orphan：表示当对一个ORM对象解除了父表中的关联对象的时候，自己便会被删除掉。当然如果父表中的数据被删除，自己也会被删除。这个选项只能用在一对多上，不能用在多对多以及多对一上。并且还需要在子模型中的relationship中，增加一个single_parent=True的参数。 
4. merge：默认选项。当在使用session.merge，合并一个对象的时候，会将使用了relationship相关联的对象也进行merge操作。 
5. expunge：移除操作的时候，会将相关联的对象也进行移除。这个操作只是从session中移除，并不会真正的从数据库中删除。 
6. all：是对save-update, merge, refresh-expire, expunge, delete几种的缩写。

## 排序

1. order_by：可以指定根据这个表中的某个字段进行排序，如果在前面加了一个-，代表的是降序排序。
2. 在模型定义的时候指定默认排序：有些时候，不想每次在查询的时候都指定排序的方式，可以在定义模型的时候就指定排序的方式。有以下两种方式：
    * relationship的order_by参数：在指定relationship的时候，传递order_by参数来指定排序的字段。
    * 在模型定义中，添加以下代码：

     __mapper_args__ = {
         "order_by": title
       }
    即可让文章使用标题来进行排序。
3. 正序排序与倒序排序：默认是使用正序排序。如果需要使用倒序排序，那么可以使用这个字段的`desc()`方法，或者是在排序的时候使用这个字段的字符串名字，然后在前面加一个负号。

## limit、offset和切片操作

1. limit：可以限制每次查询的时候只查询几条数据。
2. offset：可以限制查找数据的时候过滤掉前面多少条。
3. 切片：可以对Query对象使用切片操作，来获取想要的数据。可以使用`slice(start,stop)`方法来做切片操作。也可以使用`[start:stop]`的方式来进行切片操作。一般在实际开发中，中括号的形式是用得比较多的。希望大家一定要掌握。示例代码如下：
```python
articles = session.query(Article).order_by(Article.id.desc())[0:10]
```

## 懒加载

在一对多，或者多对多的时候，如果想要获取多的这一部分的数据的时候，往往能通过一个属性就可以全部获取了。比如有一个作者，想要或者这个作者的所有文章，那么可以通过user.articles就可以获取所有的。但有时候我们不想获取所有的数据，比如只想获取这个作者今天发表的文章，那么这时候我们可以给relationship传递一个lazy='dynamic'，以后通过user.articles获取到的就不是一个列表，而是一个AppenderQuery对象了。这样就可以对这个对象再进行一层过滤和排序等操作。
通过`lazy='dynamic'`，获取出来的多的那一部分的数据，就是一个`AppenderQuery`对象了。这种对象既可以添加新数据，也可以跟`Query`一样，可以再进行一层过滤。
总而言之一句话：如果你在获取数据的时候，想要对多的那一边的数据再进行一层过滤，那么这时候就可以考虑使用`lazy='dynamic'`。
lazy可用的选项：

1. `select`：这个是默认选项。还是拿`user.articles`的例子来讲。如果你没有访问`user.articles`这个属性，那么sqlalchemy就不会从数据库中查找文章。一旦你访问了这个属性，那么sqlalchemy就会立马从数据库中查找所有的文章，并把查找出来的数据组装成一个列表返回。这也是懒加载。
2. `dynamic`：这个就是我们刚刚讲的。就是在访问`user.articles`的时候返回回来的不是一个列表，而是`AppenderQuery`对象。

## group by

根据某个字段进行分组。比如想要根据性别进行分组，来统计每个分组分别有多少人，那么可以使用以下代码来完成：
```python
session.query(User.gender,func.count(User.id)).group_by(User.gender).all()
```

## having

having是对查找结果进一步过滤。比如只想要看未成年人的数量，那么可以首先对年龄进行分组统计人数，然后再对分组进行having过滤。示例代码如下：
```python
result = session.query(User.age,func.count(User.id)).group_by(User.age).having(User.age >= 18).all()
```

## join

1. join分为left join（左外连接）和right join（右外连接）以及内连接（等值连接）。
2. 参考的网页：http://www.jb51.net/article/15386.htm
3. 在sqlalchemy中，使用join来完成内连接。在写join的时候，如果不写join的条件，那么默认将使用外键来作为条件连接。
4. query查找出来什么值，不会取决于join后面的东西，而是取决于query方法中传了什么参数。就跟原生sql中的select 后面那一个一样。
比如现在要实现一个功能，要查找所有用户，按照发表文章的数量来进行排序。示例代码如下：
```python
result = session.query(User,func.count(Article.id)).join(Article).group_by(User.id).order_by(func.count(Article.id).desc()).all()
```

## subquery

子查询可以让多个查询变成一个查询，只要查找一次数据库，性能相对来讲更加高效一点。不用写多个sql语句就可以实现一些复杂的查询。那么在sqlalchemy中，要实现一个子查询，应该使用以下几个步骤：
1. 将子查询按照传统的方式写好查询代码，然后在`query`对象后面执行`subquery`方法，将这个查询变成一个子查询。
2. 在子查询中，将以后需要用到的字段通过`label`方法，取个别名。
3. 在父查询中，如果想要使用子查询的字段，那么可以通过子查询的返回值上的`c`属性拿到。
整体的示例代码如下：
```python
stmt = session.query(User.city.label("city"),User.age.label("age")).filter(User.username=='李A').subquery()
result = session.query(User).filter(User.city==stmt.c.city,User.age==stmt.c.age).all()
```



## alembic

使用alembic的步骤：
1. 定义好自己的模型。
2. 使用alembic创建一个仓库：`alembic init [仓库的名字，推荐使用alembic]`。
3. 修改配置文件：
    * 在`alembic.ini`中，给`sqlalchemy.url`设置数据库的连接方式。这个连接方式跟sqlalchemy的方式一样的。
    * 在`alembic/env.py`中的`target_metadata`设置模型的`Base.metadata`。但是要导入`models`，需要将models所在的路径添加到这个文件中。示例代码如下：
        ```python
        import sys,os
        sys.path.append(os.path.dirname(os.path.dirname(__file__)))
        ```
4. 将ORM模型生成迁移脚本：`alembic revision --autogenerate -m 'message'`。
5. 将生成的脚本映射到数据库中：`alembic upgrade head`。
6. 以后如果修改了模型，重复4、5步骤。
7. 注意事项：在终端中，如果想要使用alembic，则需要首先进入到安装了alembic的虚拟环境中，不然就找不到这个命令。

### 常用命令：
1. init：创建一个alembic仓库。
2. revision：创建一个新的版本文件。
3. --autogenerate：自动将当前模型的修改，生成迁移脚本。
4. -m：本次迁移做了哪些修改，用户可以指定这个参数，方便回顾。
5. upgrade：将指定版本的迁移文件映射到数据库中，会执行版本文件中的upgrade函数。如果有多个迁移脚本没有被映射到数据库中，那么会执行多个迁移脚本。
6. [head]：代表最新的迁移脚本的版本号。
7. downgrade：会执行指定版本的迁移文件中的downgrade函数。
8. heads：展示head指向的脚本文件版本号。
9. history：列出所有的迁移版本及其信息。
10. current：展示当前数据库中的版本号。

### 经典错误：
1. FAILED: Target database is not up to date.
    * 原因：主要是heads和current不相同。current落后于heads的版本。
    * 解决办法：将current移动到head上。alembic upgrade head
2. FAILED: Can't locate revision identified by '77525ee61b5b'
    * 原因：数据库中存的版本号不在迁移脚本文件中
    * 解决办法：删除数据库的alembic_version表中的数据，重新执行alembic upgrade head
3. 执行`upgrade head`时报某个表已经存在的错误：
    * 原因：执行这个命令的时候，会执行所有的迁移脚本，因为数据库中已经存在了这个表。然后迁移脚本中又包含了创建表的代码。
    * 解决办法：（1）删除versions中所有的迁移文件。（2）修改迁移脚本中创建表的代码。

# Flask SQLAlchemy

## 安装：
```shell
pip install flask-sqlalchemy
```

## 数据库连接
1. 跟sqlalchemy一样，定义好数据库连接字符串DB_URI。
2. 将这个定义好的数据库连接字符串DB_URI，通过`SQLALCHEMY_DATABASE_URI`这个键放到`app.config`中。示例代码：`app.config["SQLALCHEMY_DATABASE_URI"] = DB_URI`.
3. 使用`flask_sqlalchemy.SQLAlchemy`这个类定义一个对象，并将`app`传入进去。示例代码：`db = SQLAlchemy(app)`。

## 创建ORM模型
1. 还是跟使用sqlalchemy一样，定义模型。现在不再是需要使用`delarative_base`来创建一个基类。而是使用`db.Model`来作为基类。
2. 在模型类中，`Column`、`String`、`Integer`以及`relationship`等，都不需要导入了，直接使用`db`下面相应的属性名就可以了。
3. 在定义模型的时候，可以不写`__tablename__`，那么`flask_sqlalchemy`会默认使用当前的模型的名字转换成小写来作为表的名字，并且如果这个模型的名字使用了多个单词并且使用了驼峰命名法，那么会在多个单词之间使用下划线来进行连接。**虽然flask_sqlalchemy给我们提供了这个特性，但是不推荐使用。因为明言胜于暗喻**

## 将ORM模型映射到数据库
1. db.drop_all()
2. db.create_all()

## 使用session
以后session也不需要使用`sessionmaker`来创建了。直接使用`db.session`就可以了。操作这个session的时候就跟之前的`sqlalchemy`的`session`是iyimoyiyang的。

## 查询数据
如果查找数据只是查找一个模型上的数据，那么可以通过`模型.query`的方式进行查找。`query`就跟之前的sqlalchemy中的query方法是一样用的。示例代码如下：
```python
users = User.query.order_by(User.id.desc()).all()
print(users)
```

# Flask_Script

Flask-Script的作用是可以通过命令行的形式来操作Flask。例如通过命令跑一个开发版本的服务器、设置数据库，定时任务等。

要使用Flask-Script，可以通过`pip install flask-script`安装最新版本。

## 命令的添加方式

1. 使用`manage.commad`：这个方法是用来添加那些不需要传递参数的命令。示例代码如下：
    ```python
    manager = Manager(app)
    manager.add_command("db",db_manager)

    @manager.command
    def greet():
        print('你好')
    ```
2. `使用manage.option`：这个方法是用来添加那些需要传递参数的命令。有几个参数就需要写几个`option`。示例代码如下：
    ```python
    @manager.option("-u","--username",dest="username")
    @manager.option("-e","--email",dest="email")
    def add_user(username,email):
        user = BackendUser(username=username,email=email)
        db.session.add(user)
        db.session.commit()
    ```

3. 如果有一些命令是针对某个功能的。比如有一堆命令是针对ORM与表映射的，那么可以将这些命令单独放在一个文件中方便管理。也是使用`Manager`的对象来添加。然后到主manage文件中，通过`manager.add_command`来添加。示例代码如下：
db_script.py
```python
from flask_script import Manager

db_manager = Manager()

@db_manager.command
def init():
    print('迁移仓库创建完毕！')

@db_manager.command
def revision():
    print('迁移脚本生成成功！')

@db_manager.command
def upgrade():
    print('脚本映射到数据库成功！')
```

manage.py
```python
from db_script import db_manager

manager = Manager(app)
manager.add_command("db",db_manager)
```

# migrate

在实际的开发环境中，经常会发生数据库修改的行为。

一般我们修改数据库不会直接手动的去修改，而是去修改ORM对应的模型，然后再把模型映射到数据库中。

这时候如果有一个工具能专门做这种事情，就显得非常有用了，而flask-migrate就是做这个事情的。

flask-migrate是基于Alembic进行的一个封装，并集成到Flask中，而所有的迁移操作其实都是Alembic做的，他能跟踪模型的变化，并将变化映射到数据库中。

## 安装

```python
pip install flask-migrate
```

## manage.py

```python
from flask_script import Manager
from zhiliao import app
from exts import db
from flask_migrate import Migrate,MigrateCommand

manager = Manager(app)

# 用来绑定app和db到flask_migrate的
Migrate(app,db)
# 添加Migrate的所有子命令到db下
manager.add_command("db",MigrateCommand)


if __name__ == '__main__':
    manager.run()
```

## 常用命令

1. 初始化一个环境：`python manage.py db init`
2. 自动检测模型，生成迁移脚本：`python manage.py db migrate`
3. 将迁移脚本映射到数据库中：`python manage.py db upgrade`
4. 更多命令：`python manage.py db --help`

# WTForms

这个库一般有两个作用。第一个就是做表单验证，把用户提交上来的数据进行验证是否合法。第二个就是做模版渲染。

## 表单验证

1. 自定义一个表单类，继承自wtforms.Form类。
2. 定义好需要验证的字段，字段的名字必须和模版中那些需要验证的input标签的name属性值保持一致。
3. 在需要验证的字段上，需要指定好具体的数据类型。
4. 在相关的字段上，指定验证器。
5. 以后在视图中，就只需要使用这个表单类的对象，并且把需要验证的数据，也就是request.form传给这个表单类，以后调用form.validate()方法，如果返回True，那么代表用户输入的数据都是合法的，否则代表用户输入的数据是有问题的。如果验证失败了，那么可以通过form.errors来获取具体的错误信息。
示例代码如下：
ReistForm类的代码：
```python
class RegistForm(Form):
    username = StringField(validators=[Length(min=3,max=10,message='用户名长度必须在3到10位之间')])
    password = StringField(validators=[Length(min=6,max=10)])
    password_repeat = StringField(validators=[Length(min=6,max=10),EqualTo("password")])
```
视图函数中的代码：
```python
form = RegistForm(request.form)
if form.validate():
    return "success"
else:
    print(form.errors)
    return "fail"
```

## 常用的验证器

数据发送过来，经过表单验证，因此需要验证器来进行验证，以下对一些常用的内置验证器进行讲解：
1. Email：验证上传的数据是否为邮箱。
2. EqualTo：验证上传的数据是否和另外一个字段相等，常用的就是密码和确认密码两个字段是否相等。
3. InputRequir：原始数据的需要验证。如果不是特殊情况，应该使用InputRequired。
3. Length：长度限制，有min和max两个值进行限制。
4. NumberRange：数字的区间，有min和max两个值限制，如果处在这两个数字之间则满足。
5. Regexp：自定义正则表达式。
6. URL：必须要是URL的形式。
7. UUID：验证UUID。

## 自定义验证器

如果想要对表单中的某个字段进行更细化的验证，那么可以针对这个字段进行单独的验证。步骤如下：
1. 定义一个方法，方法的名字规则是：`validate_字段名(self,filed)`。
2. 在方法中，使用`field.data`可以获取到这个字段的具体的值。
3. 如果数据满足条件，那么可以什么都不做。如果验证失败，那么应该抛出一个`wtforms.validators.ValidationError`的异常，并且把验证失败的信息传到这个异常类中。
示例代码：
```python
captcha = StringField(validators=[Length(4,4)])
    # 1234
    def validate_captcha(self,field):
        if field.data != '1234':
            raise ValidationError('验证码错误！')
```

# 文件上传

1. 在模版中，form表单中，需要指定`encotype='multipart/form-data'`才能上传文件。
2. 在后台如果想要获取上传的文件，那么应该使用`request.files.get('avatar')`来获取。
3. 保存文件之前，先要使用`werkzeug.utils.secure_filename`来对上传上来的文件名进行一个过滤。这样才能保证不会有安全问题。 
4. 获取到上传上来的文件后，使用`avatar.save(路径)`方法来保存文件。、
5. 从服务器上读取文件，应该定义一个url与视图函数，来获取指定的文件。在这个视图函数中，使用`send_from_directory(文件的目录,文件名)`来获取。
示例代码如下：
```python
@app.route('/upload/',methods=['GET','POST'])
def upload():
    if request.method == 'GET':
        return render_template('upload.html')
    else:
        # 获取描述信息
        desc = request.form.get("desc")
        avatar = request.files.get("avatar")
        filename = secure_filename(avatar.filename)
        avatar.save(os.path.join(UPLOAD_PATH,filename))
        print(desc)
        return '文件上传成功'

@app.route('/images/<filename>/')
def get_image(filename):
    return send_from_directory(UPLOAD_PATH,filename)
```

## 对上传文件使用表单验证

1. 定义表单的时候，对文件的字段，需要采用`FileField`这个类型。
2. 验证器应该从`flask_wtf.file`中导入。`flask_wtf.file.FileRequired`是用来验证文件上传是否为空。`flask_wtf.file.FileAllowed`用来验证上传的文件的后缀名。
3. 在视图文件中，使用`from werkzeug.datastructures import CombinedMultiDict`来把`request.form`与`request.files`来进行合并。再传给表单来验证。
示例代码如下：
```python
from wtforms import Form,FileField,StringField
from wtforms.validators import InputRequired
# flask_wtf
from flask_wtf.file import FileRequired,FileAllowed

class UploadForm(Form):
    avatar = FileField(validators=[FileRequired(),FileAllowed(['jpg','png','gif'])])
    desc = StringField(validators=[InputRequired()])
```



```python
from werkzeug.datastructures import CombinedMultiDict
form = UploadForm(CombinedMultiDict([request.form,request.files]))
```

# context 上下文

## Local对象：
在`Flask`中，类似于`request`的对象，其实是绑定到了一个`werkzeug.local.Local`对象上。这样，即使是同一个对象，那么在多个线程中都是隔离的。类似的对象还有`session`以及`g`对象。

## Thread Local对象：
只要满足绑定到这个对象上的属性，在每个线程中都是隔离的，那么他就叫做`Thread Local`对象。


## 应用上下文和请求上下文：
应用上下文和请求上下文都是存放到一个`LocalStack`的栈中。和应用app相关的操作就必须要用到应用上下文，比如通过`current_app`获取当前的这个`app`。和请求相关的操作就必须用到请求上下文，比如使用`url_for`反转视图函数。
1. 在视图函数中，不用担心上下文的问题。因为视图函数要执行，那么肯定是通过访问url的方式执行的，那么这种情况下，Flask底层就已经自动的帮我们把请求上下文和应用上下文都推入到了相应的栈中。
2. 如果想要在视图函数外面执行相关的操作，比如获取当前的app(current_app)，或者是反转url，那么就必须要手动推入相关的上下文：
    * 手动推入app上下文：
        ```python
        # 第一种方式：
        app_context = app.app_context()
        app_context.push()
        # 第二种方式：
        with app.app_context():
            print(current_app)
        ```
    * 手动推入请求上下文：推入请求上下文到栈中，会首先判断有没有应用上下文，如果没有那么就会先推入应用上下文到栈中，然后再推入请求上下文到栈中：
        ```python
        with app.test_request_context():
            print(url_for('my_list'))
        ```

## 为什么上下文需要放在栈中：
1. 应用上下文：Flask底层是基于werkzeug，werkzeug是可以包含多个app的，所以这时候用一个栈来保存。如果你在使用app1，那么app1应该是要在栈的顶部，如果用完了app1，那么app1应该从栈中删除。方便其他代码使用下面的app。
2. 如果在写测试代码，或者离线脚本的时候，我们有时候可能需要创建多个请求上下文，这时候就需要存放到一个栈中了。使用哪个请求上下文的时候，就把对应的请求上下文放到栈的顶部，用完了就要把这个请求上下文从栈中移除掉。


## 保存全局对象的g对象：
g对象是在整个Flask应用运行期间都是可以使用的。并且他也是跟request一样，是线程隔离的。这个对象是专门用来存储开发者自己定义的一些数据，方便在整个Flask程序中都可以使用。一般使用就是，将一些经常会用到的数据绑定到上面，以后就直接从g上面取就可以了，而不需要通过传参的形式，这样更加方便。


## 常用的钩子函数：
在Flask中钩子函数是使用特定的装饰器装饰的函数。为什么叫做钩子函数呢，是因为钩子函数可以在正常执行的代码中，插入一段自己想要执行的代码。那么这种函数就叫做钩子函数。（hook）
1. `before_first_request`：Flask项目第一次部署后会执行的钩子函数。
2. `before_request`：请求已经到达了Flask，但是还没有进入到具体的视图函数之前调用。一般这个就是在视图函数之前，我们可以把一些后面需要用到的数据先处理好，方便视图函数使用。
3. `context_processor`：使用这个钩子函数，必须返回一个字典。这个字典中的值在所有模版中都可以使用。这个钩子函数的函数是，如果一些在很多模版中都要用到的变量，那么就可以使用这个钩子函数来返回，而不用在每个视图函数中的`render_template`中去写，这样可以让代码更加简洁和好维护。
4. `errorhandler`：在发生一些异常的时候，比如404错误，比如500错误。那么如果想要优雅的处理这些错误，就可以使用`errorhandler`来出来。需要注意几点：
    * 在errorhandler装饰的钩子函数下，记得要返回相应的状态码。
    * 在errorhandler装饰的钩子函数中，必须要写一个参数，来接收错误的信息，如果没有参数，就会直接报错。
    * 使用`flask.abort`可以手动的抛出相应的错误，比如开发者在发现参数不正确的时候可以自己手动的抛出一个400错误。
示例代码如下：
```python
@app.errorhandler(404)
def page_not_found(error):
    return render_template('404.html'),404
```

# Cookie 和 Session

## 什么是cookie
在网站中，http请求是无状态的。也就是说即使第一次和服务器连接后并且登录成功后，第二次请求服务器依然不能知道当前请求是哪个用户。cookie的出现就是为了解决这个问题，第一次登录后服务器返回一些数据（cookie）给浏览器，然后浏览器保存在本地，当该用户发送第二次请求的时候，就会自动的把上次请求存储的cookie数据自动的携带给服务器，服务器通过浏览器携带的数据就能判断当前用户是哪个了。cookie存储的数据量有限，不同的浏览器有不同的存储大小，但一般不超过4KB。因此使用cookie只能存储一些小量的数据。
1. cookie有有效期：服务器可以设置cookie的有效期，以后浏览器会自动的清除过期的cookie。
2. cookie有域名的概念：只有访问同一个域名，才会把之前相同域名返回的cookie携带给服务器。也就是说，访问谷歌的时候，不会把百度的cookie发送给谷歌。

## flask操作cookie
1. 设置cookie：设置cookie是应该在Response的对象上设置。`flask.Response`对象有一个`set_cookie`方法，可以通过这个方法来设置`cookie`信息。
在Chrome浏览器中查看cookie的方式：
    * 右键->检查->Network->重新加载页面->找到请求，然后查看Response Headers中的cookie
    * 点击url输入框左边的信息icon，然后找到相应的域名，再展开查看cookie。
    * 在Chrome的设置界面->高级设置->内容设置->所有cookie->找到当前域名下的cookie。 
2. 删除cookie：通过`Response.delete_cookie`，指定cookie的key，就可以删除cookie了。
3. 设置cookie的有效期：
    * max_age：以秒为单位，距离现在多少秒后cookie会过期。
    * expires：为 `datetime` 类型。这个时间需要设置为格林尼治时间，也就是要距离北京少8个小时的时间（+8）。
    * 如果max_age和expires都设置了，那么这时候以max_age为标准。
    * max_age在IE8以下的浏览器是不支持的。expires虽然在新版的HTTP协议中是被废弃了，但是到目前为止，所有的浏览器都还是能够支持，所以如果想要兼容IE8以下的浏览器，那么应该使用expires，否则可以使用max_age。
    * 默认的过期时间：如果没有显示的指定过期时间，那么这个cookie将会在浏览器关闭后过期。
示例代码：
```python
@app.route('/')
def hello_world():
    resp = Response("知了课堂")
    expires = datetime(year=2017,month=12,day=11,hour=16,minute=0,second=0)
    # 使用expires参数，就必须使用格林尼治时间
    # 要相对北京时间少8个小时
    expires = datetime.now() + timedelta(days=30,hours=16)
    # 在新版本的http协议中，expires参数视为被废弃
    # max_age，在IE8一下的浏览器中是不支持的
    # resp.set_cookie('username','zhiliao',expires=expires,max_age=60)
    resp.set_cookie('username','zhiliao')
    return resp
```
4. 设置cookie的有效域名：cookie默认是只能在主域名下使用。如果想要在子域名下使用，那么应该给`set_cookie`传递一个`domain='.hy.com'`，这样其他子域名才能访问到这个cookie信息。


## session
1. session的基本概念：session和cookie的作用有点类似，都是为了存储用户相关的信息。不同的是，cookie是存储在本地浏览器，session是一个思路、一个概念、一个服务器存储授权信息的解决方案，不同的服务器，不同的框架，不同的语言有不同的实现。虽然实现不一样，但是他们的目的都是服务器为了方便存储数据的。session的出现，是为了解决cookie存储数据不安全的问题的。
2. session与cookie的结合使用：
    * session存储在服务器端：服务器端可以采用mysql、redis、memcached等来存储session信息。原理是，客户端发送验证信息过来（比如用户名和密码），服务器验证成功后，把用户的相关信息存储到session中，然后随机生成一个唯一的session_id，再把这个session_id存储cookie中返回给浏览器。浏览器以后再请求我们服务器的时候，就会把这个session_id自动的发送给服务器，服务器再从cookie中提取session_id，然后从服务器的session容器中找到这个用户的相关信息。这样就可以达到安全识别用户的需求了。
    * session存储到客户端：原理是，客户端发送验证信息过来（比如用户名和密码）。服务器把相关的验证信息进行一个非常严格和安全的加密方式进行加密，然后再把这个加密后的信息存储到cookie，返回给浏览器。以后浏览器再请求服务器的时候，就会自动的把cookie发送给服务器，服务器拿到cookie后，就从cookie找到加密的那个session信息，然后也可以实现安全识别用户的需求了。


## flask操作session
1. 设置session：通过`flask.session`就可以操作session了。操作`session`就跟操作字典是一样的。`session['username']='zhiliao'`。
2. 获取session：也是类似字典，`session.get(key)`。
3. 删除session中的值：也是类似字典。可以有三种方式删除session中的值。
    * `session.pop(key)`。
    * `del session[key]`。
    * `session.clear()`：删除session中所有的值。
4. 设置session的有效期：如果没有设置session的有效期。那么默认就是浏览器关闭后过期。如果设置session.permanent=True，那么就会默认在31天后过期。如果不想在31天后过期，那么可以设置`app.config['PERMANENT_SESSION_LIFETIME'] = timedelta(hour=2)`在两个小时后过期。

# Restful

## 安装
Flask-Restful需要在Flask 0.8以上的版本，在Python2.6或者Python3.3上运行。通过pip install flask-restful即可安装。

## 基本使用
1. 从`flask_restful`中导入`Api`，来创建一个`api`对象。
2. 写一个视图函数，让他继承自`Resource`，然后在这个里面，使用你想要的请求方式来定义相应的方法，比如你想要将这个视图只能采用`post`请求，那么就定义一个`post`方法。
3. 使用`api.add_resource`来添加视图与`url`。
示例代码如下：
```python
class LoginView(Resource):
    def post(self,username=None):
        return {"username":"zhiliao"}

api.add_resource(LoginView,'/login/<username>/','/regist/')
```
注意事项：
* 如果你想返回json数据，那么就使用flask_restful，如果你是想渲染模版，那么还是采用之前的方式，就是`app.route`的方式。
* url还是跟之前的一样，可以传递参数。也跟之前的不一样，可以指定多个url。
* endpoint是用来给url_for反转url的时候指定的。如果不写endpoint，那么将会使用视图的名字的小写来作为endpoint。


## 参数验证
Flask-Restful插件提供了类似WTForms来验证提交的数据是否合法的包，叫做reqparse。以下是基本用法：
    ```python
    parser = reqparse.RequestParser()
    parser.add_argument('username',type=str,help='请输入用户名')
    args = parser.parse_args()
    ```
add_argument可以指定这个字段的名字，这个字段的数据类型等。以下将对这个方法的一些参数做详细讲解： 
1. default：默认值，如果这个参数没有值，那么将使用这个参数指定的值。 
2. required：是否必须。默认为False，如果设置为True，那么这个参数就必须提交上来。 3. type：这个参数的数据类型，如果指定，那么将使用指定的数据类型来强制转换提交上来的值。 
4. choices：选项。提交上来的值只有满足这个选项中的值才符合验证通过，否则验证不通过。 
5. help：错误信息。如果验证失败后，将会使用这个参数指定的值作为错误信息。 
6. trim：是否要去掉前后的空格。

其中的type，可以使用python自带的一些数据类型，也可以使用flask_restful.inputs下的一些特定的数据类型来强制转换。比如一些常用的： 
1. url：会判断这个参数的值是否是一个url，如果不是，那么就会抛出异常。 
2. regex：正则表达式。 
3. date：将这个字符串转换为datetime.date数据类型。如果转换不成功，则会抛出一个异常。

## 返回

对于一个视图函数，你可以指定好一些字段用于返回。以后可以使用ORM模型或者自定义的模型的时候，他会自动的获取模型中的相应的字段，生成json数据，然后再返回给客户端。这其中需要导入flask_restful.marshal_with装饰器。并且需要写一个字典，来指示需要返回的字段，以及该字段的数据类型。示例代码如下：
```python
class ProfileView(Resource):
    resource_fields = {
        'username': fields.String,
        'age': fields.Integer,
        'school': fields.String
    }

    @marshal_with(resource_fields)
    def get(self,user_id):
        user = User.query.get(user_id)
        return user
```
在get方法中，返回user的时候，flask_restful会自动的读取user模型上的username以及age还有school属性。组装成一个json格式的字符串返回给客户端。

## 重命名属性

很多时候你面向公众的字段名称是不同于内部的属性名。使用 attribute可以配置这种映射。比如现在想要返回user.school中的值，但是在返回给外面的时候，想以education返回回去，那么可以这样写：
```python
resource_fields = {
    'education': fields.String(attribute='school')
}
```

## 默认值
在返回一些字段的时候，有时候可能没有值，那么这时候可以在指定fields的时候给定一个默认值，示例代码如下：
```python
resource_fields = {
    'age': fields.Integer(default=18)
}
```

## 复杂结构
有时候想要在返回的数据格式中，形成比较复杂的结构。那么可以使用一些特殊的字段来实现。比如要在一个字段中放置一个列表，那么可以使用fields.List，比如在一个字段下面又是一个字典，那么可以使用fields.Nested。以下将讲解下复杂结构的用法：
```python
class ArticleView(Resource):

    resource_fields = {
        'aritlce_title':fields.String(attribute='title'),
        'content':fields.String,
        'author': fields.Nested({
            'username': fields.String,
            'email': fields.String
        }),
        'tags': fields.List(fields.Nested({
            'id': fields.Integer,
            'name': fields.String
        })),
        'read_count': fields.Integer(default=80)
    }

    @marshal_with(resource_fields)
    def get(self,article_id):
        article = Article.query.get(article_id)
        return article
```


## Flask-restful注意事项
1. 在蓝图中，如果使用`flask-restful`，那么在创建`Api`对象的时候，就不要再使用`app`了，而是使用蓝图。
2. 如果在`flask-restful`的视图中想要返回`html`代码，或者是模版，那么就应该使用`api.representation`这个装饰器来定义一个函数，在这个函数中，应该对`html`代码进行一个封装，再返回。示例代码如下：
```python
@api.representation('text/html')
def output_html(data,code,headers):
    print(data)
    # 在representation装饰的函数中，必须返回一个Response对象
    resp = make_response(data)
    return resp

class ListView(Resource):
    def get(self):
        return render_template('index.html')
api.add_resource(ListView,'/list/',endpoint='list')
```

# signal 信号

## 使用signal

使用信号分为3步，第一是定义一个信号，第二是监听一个信号，第三是发送一个信号。以下将对这三步进行讲解：

1. 定义信号：定义信号需要使用到blinker这个包的Namespace类来创建一个命名空间。比如定义一个在访问了某个视图函数的时候的信号。示例代码如下：
    ```python
    # Namespace的作用：为了防止多人开发的时候，信号名字冲突的问题
    from blinker import Namespace

    mysignal = Namespace()
    visit_signal = mysignal.signal('visit-signal')
    ```
2. 监听信号：监听信号使用singal对象的connect方法，在这个方法中需要传递一个函数，用来接收以后监听到这个信号该做的事情。示例代码如下：
    ```python
    def visit_func(sender,username):
        print(sender)
        print(username)
    mysignal.connect(visit_func)
    ```
3. 发送信号：发送信号使用singal对象的send方法，这个方法可以传递一些其他参数过去。示例代码如下：
  ```python
  mysignal.send(username='zhiliao')
  ```

## Flask内置的信号
1. template_rendered：模版渲染完成后的信号。
2. before_render_template：模版渲染之前的信号。
3. request_started：模版开始渲染。
4. request_finished：模版渲染完成。
5. request_tearing_down：request对象被销毁的信号。
6. got_request_exception：视图函数发生异常的信号。一般可以监听这个信号，来记录网站异常信息。
7. appcontext_tearing_down：app上下文被销毁的信号。
8. appcontext_pushed：app上下文被推入到栈上的信号。
9. appcontext_popped：app上下文被推出栈中的信号
10. message_flashed：调用了Flask的`flashed`方法的信号。

# Redis

## RDB同步机制
1. 开启和关闭：默认情况下是开启了。如果想关闭，那么注释掉`redis.conf`文件中的所有`save`选项就可以了。
2. 同步机制：
    * save 900 1：如果在900s以内发生了1次数据更新操作，那么就会做一次同步操作。
    * save 300 10：如果在300s以内发生了10数据更新操作，那么就会做一次同步操作。
    * save 60 10000：如果在60s以内发生了10000数据更新操作，那么就会做一次同步操作。
3. 存储内容：具体的值，而是命令。并且是经过压缩后存储进去的。
4. 存储路径：根据`redis.conf`下的`dir`以及`rdbfilename`来指定的。默认是`/var/lib/redis/dump.rdb`。
5. 优点：
    * 存储数据到文件中会进行压缩，文件体积比aof小。
    * 因为存储的是redis具体的值，并且会经过压缩，因此在恢复的时候速度比AOF快。
    * 非常适用于备份。
6. 缺点：
    * RDB在多少时间内发生了多少写操作的时候就会出发同步机制，因为采用压缩机制，RDB在同步的时候都重新保存整个Redis中的数据，因此你一般会设置在最少5分钟才保存一次数据。在这种情况下，一旦服务器故障，会造成5分钟的数据丢失。
    * 在数据保存进RDB的时候，Redis会fork出一个子进程用来同步，在数据量比较大的时候，可能会非常耗时。

## AOF同步机制
1. 开启和关闭：默认是关闭的。如果想要开启，那么修改redis.conf中的`appendonly yes`就可以了
2. 同步机制：
    * appendfsync always：每次有数据更新操作，都会同步到文件中。
    * appendfsync everysec：每秒进行一次更新。
    * appendfsync no：使用操作系统的方式进行更新。普遍是30s更新一次。
3. 存储内容：存储的是具体的命令。不会进行压缩。
4. 存储路径：根据`redis.conf`下的`dir`以及`appendfilename`来指定的。默认是`/var/lib/redis/appendonly.aof`。
5. 优点：
    * AOF的策略是每秒钟或者每次发生写操作的时候都会同步，因此即使服务器故障，最多只会丢失1秒的数据。 
    * AOF存储的是Redis命令，并且是直接追加到aof文件后面，因此每次备份的时候只要添加新的数据进去就可以了。
    * 如果AOF文件比较大了，那么Redis会进行重写，只保留最小的命令集合。
6. 缺点：
    * AOF文件因为没有压缩，因此体积比RDB大。 
    * AOF是在每秒或者每次写操作都进行备份，因此如果并发量比较大，效率可能有点慢。
    * AOF文件因为存储的是命令，因此在灾难恢复的时候Redis会重新运行AOF中的命令，速度不及RDB。

## 给redis指定密码
1. 设置密码：在`reids.conf`配置文件中，将`requirepass pasword`取消注释，并且指定你想设置的密码。
2. 使用密码连接reids：
    * 先登录上去，然后再使用`autho password`命令进行授权。
    * 在连接的时候，通过`-a`参数指定密码进行连接。

## 其他机器连接redis
如果想要让其他机器连接本机的redis服务器，那么应该在`redis.conf`配置文件中，指定`bind 本机的ip地址`。这样别的机器就能连接成功。不像是网上说的，要指定对方的ip地址。


































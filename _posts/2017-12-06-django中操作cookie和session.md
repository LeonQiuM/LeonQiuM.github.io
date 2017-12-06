---
layout: post
title:  "django中操作cookie&session"
date:   2017-12-06
desc: "django中操作cookie&session"
keywords: "Django,session,cookie,Python"
categories: [HTML]
tags: [Django,Python,SESSION,COOKIE]
icon: icon-html
---
## cookie

----

#### cookie是什么
由于http协议是无状态的，即服务器不知道用户的上一步操作是干了什么。这严重阻碍了`交互式WEB应用程序`的实现。cookie是http的一个额外的手段，用来保存用户的一部分信息到用户端，即cookie使保存在用户端的，由用户的浏览器来维护，通常被分为两种类型：内存cookie、硬盘cookie。cookie被夹带在http请求中，一般大小限制在4KB左右

服务器在响应中添加设置过cookie后，支持cookie的浏览器都会做出反应，来使用某种方式来保存这个cookie，当用户下次再次发出请求的时候，浏览器判断当前cookie是否失效（expirse）、匹配路径（path）、匹配域名(domain)等操作后，将这段cookie加入到请求头中发送到服务器端。服务端对其进行处理

#### 在django中使用cookie
+ **实现保存用户登录状态**

```python
# views.py
def login(request):
    if request.method == "GET":
        return render(request, 'login.html')
    elif request.method == "POST":
        u = request.POST.get("username", None)
        p = request.POST.get("password", None)
        dic = user_info.get(u)
        print(dic)
        if not dic:
            return render(request, 'login.html')
        elif dic.get('pwd') == p:
           # 密码正确后设定cookie
            response = redirect('/index/')
            response.set_cookie("username", u)
            return response
        else:
            return render(request, 'login.html')
    else:
        return render(request, 'login.html')

#index
def index(request):
    ck = request.COOKIES.get("username")  # 早cookie中获取当前登录的用户
    if not ck:
         return redirect('/login/')
    return render(request, 'index.html', {'current_user': ck})
```
当用户名和密码验证成功后，在返回的`redirect`中设定`cookie`。通过这种简单的例子即可以实现保存用户的登录状态，当用户下一次访问index的时候就不在需要登录

+ **cookie的一些其他设定：**

1.`request.COOKIES` 用户发来数据中带来的COOKIE，为一个字典，可以使用get来获取
2.`response.set_cookie("字典的key"，'字典的值')` 后面不加参数的时候，只能设置一个当浏览器关闭就失效的cookie
3.`response.set_cookie("username",'value',max_age=10)`  多少秒后过期的cookie
4.通过datetime来设定过期

```python
current_date = datetime.datetime.utcnow()
current_date = current_date+ datetime.timedelta(seconds=10)   # 当前时间加上10s后
response.set_cookie("username", u, expires=current_date)  # 设定到哪个时间点后失效，如果时间设置与当前时间相同，那么就是清除这个cookie
```
 5.设定cookie作用的路径

```
response.set_cookie("username", u,path='/index/')  # 设置这个cookie只在当前url生效，例如设定一个cookie为当前页面显示多少条数据，别的页面就不会被干扰
```
6.`domain=''`  为当前设置cookie的域名
7.`secure = False` 为https传输设置cookie
8.`httponly = True` 设置cookie只做为http传输，不可以被js获取到。在js中使用`document.cookie`获取可以所有cookie，或者使用JQuery也可以操作cookie

9.带salt的cookie

```
# 设定加盐
COOKIE_SALT = "随机字符串"
response.set_signed_cookie('username', u, salt=COOKIE_SALT)

# 加盐获取
ck = request.get_singed_cookie("username",salt=COOKIE_SALT)
```

----

## session


#### session是什么 
[参考：晋哥哥的私房钱](http://www.cnblogs.com/shoru/archive/2010/02/19/1669395.html)

session一般被译为会话，从不同的角度上来看到会话，是有不同的意义的：

1.  从用户的角度来看，他打开一个网站，并进行了一系列的浏览、登录、购物等操作，这就可以被称为一个会话
2. 从底层来细细分析呢，由于http的无状态性，用户登录我们需要保存他的登录状态，当前购物车中的所有商品都需要保存下来，所以我们需要一个特定的数据结构来保存这些数据。这个东西就被称为session

所以session是一种基于http协议的用于增强http的一种数据存储结构或方案，session存储在服务端。服务端创建session的一般步骤分为：生成一个全局唯一标识`sessionid`，在对应的数据中开辟存储空间，然后将session的全局唯一标识符发送到客户端。 服务端为每一个session维护一份会话信息数据，而客户端和服务端依靠一个全局唯一的标识来访问会话信息数据。

那么客户端和服务端怎么发送这个标识符呢？一般实现有两种方式：

+ cookie，服务端通过设定cookie的方式，将sessionid发送到客户端
![](/content/images/2017/11/QQ20171107-152411.png)
+ url重写，在返回用户请求的页面之前，将页面内所有的URL后面全部以get的方式加上session标识符，用户在下次操作上，都会加上这个标识符，从而实现了会话保持，这也是当用户禁用cookie的做有效的办法


#### cookie与session的对比
1.   应用场景

  + Cookie的典型应用场景是记住密码、记住我操作，用户的账户信息通过cookie的形式保存在客户端，当用户再次请求匹配的URL的时候，账户信息会被传送到服务端，交由相应的程序完成自动登录等功能。当然也可以保存一些客户端信息，比如页面布局以及搜索历史等等。
  + Session的典型应用场景是用户登录某网站之后，将其登录信息放入session，在以后的每次请求中查询相应的登录信息以确保该用户合法。当然还是有购物车等等经典场景；
2.   安全性
  + cookie将信息保存在客户端，如果不进行加密的话，无疑会暴露一些隐私信息，安全性很差，一般情况下敏感信息是经过加密后存储在cookie中，但很容易就会被窃取。
  + session只会将信息存储在服务端，如果存储在文件或数据库中，也有被窃取的可能，只是可能性比cookie小了太多。这里涉及到的是硬件安全，但是总体来说session的安全性要高于cookie；
3.   性能
  + Cookie存储在客户端，消耗的是客户端的I/O和内存，而session存储在服务端，消耗的是服务端的资源。
  + session对服务器造成的压力比较集中，而cookie很好地分散了资源消耗，就这点来说，cookie是要优于session的；
4.   时效性
  + Cookie可以通过设置有效期使其较长时间内存在于客户端，而
  + session一般只有比较短的有效期（用户主动销毁session或关闭浏览器后引发超时）；
5.   其他
Cookie的处理在开发中没有session方便。而且cookie在客户端是有数量和大小的限制的，而session的大小却只以硬件为限制，能存储的数据无疑大了太多。

#### django中使用session


注：

1. django默认使用`django.contrib.sessions.models.Session`模块，将session存储在数据库的`django_session `表中，当然这些都是可以配置的

2. 所以需要在使用`database-backed sessions`前进行一定的设定，在数据库当中生成存储session的表以及字段

```shell
python manage.py makemigrations
python manage.py migrate
```


+ **简单实现**

```
# views.py
def login(request):
    if request.method == "GET":
        return render(request, "login.html")

    if request.method == "POST":
        user = request.POST.get('user')
        pwd = request.POST.get('pwd')
        rmb = request.POST.get('rmb',None)
        print(rmb)
        if user == 'root' and pwd == "123":
            # 直接设定值
            request.session['username'] = user
            request.session['is_login'] = True
            if rmb == "10":
                request.session.set_expiry(10)  # 设置多少秒后过期
            # 生成的session默认存储在django的默认数据库中
            return redirect('/index/')
        else:
            return render(request, "login.html")

def logout(request):
    # 注销
    if request.method == 'POST':
        print(request.session.get('username'))
        request.session.delete(request.session.session_key) # 从数据库中删除
        print(request.session.session_key)
        print(request.session.get('username'))
        return redirect('/login/')

def index(request):
    if request.session.get('is_login'):
        return render(request, 'index.html')
    else:
        return HttpResponse("Fuck off")
```
+ 在这样的应用中访问会在数据库中生成以下数据

![session-table](/content/images/2017/11/session-table.png)

+ **更多操作**
  
```
1. 获取session中的值
request.session['k1']
request.session.get('k1',None)
2. 设定session中的值
request.session['k1'] = 123
request.session.setdefault('k1',123) # 存在则不设置
3. 删除session对应的值
del request.session['k1']
4. 由于session的真是数据结构其实是个字典对象，所以拥有字典的一些方法：
request.session.keys() # 所有的key
request.session.values() # 所有的value
request.session.items() # 键值对元组
# 其他方法
request.session.iterkeys() # key的可迭代对象
request.session.itervalues() # value的可迭代对象
request.session.iteritems()  # 键值对元组的可迭代对象
5. 用户的全局唯一sessionid
request.session.session_key
6. 将所有Session失效日期小于当前日期的数据删除
request.session.clear_expired()
7. 判断是否存在
request.session.exists("session_key")
8. 删除这个sessionid，并也会删除其对应的数据
request.session.delete("session_key")
9. 过期设定
request.session.set_expiry(value)
    * 如果value是个整数，session会在些秒数后失效。
    * 如果value是个datatime或timedelta，session就会在这个时间后失效
    * 如果value是0,用户关闭浏览器session就会失效。
    * 如果value是None,session会依赖全局session失效策略。（默认是两个周）
```
+ **配置相关**

1.引擎相关
除了django自己支持的以下几种引擎以外，还可以使用redis作为session的存储，可以参考redis的设置方法    [猛戳](http://django-redis-chs.readthedocs.io/zh_CN/latest/#cache-backend)

```python
# 数据库存储
SESSION_ENGINE = 'django.contrib.sessions.backends.db'
# 缓存存储
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
# 文件存储
SESSION_ENGINE = 'django.contrib.sessions.backends.file'
SESSION_FILE_PATH = None # 缓存文件路径，如果为None，则使用tempfile模块获取一个临时地址tempfile.gettempdir()
# 缓存加数据库
SESSION_ENGINE = 'django.contrib.sessions.backends.cached_db'
# 加密cookie Session
SESSION_ENGINE = 'django.contrib.sessions.backends.signed_cookies'
```

2.通用设定

```
SESSION_COOKIE_NAME ＝ "sessionid" # Session的cookie保存在浏览器上时的key，即：sessionid＝随机字符串
SESSION_COOKIE_PATH ＝ "/"  # Session的cookie保存的路径
SESSION_COOKIE_DOMAIN = None  # Session的cookie保存的域名
SESSION_COOKIE_SECURE = False  # 是否Https传输cookie
SESSION_COOKIE_HTTPONLY = True  # 是否Session的cookie只支持http传输
SESSION_COOKIE_AGE = 1209600  # Session的cookie失效日期（2周）
SESSION_EXPIRE_AT_BROWSER_CLOSE = False # 是否关闭浏览器使得Session过期
SESSION_SAVE_EVERY_REQUEST = False # 是否每次请求都保存Session，默认修改之后才保存
```




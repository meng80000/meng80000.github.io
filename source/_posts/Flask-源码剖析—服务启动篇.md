---
title: Flask 源码剖析—服务启动篇
date: 2020-07-18 23:23:17
tags: 
	- Flask
	- web
categories: 框架
---

### 【Flask官方文档经典示例】 hello.py
```
from flask import Flask
app = Flask(__name__)

<!-- more -->

@app.route("/")
def hello():
    return "Hello World!"

if __name__ == "__main__":
    app.run()
```
输入以下命令启动应用程序：
```
$ python hello.py
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```
打开你的浏览器并在地址栏输入[http://127.0.0.1:5000/](http://127.0.0.1:5000/) 。【图1】显示连接到应用程序后的浏览器。

![1](1.png)

图1 *hello.py Flask应用程序*
### 服务是怎么启动的
从```app.run()```开始，这行代码表示启动一个服务。我们看到```app```是```Flask```一个对象，而```run()```是该对象的一个方法。我们先简单的认为定义了一个类，然后实例化这个类并调用该类的一个方法，如下：

【示例1-1】example-1-1.py
```
class Flask(object):
    def run(self):
        pass

app = Flask()
app.run()
```
如果我们运行【示例1-1】这段代码，会发现什么都没有发生。然而，【Flask官方文档经典示例】不是这样的，当你运行后它是下面这样的：
```
$ python hello.py
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```
很自然的可以想到，【Flask官方文档经典示例】中的```app.run()```不简单，我们可以看看```run()```方法定义，如下：
```
def run(self, host=None, port=None, debug=None, **options):
    from werkzeug.serving import run_simple
    ...
    try:
        run_simple(host, port, self, **options)
    finally:
        self._got_first_request = False
```
在这个方法中，我们先忽略一些配置操作，重点关注```run_simple()```函数，发现该函数是从```werkzeug.serving```模块中导入的。

到了这里我们就不得不提一下```Werkzeug```了，官方文档定义：```Werkzeug```是为Python设计的```HTTP```和```WSGI```实用程序库。至于它有什么作用，我们在这里暂且不讨论，先跟到代码里面看看它都做了什么。

我们看到这个```run_simple()```函数里面还嵌套了一个inner()函数，里面有几行关键代码，如下：
```
srv = make_server(hostname, port, application, threaded, 
                  processes, request_handler, 
                  passthrough_errors, ssl_context, 
                  fd=fd)
...
srv.serve_forever()
```
从上面的代码，我们看到在```inner()```函数里面调用了```make_server()```函数来创建一个类实例，该实例会调用```serve_forever()```方法让服务一直运行，等待客户端的请求。到这里我们大概找到了服务启动的入口了，想知道具体是怎么启动，我们还需要深入挖掘一下。

因为调用```run_simple()```函数时参数```threaded```和```processes```给的都是默认值，分别为```False```和```1```，所以在这里```make_server()```函数其实是创建了一个```BaseWSGIServer```类实例，并调用该实例的```serve_forever()```方法，具体```make_server()```函数如下：
```
def make_server(host=None, port=None, app=None, threaded=False, processes=1,
                request_handler=None, passthrough_errors=False,
                ssl_context=None, fd=None):
    if threaded and processes > 1:
        raise ValueError("cannot have a multithreaded and "
                         "multi process server.")
    elif threaded:
        return ThreadedWSGIServer(host, port, app, request_handler,
                                  passthrough_errors, ssl_context, fd=fd)
    elif processes > 1:
        return ForkingWSGIServer(host, port, app, processes, request_handler, 
                                 passthrough_errors, ssl_context, fd=fd)
    else:
        return BaseWSGIServer(host, port, app, request_handler, 
                              passthrough_errors, ssl_context, fd=fd)
```
找到```BaseWSGIServer```类，如下代码：
```
class BaseWSGIServer(HTTPServer, object):
    ...
    def serve_forever(self):
        self.shutdown_signal = False
        try:
            HTTPServer.serve_forever(self)
        except KeyboardInterrupt:
            pass
        finally:
            self.server_close()
    ...
```
【注意】接下来的代码嵌套调用比较多，所以最好是能对照着源码来看。
```srv.serve_forever()```其实是```BaseWSGIServer```类中的```serve_forever()```方法，然后我们发现```BaseWSGIServer```类继承了```HTTPServer```类，且```BaseWSGIServer```的```serve_forever()```方法中调用了```HTTPServer```的```serve_forever()```方法。找到```HTTPServer```类，如下代码：
```
class HTTPServer(SocketServer.TCPServer):
    allow_reuse_address = 1
    def server_bind(self):
        SocketServer.TCPServer.server_bind(self)
        host, port = self.socket.getsockname()[:2]
        self.server_name = socket.getfqdn(host)
        self.server_port = port
```
```HTTPServer```类中并没有```serve_forever()```方法，且这个类继承了``` SocketServer.TCPServer```，我们再找到```TCPServer```类，然而它也没有```serve_forever()```方法，且这个类继承了```BaseServer```类，所以再去```BaseServer```里面看看，如下代码：
```
def serve_forever(self, poll_interval=0.5):
    self.__is_shut_down.clear()
    try:
        while not self.__shutdown_request:
            r, w, e = _eintr_retry(select.select, [self], [], [], poll_interval)
            if self in r:
                self._handle_request_noblock()
    finally:
        self.__shutdown_request = False
        self.__is_shut_down.set()
```
所以前面看到的```srv.serve_forever()```其实是调用了```BaseServer```里面的```serve_forever()```方法，它接受一个参数```poll_interval```，用于表示```select```轮询的时间。然后进入一个无限循环，调用```select```方式进行网络IO监听。也就是说```app.run()```启动的是一个```BaseWSGIServer```，该服务通过一层一层的继承创建socket来进行网络监听，等待客户端连接。

至此，Flask服务是怎么启动的应该有个基本的了解了。

整理一下相关server类的继承关系，如下：
```BaseWSGIServer-->HTTPServer-->SocketServer.TCPServer-->BaseServer```
从上面的类继承关系，我们可以很容易的理解，因为```Flask```是一个Web框架，所以需要一个```HTTP服务```，而```HTTP服务是基于TCP服务```的，而```TCP服务最终会有一个基础服务来处理socket```。这一条线都能够解释的通。但是，那个```BaseWSGIServer```是个什么鬼？为什么会需要一层这个服务？这也是我想要去研究的，所以我会在下一篇里面进行讲解。

- [Flask 源码剖析——服务启动篇](https://segmentfault.com/a/1190000005788124)

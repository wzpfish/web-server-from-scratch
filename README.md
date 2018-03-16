# web-server-from-scratch
Learn more about web server, from socket to http server to web framework.

## Learn from source code
This project aim to learn tcp server, http server and wsgi server from Python3 lib. 

If you already know these concepts, go through the following notebooks.
* [TCP Server](https://nbviewer.jupyter.org/github/wzpfish/web-server-from-scratch/blob/master/tcpserver.ipynb)
* [HTTP Server](https://nbviewer.jupyter.org/github/wzpfish/web-server-from-scratch/blob/master/httpserver.ipynb)
* [WSGI Server](https://nbviewer.jupyter.org/github/wzpfish/web-server-from-scratch/blob/master/wsgi.ipynb)

else, read the following explaination and come back to the notebooks.

## Web Server? Web application? Web framework?
我们都知道，一个完整的HTTP连接主要有如下流程：
1. client和server建立TCP连接
2. client向server发送请求
3. server处理client请求，将相应资源返回给client
4. 关闭TCP连接

因此，一个http server需要做如下工作：处理client的连接请求，生成client要求的资源，发送给client。

当然，一个完整的server需要处理更多的细节，比如并发，缓存，日志，安全等等。有许多成熟的web server，比如nginx, apache等等。

而根据一个HTTP请求生成相应的资源这部分逻辑，则可以单独抽象出来，交由web application来处理。因此，一个web application负责的主要是：
将web server拿到的http请求转换成对应的资源，这个资源可以是静态的，也可以是动态生成的。

web application中也有许多重复的工作，比如怎么进行url映射(路由),怎么生成页面，怎么获取数据等等。因此，就出现了许多web framework专门处理
这些重复繁琐的事情，比如flask等等。

## WSGI
一个完整的HTTP通信，需要web server和web application配合完成，那一个web server怎么知道如何与web application通信呢？在Python里，规定了一个
协议，即WSGI(Web Server Gateway Interface)。只要开发web server和web application的了遵从这套协议，就可以愉快地协作了。

WSGI协议规定了server和framework要提供什么以及要做什么。

对于server:
1. 要提供什么?
```Python
def write(data):
    """start_response的返回值，用于向response body中写数据.
    """
    # write data to response body.
    
def start_response(status, response_headers, exc_info=None):
    """"暴露给application, 用于设置response的code和headers.
    status: http response code
    response_headers: http response headers, list of tuple (header_name, header_value)
    exc_info: a sys.exc_info() object. 表示执行request的出错信息.
    """"
    # handle exc_info.
    # set response headers.
    return write
```
2. 要做什么?

对于每个client过来的请求，通过调用application提供的callable来获取请求处理结果。

对于application:
1. 要提供什么？
```Python
def app(env, start_response):
    """env: wsgi环境变量，包括HTTP method, request string等等。
       start_response: server端提供的函数。
    """
    # Handle request given env.
    status = ...
    headers = ...
    start_response(status, headers)
    yield "hello"
    yield "world"
```
即提供一个callable对象，可以是函数，类或类对象等等，总之能调用就行。调用该对象需要传入两个参数，env和start_response。

2. 要做什么？

callable在执行时，必须在返回data前调用start_response将status和headers设上。

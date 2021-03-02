# Introduction

- tornado是一个Python web框架和异步网络库。通过使用**非阻塞网络I/O**，Tornado解决了C10K问题。适用于长时间轮询。

- 四个主要的组件：

  - **RequestHandler**：子类化它——创建web app等
  - **HttpServer和AsyncHTTPClient**：HTTP的客户端和服务器实现
  - **IOLoop和IOStream**：异步网络库，用于实现HTTP组件，或者其他网络协议
  - **Tornado.gen**:基于生成器的接口，保证代码异步运行

- 异步和非阻塞IO

  - Tornado使用单线程的event loop，以实现最小化并发连接的成本。所以Application里的代码都应该是异步、非阻塞的。

- 协同程序（Coroutines）

  - 实现异步的方法。

- Queue

  - **tornado.queue**模块为协同程序实现了一个异步生产者/消费者模式。

- tornado中**web application的结构**

  - 通常包括一个或多个RequestHandler子类、一个Application对象（将传入的请求路由到对应的handler）、一个启动server的main函数。

  - Application负责全局配置。其中包括路由表。

    - 路由表是映射请求到处理程序的表，是URLSpec对象（至少有一个正则表达式和一个handler）的列表，如果正则表达式有含有captuing groups，那它们就是路径参数，传递给RequestHandler的HTTP方法（get、post、patch、put、delete）；如果有一个字典作为URLSpec的第三个参数传递，那它将会传递给RequestHandler.Initialize方法。URLSpec对象如果有 名称的话，可与Requestthandler.reverse _ url一起使用。

      ```python
      class MainHandler(RequestHandler):
          def get(self):
              self.write('<a href="%s">link to story 1</a>' %
                         self.reverse_url("story", "1"))
      
      class StoryHandler(RequestHandler):
          def initialize(self, db):
              self.db = db
      
          def get(self, story_id):
              self.write("this is story %s" % story_id)
      
      app = Application([
          url(r"/", MainHandler),
          url(r"/story/([0-9]+)", StoryHandler, dict(db=db), name="story")
          ])
      ```

    - Application构造函数接受许多参数，可用于定制app的行为和启用可选特性。详见[Application.settings](https://www.tornadoweb.org/en/stable/web.html#tornado.web.Application.settings)

  - RequestHandler完成web应用的大部分工作。

    - 一般先定义一个BaseHandler继承RequestHandler，覆盖像[`get_current_user`](https://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler.get_current_user) 和[`write_error`](https://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler.write_error) 这样的方法，然后再定义其他Handler，都继承BaseHandler而不是RequestHandler。
    - RequestHandler入口点一般是get(),post()这样的Http请求处理函数。其参数从路由中获得。
    - 在Handler中，一般使用[`RequestHandler.render`](https://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler.render) 或者 [`RequestHandler.write`](https://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler.write)来生成响应。其中前者按名称加载模板并使用给定参数渲染模板，后者一般用于没有模板的输出。
    - 在请求发生并被匹配到某个RequestHandler后，会创建一个新的RequestHandler对象，Handler的下列方法会被依次调用（如果被重写了的话）：
      - **initialize()**
      - **prepare()**
      - **某个Http方法，例如get()/put()/post()。**
      - **on_finish()**
    - 一些常见的重写方法：
      - [`write_error`](https://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler.write_error) - 输出用于错误页面的 HTML
      - [`on_connection_close`](https://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler.on_connection_close) - 在客户端断开连接时调用; 应用程序可以选择检测此情况并停止进一步处理。但不能保证可以迅速检测到关闭的连接。
      - [`get_current_user`](https://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler.get_current_user) - 参见[`用户认证`](https://www.tornadoweb.org/en/stable/guide/security.html#user-authentication).
      - [`get_user_locale`](https://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler.get_user_locale) -返回[`Locale`](https://www.tornadoweb.org/en/stable/locale.html#tornado.locale.Locale) 对象（当前用户）
      - [`set_default_headers`](https://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler.set_default_headers) - 可用于在响应上设置额外的标头(例如自定义`Server` header).

  - 错误处理

  - 重定向 

    - 主要有[`RequestHandler.redirect`](https://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler.redirect) 和 [`RedirectHandler`](https://www.tornadoweb.org/en/stable/web.html#tornado.web.RedirectHandler)来处理重定向.前者用于RequestHandler中，后者用于直接在路由表中配置重定向。

  - 异步handlers

    - 某些处理程序方法(包括 prepare ()和 HTTP 方法 get ()/post ()等)可以作为协同程序重写，以实现handeler的异步。

- 认证和安全

  - cookies和secure cookies

    - cookies用来标识当前用户，使用set_cookies()方法来设置cookie。

      ```python
      class MainHandler(tornado.web.RequestHandler):
          def get(self):
              if not self.get_cookie("mycookie"):
                  self.set_cookie("mycookie", "myvalue")
                  self.write("Your cookie was not set yet!")
              else:
                  self.write("Your cookie was set!")
      ```

      

    - 防止客户端伪造Cookie：在cookie上签名。在创建app时指定cookie_secret（可以在application settings作为参数传递进来），然后使用set_secure_cookie和get_secure_cookie方法支持签名的cookie。

    - 默认有效期是30天，可修改。

  - User authentication

    - 经过了身份验证的用户，在handler中使用self.current_user获取，在模板中使用current_user获取。
    - 在app中要实现用户身份验证，需要覆盖处理请求中的get_current_user()方法以便确定当前用户（例如根据cookie的值确定当前用户或者要求用户登陆等）。
    - 可以使用装饰器[`tornado.web.authenticated`](https://www.tornadoweb.org/en/stable/web.html#tornado.web.authenticated)来装饰http方法，要求用户登录(如果用户没有登录，将重定向到登录界面)。

  - 第三方认证

    -  [`tornado.auth`](https://www.tornadoweb.org/en/stable/auth.html#module-tornado.auth)模块实现了许多流行网站（googe、facebook等）的认证和授权协议。

  - 跨站请求伪造保护

  - DNS重新绑定

- 运行和部署

  - 定义一个main函数，启动服务器。

    ```python
    def main():
        app = make_app()
        app.listen(8888)
        IOLoop.current().start()
    
    if __name__ == '__main__':
        main()
    ```

    

- 进程和端口

  - Tornado包含一个内置的多进程模式，可以同时启动多个进程。

- Running behind a load balancer

- 静态文件和侵略性文件缓存

- Debug模式和自动reloading

  - 如果向Application构造函数传递debug=true，应用程序将在debug/dev模式下运行。此时将启动几个为开发方便而设计的几个特性:
    - `autoreload=True`: 应用程序将观察其源文件的变化，并在任何变化时重新加载本身。这减少了在开发过程中手动重启服务器的需要。但是，某些故障(例如导入时的语法错误)仍然会以调试模式当前无法恢复的方式导致服务器宕机
    - `compiled_template_cache=False`: 不缓存模板
    - `static_hash_cache=False`:  不缓存静态hash(`static_url` 函数使用的)
    - `serve_traceback=True`:当在一个[`RequestHandler`](https://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler) 的异常没有被捕获，则将生成包含堆栈跟踪的错误页面
  - Autoload模式和HTTPSever的多进程模式不兼容。
  - debug模式的auto reload功能可以作为一个单独的模块使用。
# WSGI、uwsgi、uWSGI、Nginx的区别

- `WSGI 的全称是 Web Server Gateway Interface(Web服务器网关接口），它不是服务器、Python 模块、框架、API 或者任何软件，只是一种描述 Web 服务器和 Web 应用程序(使用 Web 框架，如 Django、Flask 编写的程序）进行通信的规范、协议。使用任何一个框架在编写完服务的时候都必须运行在 Web 服务器上，而这些框架本身自带了一个小型 Web 服务器，但只用于开发和测试`

- `uWSGI 是一个 Web 服务器，它实现了 WSGI、uwsgi、HTTP 等协议，所以我们把使用 Web 框架编写好的服务部署在 uWSGI 服务器上是可以直接对外提供服务的`

- `Nginx 同样是一个 Web 服务器，但它相比 uWSGI 可以提供更多的功能，比如反向代理、负载均衡、缓存静态资源、对 HTTP 请求更加友好，这些都是 uWSGI 所不具备、或者不擅长的。所以我们在将 Web 服务部署在 uWSGI 之后，还要在前面再搭一层 Nginx。此时 uWSGI 就不再暴露 HTTP 服务了，而是暴露 TCP 服务，因为它是和 Nginx 进行通信，使用 TCP 会更快一些，Nginx 来对外暴露 HTTP 服务`
- `uwsgi 是 Nginx 和 uWSGI 通信所采用的协议，我们说 uWSGI 是和 Nginx 对接，Nginx 接收到用户请求时，如果是请求的是静态资源、那么会直接返回；请求的是动态资源，那么会将请求转发给 uWSGI，然后再由 uWSGI 调用相应的 Web 服务进行处理，处理完毕之后将结果交给 Nginx，Nginx 再返回给客户端。而 uWSGI 和 Nginx 之所以能交互，也正是因为它们都支持 uwsgi 协议，Nginx 中 HttpUwsgiModule 的作用就是与 uWSGI 服务器进行交互。`
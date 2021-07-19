# Session 分布式方案

## 基于 nfs(net filesystem) 的 Session 共享

将共享服务器目录 mount 各服务器的本地 session 目录，session 读写受共享服务器 io 限制，**不能满足高并发**。

## 基于关系数据库的 Session 共享

这种方案普遍使用。使用关系数据库存储 session 数据，对于 mysql 数据库，建议使用 heap 引擎。这种方案性能取决于数据库的性能，在高并发下容易造成表锁（虽然可以采用行锁的存储引擎，性能会下降），并且需要自己实现 session 过期淘汰机制。

## 基于 Cookie 的 Session 共享

这种方案也在大型互联网中普遍使用，将用户的 session 加密序列化后以 cookie 的方式保存在网站根域名下（比如 [taobao.com](https://www.funtl.com/zh/interview/taobao.com)），当用户访问所有二级域名站点式，浏览器会传递所有匹配的根域名的 cookie 信息，这样实现了用户 cookie 化 session 的多服务共享。此方案能够节省大量服务器资源，缺点是存储的信息长度受到 http 协议限制；cookie 的信息还需要做加密解密；请求任何资源时都会将 cookie 附加到 http 头上传到服务器，占用了一定带宽。

## 基于 Web 容器的 Session 机制

利用容器机制，通过配置即可实现。

## 基于 Zookeeper 的分布式 Session 存储

## 基于 Redis/Memcached 的 Session 共享存储

这些 key/value 非关系存储有较高的性能，轻松达到 2000 左右的 qps，内置的过期机制正好满足 session 的自动实效特性。
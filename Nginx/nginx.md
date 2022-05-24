  

# 概述

`Nginx` 是开源、高性能、高可靠的 `Web` 和反向代理服务器，而且支持热部署，几乎可以做到 7 * 24 小时不间断运行，即使运行几个月也不需要重新启动，还能在不间断服务的情况下对软件版本进行热更新。性能是 `Nginx` 最重要的考量，其占用内存少、并发能力强、能支持高达 5w 个并发连接数，最重要的是， `Nginx` 是免费的并可以商业化，配置使用也比较简单。

# 特点

- 高并发、高性能；
- 模块化架构使得它的扩展性非常好；
- 异步非阻塞的事件驱动模型这点和 `Node.js` 相似；
- 相对于其它服务器来说它可以连续几个月甚至更长而不需要重启服务器使得它具有高可靠性；
- 热部署、平滑升级；
- 完全开源，生态繁荣；

# 作用

Nginx 的最重要的几个使用场景：

1. 静态资源服务，通过本地文件系统提供服务；
2. 反向代理服务，延伸出包括缓存、负载均衡等；
3. `API` 服务， `OpenResty` ；

对于前端来说 `Node.js` 并不陌生， `Nginx` 和 `Node.js` 的很多理念类似， `HTTP` 服务器、事件驱动、异步非阻塞等，且 `Nginx` 的大部分功能使用 `Node.js` 也可以实现，但 `Nginx` 和 `Node.js` 并不冲突，都有自己擅长的领域。 `Nginx` 擅长于底层服务器端资源的处理（静态资源处理转发、反向代理，负载均衡等）， `Node.js` 更擅长上层具体业务逻辑的处理，两者可以完美组合。

# 特性

## 1.反向代理

只知道访问的代理服务器，不知道实际访问的机器

### 代理

代理是在服务器和客户端之间假设的一层服务器，代理将接收客户端的请求并将它转发给服务器，然后将服务端的响应转发给客户端。

不管是正向代理还是反向代理，实现的都是上面的功能。



## 2.负载均衡

提供路由功能(根据不同算法进行路由)

```
			# 反向代理配置
        upstream server_list{
                # 这个是tomcat的访问路径
                 server localhost:8081 weight=1;
                 server localhost:8082 weight=1;
        }
    server {
        listen       8080;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;
			##默认页面
       # location / {
           # root   html;
           # index  index.html index.htm;
       # }
       //配置的路由代理
        location / {
            root   html;
            index  index.html index.htm;
            proxy_pass http://server_list/;
        }

```

### 几种方式

#### 轮询(默认)

每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器挂掉了，能自动剔除。

#### weight 权重

weight 代表权重，默认为1,权重越高被分配的客户端越多

指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。

```
# 反向代理配置
upstream server_list{
    # 这个是tomcat的访问路径
    server localhost:8080 weight=5;#采用权重分配策略
    server localhost:9999 weight=1;
}
```

#### ip_hash

每个请求按访问ip的hash值分配，这样每个访问客户端会`固定访问一个后端服务器`，

可以解决会话Session丢失的问题

```
upstream backserver { 
    ip_hash;  #采用ip_hash策略
    server 127.0.0.1:8080; 
    server 127.0.0.1:9090; 
}
```

#### 最少连接

请求会被转发到连接数最少的服务器上(`谁最闲就把请求给谁来处理`)

```
upstream backserver { 
    least_conn; # 配置最小连接分配策略
    server 127.0.0.1:8080; 
    server 127.0.0.1:9090; 
}
```

db.order_current.find({"useTime":{$gt: NumberLong("1405219591000")}}).count()
* [【深入浅出Nginx系列】Nginx入门？看这一篇就够了（概念篇）](https://blog.csdn.net/weixin_47025166/article/details/126041580)
* [【深入浅出Nginx系列】Nginx入门？看这一篇就够了（实战篇）](https://blog.csdn.net/weixin_47025166/article/details/126047843?csdn_share_tail=%7B%22type%22%3A%22blog%22%2C%22rType%22%3A%22article%22%2C%22rId%22%3A%22126047843%22%2C%22source%22%3A%22weixin_47025166%22%7D&ctrtid=FklNh)



# 一、Mac安装Nginx

* [Homebrew官网](https://brew.sh/)



## 1. 检查电脑是否有nginx

```sh
$ brew search nginx
==> Formulae
nginx ✔              # 说明已经安装
```



## 2. brew安装nginx

```sh
# 安装
$ brew install nginx

# 重装
$ brew reinstall nginx
```



## 3. 查看nginx的信息

```sh
# 查看nginx版本
$ nginx -v
nginx version: nginx/1.25.5

# 查看nginx详细信息
$ brew info nginx
==> nginx: stable 1.25.5 (bottled), HEAD
HTTP(S) server and reverse proxy, and IMAP/POP3 proxy server
https://nginx.org/
Installed
/opt/homebrew/Cellar/nginx/1.25.5 (26 files, 2.4MB) *
......
```



## 4. 查看nginx.conf配置文件 

```sh
# 测试配置文件的正确性
$ nginx -t
nginx: the configuration file /opt/homebrew/etc/nginx/nginx.conf syntax is ok
nginx: configuration file /opt/homebrew/etc/nginx/nginx.conf test is successful
```



## 5. 查看帮助

```sh
$ nginx -h
```



# 二、运行Nginx

```sh
# 启动
$ nginx

# 关闭
$ nginx -s stop

# 重新启动，热启动，修改配置重启不影响线上
$ nginx -s reload
```

**浏览器访问localhost:8080**：http://localhost:8080/

**如果页面出现---- Welcome to nginx! ------即代表成功**

【默认端口,如需更改请求配置文件conf更改】



## 1. 查看是否运行nginx

```sh
# 查看状态
$ ps -ef|grep nginx
```

![](images/006.png)



## 2. `brew services start nginx` 对比

* [MacOS - brew 里运行 nginx 和直接运行 nginx ，有什么区别吗？](https://v2ex.com/t/812279)

```sh
# 启动服务
$ brew services start nginx

# 关闭
$ brew services stop nginx

# 重启
$ brew services restart nginx
```

brew services 是托管给 launchd 的，相当于系统服务。



# 三、文件目录

Homebrew 默认安装路径如下：

- macOS ARM：`/opt/homebrew`
- macOS Intel： `/usr/local`

```sh
# 显示 Homebrew Cellar 路径
$ brew --cellar
/opt/homebrew/Cellar

# 显示 Homebrew Caskroom 路径
$ brew --caskroom
/opt/homebrew/Caskroom
```



```sh
# nginx路径
$ which nginx  
$ where nginx
/opt/homebrew/bin/nginx

# 配置文件路径
$ nginx -t
nginx: the configuration file /opt/homebrew/etc/nginx/nginx.conf syntax is ok
nginx: configuration file /opt/homebrew/etc/nginx/nginx.conf test is successful

# 查看nginx详情，也可以看到路径
$ brew info nginx
```



# 四、修改index.html文件

nginx允许成功后，打开 http://localhost:8080/ ，说明成功了。

![](images/001.png)

a. 先找到nginx路径 `$ where nginx` ，找到 `/opt/homebrew/bin/nginx` :

![](images/002.png)

可以发现文件是快捷入库，需要找到源文件

![](images/003.png)

源文件地址：`/opt/homebrew/var/www`

![](images/004.png)

我们找到源文件，修改试试：

![](images/005.png)

# 五、配置文件设置

```sh
# 测试配置文件的正确性
$ nginx -t
nginx: the configuration file /opt/homebrew/etc/nginx/nginx.conf syntax is ok
nginx: configuration file /opt/homebrew/etc/nginx/nginx.conf test is successful
```

![](images/007.png)



大致内容

```sh
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       8082;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
    include servers/*;
}
```





# 六、Vue项目部署到Nginx

Vue打包，看其他地方



https://juejin.cn/post/6844904096973979662














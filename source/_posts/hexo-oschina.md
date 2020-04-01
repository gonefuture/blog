---
title: oschina git仓库提交webHook触发hexo博客更新 （转载自zooooooooy）
---

#### 创建一个项目
![项目界面](/images/TIM截图20180428143154.png)
1. 将服务器上的代码整个打包传到项目上来，使用如下命令
	1.1	git init
	1.2 git add .
	1.3 git commit -m 'init'
	1.4 git remote add origin 'your remote ssh address'
2. 后续就需要配置openresty，解决自动提交webHook触发脚本执行的功能
	2.1 openresty的安装可以参照官网
	2.2 启动还是跟nginx一样的，start nginx
	2.3 配置nginx的配置，执行lua脚本
	2.4 lua脚本中需要验证密码这一块，我暂时没处理，oschina的验证规则是在post body里面有一个参数是password，可以和自己的配置的webHook秘钥进行匹配就可以
	2.5 lua脚本只是简单执行依据脚本命令
	```
    os.execute("bash /opt/lua/hexo.sh");
    ngx.say("OK")
    ngx.exit(200)
    
    ```
    2.6 主要是后续的shell命令，操作的流程是获取git 远程的代码，执行hexo的generate生成静态文件
    ```
    #! /bin/bash
    blog_dir=/opt/blog
    branch=master
    hexo=/root/.nvm/versions/node/v9.11.1/bin/hexo

    cd $blog_dir
    git reset --hard origin/$branch
    git clean -f
    git pull origin master

    cd $blog_dir
    $hexo generate
    echo "success"

    ```
    2.7 静态文件的目录是public，前面装好的openresty也可以用作服务代理，将域名的root访问路径指向到public目录即可
    2.8 漏了nginx的转发配置了，有两个  一个是webHook地址，一个是服务器的请求地址
    ```
    server {
        listen       80;
        server_name  db101.cn;

        lua_code_cache off;

        location / {
            root   /opt/blog/public;
            index  index.html index.htm;
        }

        location /hexo/github {
            content_by_lua_file /opt/lua/hexo.lua;
        }
    }

	```
_ _ _
后续就可以当更新了source下面的md文件。可以直接通过域名访问到最新的更新了。也可以修改其他的主体或者配置一类的文件。当有push操作时触发webHook操作，调用服务器的地址，最终更新页面。



上述步骤参考的网址 > https://blog.liaol.net/
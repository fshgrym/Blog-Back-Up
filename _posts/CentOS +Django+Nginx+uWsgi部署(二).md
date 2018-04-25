---
title: CentOS +Django+Nginx+uWsgi部署(二)
date: 2018-4-14 19:08:32
categories:
tags:
     - Python
     - Django
     - Nginx
---
> 收集静态文件

打开项目的seeting文件，加入到文件后面。

```
STATIC_ROOT = os.path.join(BASE_DIR, "static/")
```

进入虚拟环境，cd到项目文件manage.py下运行命令，会自动收集Django注册过app的静态文件。

```
python manage.py collectstatic
```

>uwsgi

在项目目录下新建conf/uwsgi/uwsgi.ini

<!--more-->

```
# mysite_uwsgi.ini file`
    [uwsgi]

    # Django-related settings
    # the base directory (full path)
    chdir           = /home/fsh/fsh/project/blog_project_app/
    # Django's wsgi file
    module          = blog_project_app.wsgi
    # the virtualenv (full path)

    # process-related settings
    # master
    master          = true
    # maximum number of worker processes
    processes       = 4
    threads         = 2
    # the socket (use the full path to be safe
    socket          = 127.0.0.1:8000
    # ... with appropriate permissions - may be needed
    # chmod-socket    = 664
    # clear environment on exit
    vacuum          = true
    virtualenv = /home/fsh/.virtualenvs/Blog_app

    python-autoreload=1

    logto = /tmp/mylog.log

    stats = %(chdir)/conf/uwsgi/uwsgi.status
    pidfile = %(chdir)/conf/uwsgi/uwsgi.pid
```
```
说明
python-autoreload=1  python文件修改自动重启nginx
chdir： 表示需要操作的目录，也就是项目的目录
module： wsgi文件的路径
processes： 进程数
virtualenv：虚拟环境的目录
stats = %(chdir)/conf/uwsgi/uwsgi.status #会自动生成
pidfile = %(chdir)/conf/uwsgi/uwsgi.pid #会自动生成
两个文件可以让uwsgi服务启动(start)停止(stop)重新装载(reload)
```

运行uwsgi

```
uwsgi -i ~/项目/conf/uwsgi/uwsgi.ini &
```

重载

```
uwsgi --reload ~/项目/conf/uwsgi/uwsgi.pid
```

停止

```
uwsgi --reload ~/项目/conf/uwsgi/uwsgi.pid
```

> Nginx

在项目下面新建conf/nginx/blog.conf文件

```
# the upstream component nginx needs to connect to
upstream django {
# server unix:///path/to/your/mysite/mysite.sock; # for a file socket
server 127.0.0.1:8000; # for a web port socket (we'll use this first)
}
# configuration of the server

error_log  /home/fsh/fsh/project/error.log;
server {
# the port your site will be served on
listen      80;
# the domain name it will serve for
server_name 118.24.154.138 ; # substitute your machine's IP address or FQDN
charset     utf-8;

# max upload size
client_max_body_size 75M;   # adjust to taste

# Django media
location /media  {
    alias /home/fsh/fsh/project/blog_project_app/media;  # 指向django的media目录
}
location /static {
    alias /home/fsh/fsh/project/blog_project_app/static; # 指向django的static目>录
}

# Finally, send all non-media requests to the Django server.
location / {
    uwsgi_pass  django;
    include     uwsgi_params; # the uwsgi_params file you installed
}
}
```

软连接
```
sudo ln -s ~/blog/conf/nginx/blog.conf /etc/nginx/conf.d/
```

启动Nginx

```
sudo /usr/sbin/nginx
```

403错误

更改/etc/nginx/nginx.conf里面的`user nginx`改成`user root`

500错误

一般是路径问题，在uwsgi的错误log里面查看。

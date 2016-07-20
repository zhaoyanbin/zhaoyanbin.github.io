---
layout: post
title: "uwsgi+django服务配置"
date: 2016-07-20 13:51:49 +0800
comments: true
categories: django uwsgi
---
uwsgi+django的服务配置和关键配置文件说明
<!--more-->
#1、django目录结构
+ Proejct_Name
    + djangosite
        + \_\_init\_\_.py
        + manage.py
        + settings.py
        + wsgi.py(***uwsgi启动配置***)
        + urls.py(***import每个app的urls.py***)
        + model.py(***放置所有app共用model***)
        + views.py(***放置所有app共用view***)
        + apps/
            + myapp1
            	+ model.py
            	+ views.py
            	+ urls.py
            	+ static/
            + myapp2
        + tests/(***project级别的测试，针对每个app的测试还要有单独的测试代码***)
        + static/
            + themes1/ 
                + css/
                + js/
                + images/
            + themes2/
                + css
                + js/
                + images/
        + templates
            + base.html
            + home.html
            + myapp1
                + ...
            + myapp2
                + ...
	+ src(***src代码库***)
		+ third\_party(***加载第三方模块，为了避免版本冲突，将使用的第三方库或者期望覆盖的库放到这里***)
			+ python\*.\*
				+ site\_packages
		+ utils
		+ tools
			+ path\_tools.py(***这里将third\_party的site\_packages***)
	+ uwsgi\_run
		+ real\_run\_systemd
		+ uwsgi.service
	+ uwsgi.xml

#2、settings.py文件的配置说明
> 1.在path\_tools.py添加{third\_party lib path}到sys_path
<pre><code>def _libs_path():
    libs_path = "../third_party/lib/python2.7/site-packages/"
    libs_path = os.path.abspath(os.path.join(os.path.dirname(__file__),libs_path))
    sys.path.insert(0, libs_path)
    return sys.path
LIBS_PATH = _libs_path()</code></pre>

> 2、添加third\_party库的引用：
<pre><code>from Project_Name.src.tools.path_tools import LIBS_PATH as _</code></pre>

> 3、其它设置：
<pre><code>BASE_DIR = os.path.abspath(os.path.dirname(__file__))
ROOT_DIR = os.path.dirname(BASE_DIR)
SRC_DIR = os.path.join(ROOT_DIR, 'src')
RUN_DIR = os.path.join(SRC_DIR, 'run')
LOG_DIR = os.path.join(SRC_DIR, 'logs')
DB_DIR = os.path.join(SRC_DIR, 'logs')
DEBUG = True
TEMPLATE_DEBUG = DEBUG
......
# Absolute path to the directory static files should be collected to.
# Don't put anything in this directory yourself; store your static files
# in apps' "static/" subdirectories and in STATICFILES_DIRS.
# Example: "/home/media/media.lawrence.com/static/"
STATIC_ROOT = os.path.join(RUN_DIR, 'static')
# URL prefix for static files.
# Example: "http://media.lawrence.com/static/"
STATIC_URL = '/static/'
STATICFILES_DIRS = (
    # Put strings here, like "/home/html/static" or "C:/www/django/static".
    # Always use forward slashes, even on Windows.
    # Don't forget to use absolute paths, not relative paths."
    os.path.join(BASE_DIR, 'static'),
)</code></pre>

#3、wsgi.py文件的配置说明
<pre><code>from Project_Name.src.tools.path_tools import LIBS_PATH as _ #加载第三方库路径
import os
import sys
from django.core.wsgi import get_wsgi_application
DJANGO_LATEST_PATH = '/opt/tiger/ss_lib/django-latest'
sys.path.insert(0, DJANGO_LATEST_PATH) #加载django路径
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "Project_Name.djangosite.settings") ＃设置setting加载路径
application = get_wsgi_application()</code></pre>

#4、systemd服务配置与启动
>1、systemd启动脚本real\_run\_systemd
<pre><code>#! /bin/bash
ulimit -c 1000000
ulimit -n 50000
export DJANGO_SETTINGS_MODULE="djangosite.settings"
cd /your/path/to/Project #{Project_Name}
prog_name="{your prog name}"
echo "`date '+%Y-%m-%d %H:%M:%S'`  `hostname` : $prog_name start" >> /your/log/parent/dir/${prog_name}_svc.log
exec /your/uwsgi/service/location/uwsgi -x uwsgi.xml  1>/dev/null 2>>/your/error/log/dir/${prog_name}_err.log</code></pre>

>2、systemd服务配置uwsgi.service
<pre><code>ExecStart=/your/online/real_run_systemd/dir/real_run_systemd
ExecReload=/bin/kill -s HUP $MAINPID
Restart=always
ExecStop=/bin/kill -s INT $MAINPID
KillMode=control-group
KillSignal=INT
[Install]
WantedBy=default.target</pre>

关于关闭DEBUG模式和systemctl后续补充。

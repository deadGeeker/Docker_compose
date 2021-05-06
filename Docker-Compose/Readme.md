### 安装Docker-Compose
[Docker-Compose官方网站安装指导请点击这里](https://docs.docker.com/compose/install/)

---
##### 安装前提
+ 以我的Linux虚拟机为例，必须安装Docker-Engine以后才能安装和运行Docker-Compose
###### 关于Docker-Engine的安装和命令教程可以移步我的[github项目](https://github.com/deadGeeker/Docker_pathtoGod/blob/main/%E5%AD%A6%E4%B9%A0%E4%B9%8B%E8%B7%AF/%E7%9B%AE%E5%BD%95.md)，项目中有什么需要进步的地方欢迎各位码友指出并私信，谢谢。  

##### 官方下载命令
+ $ sudo curl -L "https://github.com/docker/compose/releases/download/1.29.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
+ 下载命令拆分分析：
+ + $ curl 根据url语法在命令行下工作的文本传输工具，可视为下载工具
+ + $ curl -L 会自动跳转到后面跟着的URL所对应的网址
+ + $ uname -s 会返回当前操作系统的类别(Linux Windows Mac)
+ + $ uname -m 会返回当前操作系统的类别(x86_64[64位] i686[32位])
+ + /usr/local/bin Linux系统下环境变量PATH指向的路径 通过指令 $ env | grep PATH 可以得出

##### 将可执行权限应用于二进制文件：
+  sudo chmod +x /usr/local/bin/docker-compose

##### 测试安装：
+ $ docker-compose --version
+ ![avatar](/png/https://github.com/deadGeeker/Docker_compose/blob/main/Docker-Compose/png/1.PNG)

### Docker-Compose的相关概念
---
+ Compose是一个工具，这个工具可以创建和运行Docker的多容器，使用一个命令，就可以从配置中创建并启动所有服务
+ 举一个例子：在Docker中你要运行100个服务，意味着需要100个容器跑起来
+ 在Docker中跑一个容器需要经历三个步骤：
+ + 拉取父镜像   
+ + 根据需求编写DockerFile文件
+ + 打包生成子镜像并运行
+ 假设这100个服务的父镜像各不相同，服务与服务之间没有任何依赖
+ 那就意味着你需要操作300个步骤才能满足跑100个服务的需要
+ 更何况步骤中的编写DockerFile文件是需要时间验证跟反复确认的
+ 从效率和时间出发，Compose工具诞生并被引入到Docker的多服务部署的场景中
---
### 如何理解Docker-Compose的用法
---
+ 这边我准备了一个小项目，项目的目录结构如下所示：
+ docker_Compose/  
├── docker-compose.yml  
└── web  
&nbsp;&nbsp;├── Dockerfile  
&nbsp;&nbsp;├── requirements.txt  
&nbsp;&nbsp;└── web.py  

+ 项目的需求是运行一个web应用，这个web应用是基于Flask框架，并使用了redis的服务。
+ 看到Flask框架，或者说这个web应用的源代码是.py文件，那么一定有python镜像，
+ 由顶层设计思想出发，先把最上层的业务类代码先写出来，代码如下：
+ + <font color=blue>\# web.py:</font>
+ + from flask import Flask
+ + from redis import Redis

+ + app = Flask(__name__)
+ + <font color=blue>\# redis默认端口是6379</font>
+ + redis = Redis(host='redis', port=6379)

+ + @app.route('/')
+ + def hello():  
        redis.incr('number')
        return 'hello docker-compose! # %s' % redis.get('number')

+ + if __name__ == "__main__":
+ + <font color=blue>\# 容器的端口是80</font>
+ + app.run(host="0.0.0.0", port=80, debug=True)
+ 1. 这段代码最终是在浏览器页面中展示函数hello()中的内容，在假设redis镜像已经引入的前提下：
+ 2. 要实现一个容器去跑这个web.py，还得引入python镜像，然后下载python包(flask、redis)
+ 3. 接着运行web.py中的代码，从而启动flask和redis的相关服务。
+ 根据上面第二步和第三步的思路，可以把Dockerfile的框架布置如下：
---
+ + \# <font color=blue>引入python镜像</font>
+ + FROM python:3.8
+ + \# <font color=blue>下载python包(flask、redis)</font>
+ + RUN pip install flask==1.1.2
+ + RUN pip install redis==3.5.3
+ + \# <font color=blue>运行web.py</font>
+ + CMD python web.py
---
+ 为了保证Dockerfile中的最后一层CMD python web.py能够正确执行
+ 由于web.py在/web目录下，所以要指定工作目录WORKDIR的路径为/web
+ 而且从build Dockerfile的工作原理出发，Dockerfile中的每一条命令
+ 都是一层layer文件，尽可能用少的layer构建镜像能够更精简，为此，要
+ 尽可能的把具有相同功能(命令)的layer层合并到一层，所以，最终的Dockerfile如下：
---
+ + \# <font color=blue>引入python镜像</font>
+ + FROM python:3.8
+ + \# <font color=blue>指定工作目录WORKDIR</font>
+ + WORKDIR /web
+ + \# <font color=blue>下载python包(flask、redis)</font>
+ + RUN pip install -r requirements.txt
+ + \# <font color=blue>运行web.py</font>
+ + CMD python web.py
---
+ 上面的requirements.txt中写入的是web应用需要用到的python包
+ + flask
+ + redis
+ 这样就保证了在后面web应用要增加功能而需要引入额外的python包时
+ 可以直接在requirements.txt中声明即可

+ 那么到这里项目就全部结束了吗，当然是不可能的，不要忘了
+ 上面的Dockerfile的前提redis镜像还没有引入，而且跟我们的docker-compose.yml
+ 还没有产生联系，那么又怎么算是一个Docker-Compose项目？？？

+ 关于yaml的用法介绍请跳转[官方文档](https://docs.docker.com/compose/compose-file/#reference-and-guidelines)
+ docker-compose.yml内容如下：
---
+ + services:
+ + redis:
    image: redis
+ + web:
    build:
      context: /home/cyrus/demo/docker_Compose/web
    depends_on:
    - redis
    ports:
    - 8001:80/tcp
    volumes:
    - /home/cyrus/demo/docker_Compose/web:/web:rw  

+ + \# 目前compose文件支持的版本为2.2~3.3，不在这个区间的版本在执行docker-compose up时会报错
+ + version: '3.0'
---
+ 万事俱备，只欠东风，让我们的代码跑起来！！！
+ 在docker_Compose/目录下执行：
+ $ docker-compose up
+ ![avatar](https://github.com/deadGeeker/Docker_compose/blob/main/Docker-Compose/png/4.PNG)  
+ 分析报错原因：<font color=red>Could not open requirements file</font>
+ 原来是在执行Dockerfile镜像时，会生成一个临时容器，临时容器在整个Dockerfile文件执行完毕时才 + 会退出，requirements file在本地文件中，容器文件中没有，所以我们要将requirements file复制到
+ 临时容器中，在Dockerfile中WORKDIR前加入：
+ + COPY ./ /web/
+ 然后再次让我们的代码跑起来！！！
+ ![avatar](https://github.com/deadGeeker/Docker_compose/blob/main/Docker-Compose/png/5.PNG)
+ 可以看到，运行成功，服务跑起来了，让我们在浏览器中输入127.0.0.1：8001
+ ![avatar](/png/https://github.com/deadGeeker/Docker_compose/blob/main/Docker-Compose/png/2.PNG)
+ 最后，让我们查看一下生成的镜像和容器，关于这里的指令有不清楚的可以移步到我的github项目[Docker学习之路](https://github.com/deadGeeker/Docker_pathtoGod/blob/main/%E5%AD%A6%E4%B9%A0%E4%B9%8B%E8%B7%AF/%E7%9B%AE%E5%BD%95.md)
+ 查看生成的镜像
+ ![avatar](https://github.com/deadGeeker/Docker_compose/blob/main/Docker-Compose/png/6.PNG)
+ 查看生成的容器
+ ![avatar](https://github.com/deadGeeker/Docker_compose/blob/main/Docker-Compose/png/7.PNG)
---
使用compose一键生成多应用服务的讲述到此为止，如文档中有需要提高的地方，欢迎各位码友指出，中国的软件业兴旺就靠各位了！！！

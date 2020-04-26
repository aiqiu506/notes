### Docker 中10大常用的命令

```
|--------------------------------------------
  Author:       intro                        
  CreateAt:   2018-09-12 22:01:03          
|--------------------------------------------
  
```

#### 1、 run 启动一个容器 通常会基于某一个image来启动一个容器


用法: 
``` 
    docker run -it [--name 容器名称] image:基于的镜像  进入容器时执行的命令 
```


参数:

    -i  启动的是一个交互式容器
    
    -t  模拟一个终端方式[其实就是对应交互一个文件系统]
    
    --name  给容器取一个别名
    
    -p   将宿主机端口与容器中导出的端口进行映射
    
    -v  卷  用于将宿主机中的文件或文件夹共享到容器中

例子:

> docker run -it --name test ubantu /bin/bash

   基于ubantu的镜像启动一个名为test的容器并以bash与之交互

----


#### 2、build 通过Dockerfile文件构建一个image(镜像)


用法: 
``` 
  docker build -t [images name] [Dockerfile文件所在路径]
```
参数:
    
    - t  构建的新镜像的名称


​    
例子:

> docker build -t new-image ./

   基于当前目录下的Dockerfile文件来构建一个名为new-image的镜像

   

#### 3、rmi 删除一个image(镜像)


用法: 
``` 
  docker rmi [images name] 
```
参数:
    
    - f  强制删除


​    
例子:

> docker rmi -f new-image

   强制删除一个名为new-image的镜像


#### 4、rm 删除一个container(容器)


用法: 
``` 
  docker rm [images name] 
```
参数:
    
     - f  强制删除


​    
例子:

> docker rm -f test

   强制删除一个名为test的容器


#### 5、exec 进入一个正在执行的容器中


用法: 
``` 
  docker exec -it [container name] [command 要执行的命令]
```
参数:
    
    与run 基本一致


​    
例子:

> docker exec -it test /bin/bash

   通过bash 进入当前运行的test容器中


#### 6、inspect 查看容器的配置


用法: 
``` 
  docker inspect  [container name]
```

例子:

> docker inspect test

   查看test的配置信息,返回一个json格式的对象

#### 7、top 查看容器中正在执行的进程(并不进入容器中)


用法: 
``` 
  docker top   [container name]
```

例子:

> docker top test

   查看test容器中正在运行的进程

#### 8、ps  查看容器列表


用法: 
``` 
  docker ps 
```
 参数:
    
    - a  所有容器,包含当前非运行状态的容器 


​    
例子:

> docker ps -a

   查看宿主机中所有的容器  

#### 9、images  查看镜像列表


用法: 
``` 
  docker images 
```
 参数:
    
    - a  所有镜像,包含中间层镜像


​    
例子:

> docker  images  -a

   查看宿主机中所有的镜像 

#### 9、start/stop  启动/关闭 一个已经存在的容器


用法: 
``` 
  docker start/stop 容器名称
```

例子:

> docker  start  test

   启动test容器 

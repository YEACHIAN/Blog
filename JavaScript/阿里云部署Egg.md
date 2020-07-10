## 创建私有镜像库

1. 进入[容器镜像服务控制台](https://cr.console.aliyun.com/)
2. 左上角选择区域
3. 左侧菜单->容器镜像服务->默认实例->镜像仓库
4. 点击`创建镜像仓库`
5. 在弹出的对话框中配置镜像仓库，设置`命名空间`、`仓库名称`、`摘要`和`仓库类型`, 点击`下一步`
6. 在设置代码源对话框中，将代码源设为`本地仓库`, 点击`创建镜像仓库`

## 管理私有仓库

点击刚创建的仓库右侧的`管理`, 可以看到`操作指南`会有很多命令行的操作

## 修改项目package.json

```sh
"start": "egg-scripts start --env=prod"
```

## 使用多阶段构建制作本地镜像

Docker v17.05 开始支持多阶段构建 ( multistage builds ), 可以提升速度缩减大小
项目根目录创建`Dockerfile`文件
分为2个阶段构建

- base阶段进行npm包的安装
- build阶段拷贝文件

```sh
# FROM node:8-alpine as base
# WORKDIR /usr/src/app
# COPY package.json .
# RUN npm install --registry=https://registry.npm.taobao.org

FROM node:8-alpine as build
ENV TZ Asia/Shanghai
WORKDIR /usr/src/app
# 从其它阶段或镜像复制文件COPY --from=xxx 第一个参数只能是绝对地址 第二个参数是WORKDIR指定的相对地址
# COPY --from=base /usr/src/app/node_modules ./node_modules
COPY ./node_modules ./node_modules
COPY . .
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo Asia/Shanghai > /etc/timezone
EXPOSE 7001
CMD npm start
```

项目根目录创建`.dockerignore`文件用来排除构建镜像时不需要的文件或目录

```
node_modules/*
.git/*
testCI/*
assets/*
documents/*
npm-debug.log
log/*
run/*
logs/*
```

开始构建

```sh
$ sudo docker build -t 仓库地域/命名空间/仓库名称:latest .
$ sudo docker build --target xxx -t username/imagename:tag . # 只想构建xxx阶段的镜像
$ sudo docker image ls | grep xxx # 查看所有镜像以验证
$ docker tag <ImageId> 仓库地域/命名空间/仓库名称:TAG # 注意如果只改tag 会生成ImageId一模一样tag不一样的镜像
```

容器的一些基本操作

```sh
$ sudo docker run -p 7004:7001 --rm 仓库地域/命名空间/仓库名称:latest # 运行容器 运行结束则删除容器 镜像expose的是7001 对外展现为7004
$ sudo docker container ls # 列出所有正在运行的容器
$ sudo docker container ls -a # 列出所有容器包括运行的和结束的
$ docker container stop # 终止启动的容器
$ docker container start # 启动处于终止状态的容器
$ docker container restart # 将一个运行态的容器终止，然后再重新启动它
```

## 自动构建本地镜像并上传到私有仓库

创建Makefile并登陆docker私有仓库

```sh
$ touch 项目根目录/Makefile
# 域名就是仓库所在地区的域名
# 用于登录的用户名为阿里云账号全名，密码为开通服务时设置的密码
$ sudo docker login --username=xxxxxx@xx.com registry.cn-xxxxxx.aliyuncs.com
```

填入Makefile的内容

```sh
.PHONY: build

NAME=仓库名称
REGISTRY=registry.cn-仓库地域.aliyuncs.com
NAMESPACE=命名空间
TAG=latest
TOKEN=触发器token 

# 完整格式: registry.cn-仓库地域.aliyuncs.com/命名空间/仓库名:TAG
build:
	echo building ${NAME}:${TAG}
	docker build -t ${REGISTRY}/${NAMESPACE}/${NAME}:${TAG} .
	docker push ${REGISTRY}/${NAMESPACE}/${NAME}:${TAG}
	# curl https://cs.console.aliyun.com/hook/trigger?token=${TOKEN} # 暂时不用
```

运行Makefile使其自动构建本地镜像并上传至私有仓库

```sh
# sudo是因为需要读取~/.docker/config.json需要权限
sudo make
```

### 验证镜像是否已经上传

点击仓库右侧的`管理`进入镜像仓库详情页，单击左侧导航栏中的`镜像版本`

## 创建私有镜像仓库登录密钥类型的密钥

1. [容器服务管理控制台](https://cs.console.aliyun.com/)->应用配置->[管理字典](https://cs.console.aliyun.com/#/k8s/secret/list)
2. 选择所需的集群和命名空间，单击右上角的`创建`
3. `类型`选择`私有仓库登陆密钥`

## 通过私有镜像仓库创建应用

1. [容器服务管理控制台](https://cs.console.aliyun.com/)->应用->[无状态](https://cs.console.aliyun.com/#/k8s/deployment/list), 进入**无状态Deployment**页面
2. 选择所需的集群和命名空间，单击右上角的**使用镜像创建**，会经历如下几个步骤：

### 一、应用基本信息

在**应用基本信息**填写信息，其中`副本数量`就是Pod数量

### 二、容器配置

#### 基本配置

`镜像Tag`一定要用鼠标点选，不要自己键盘输入！不然拉取不到镜像
如果看不到tag说明镜像地址是错误的， 把`-vpc`删了，使用公网镜像地址!
更新：一定要用**专有网络**镜像地址(带vpc)而不是公网或经典网络(内网)，不然导致镜像组拉取镜像失败

#### 端口设置

无状态里的端口不用手动设置，会自动设置的，如果你在服务进行了端口映射如`-p 7004:7001`，这里就会出现所有的端口

#### 容器配置的健康检查解释

容器的健康检查有2种

1. 存活检查(liveness) -> 重启容器
2. 就绪检查(Readiness) -> 接收流量

健康检查又分为3种方式

1. HTTP请求 -> GET请求
2. TCP连接 -> TCP Socket
3. 命令行 -> 探针检测命令

用的比较多的是HTTP请求

1. 协议
   - HTTP
   - HTTPS
2. 路径
   一般访问首页`/`
3. 端口(EXPOSE端口)
   - 端口号 1~65535
   - 端口名
4. HTTP头
   允许重复的header
5. 延迟探测时间initialDelaySeconds
   容器启动多久后开始探测, 默认3s后
6. 执行探测频率periodSeconds
   每次探测的间隔 >= 1s
7. 超时时间timeoutSeconds
   探测超时时间 >= 1s
8. 健康阀值
   连续多少次成功算成功 >= 1, 存活检查只能是1
9. 不健康阀值
   连续多少次失败算失败 >= 1 默认3

### 三、高级配置（创建服务和路由）

完成容器配置后，单击`下一步`，进行`高级配置`，分为3部分

1. 访问设置

   1. 服务Service（注解annotation支持负载均衡配置参数）
      1. 虚拟集群IP（ClusterIP）集群内的服务（外部不可访问，但可以通过路由访问）
      2. 节点端口（NodePort）可路由到ClusterIP，通过`:`可从集群外部访问
      3. 负载均衡（LoadBalancer）公网内网都可访问 可路由到ClusterIP、NodePort
   2. 路由Ingress（通过`镜像`创建应用时，仅能为一个服务创建路由）
      为后端Pod（容器组）配置路由规则

2. 伸缩设置（可选）

   根据

   ```
   容器组Pod
   ```

   的CPU、内存的占用调整Pod的数量

   - 指标（CPU、内存）
   - 触发条件（到达该百分比触发扩容）
   - 最大容器数量
   - 最小容器数量

3. 调度设置（可选）

   - 升级方式
     1. 滚动升级 rollingupdate （一般是滚动升级）
     2. 替换升级 recreate
   - 节点亲和性
   - 应用非亲和性

#### 创建服务和路由

类型选择虚拟集群IP（ClusterIP），勾选`实例间服务发现(Headless Service)`，服务端口填你要在集群内网暴露的端口，容器端口填EXPOSE的端口（注意服务端口不要跟其它服务的服务端口冲突了）
路由的端口直接填服务端口即可，路径填`/`，就可以通过填的域名进行外网访问（80端口的）

无状态创建后，也可以去[容器服务管理控制台](https://cs.console.aliyun.com/)->`路由与负载均衡`管理服务和路由

### 无状态服务端口总结

- 服务端口：在集群内网运行时的端口
- 容器端口：镜像端口、Dockerfile文件里EXPOSE的端口
- 无状态里的端口：不用设置，全删了也会自动设置的，看起来是用到的所有端口都会被添加
- 无状态里健康检查的端口：容器端口

### 四、创建完成

最后单击`创建`。创建成功后，默认进入创建完成页面，会列出应用包含的对象，您可以单击`查看应用详情`进行查看

### 创建触发器

在无状态详情里面，`创建触发器`，创建重新部署的触发器，得到触发器的token
修改`Makefile`，取消注释，替换token，然后重新部署

## 参考

[#11](https://github.com/TENCHIANG/blog/issues/11)
[Dockerfile构建容器镜像 - 运维笔记](https://www.cnblogs.com/kevingrace/p/6698596.html)
[运维-makefile的书写（节省dockerFile的批量构建的问题）](https://blog.csdn.net/u012983826/article/details/51866757)
[使用私有镜像仓库创建应用](https://help.aliyun.com/document_detail/86307.html)
[镜像创建无状态Deployment应用](https://help.aliyun.com/document_detail/87784.html)
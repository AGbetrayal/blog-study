# 如何创建vue项目
## 第一步: npm的安装
### 首先从官网下载nodejs,然后一直next,直到finish完成安装. 打开控制命令行程序(cmd), 检查是否正常: (1) node -v 查看node版本是否正常 (2) npm -v npm版本是否正常  需要注意的是: node虽然自带npm,但并不一定是最新版本,可以根据npm install -g npm 自行升级到最新版本
### npm默认是国外的镜像, 下载非常慢, 可以更改镜像来源(比如:改为淘宝等国内镜像) 通过安装nrm 来修改镜像来源, 安装nrm : npm install -g nrm 执行安装命令即可. 一般使用nrm的两个命令:(1) nrm list 可以查看能使用的镜像列表 (2) nrm use 镜像名 切换到那个镜像来源
## 第二步: 项目初始化
### 步骤一: 安装vue-cli
### npm install -g cnpm  //将npm 升级为 cnpm , cnpm比npm功能强大更好用 
### cnpm install vue //安装vue
### cnpm install vue-cli -g      //全局安装 vue-cli  
### 查看vue-cli是否安装成功 : vue list  //如果安装成功就可以通过vue命令创建vue项目了
## 
### 步骤二 : 通过vue命令创建vue项目
### 选定路径，新建vue项目, 可以在随便一个目录创建文件夹, 然后cd 到文件目录
### vue init webpack  ”项目名称“  (在该目录下进入cmd 命令行, 执行命令创建一个vue项目.)
### 注意点: 在执行创建项目命令时遇到:
```
...
User ESLint to lint your code ? no 
Set up unit tests ? no
Setup e2e tests with nighwatch ? no
```
都选择no, 别选这yes. 如果选择yes , 就是严格的格式, 在往后编写项目的时候会出现许多警告

### 现在已经创建好了，那就让项目先安装下依赖再运行一下，会出现下面的页面，操作指令是：
### (1) cnpm install 
### (2) cnpm run dev
### 出现 ****: http:// localhost:8080 则运行成功
### cnpm run build 为打包命令, 执行成功完成时会多一个dist的文件夹, 该文件夹是压缩过的, 放到线上只把这文件夹放上去即可


### 资料来源: https://www.jianshu.com/p/02b12c600c7b
### markdown使用教程: https://blog.csdn.net/m0_37447148/article/details/84968994
### markdown使用教程: https://blog.csdn.net/qq_41559171/article/details/88029787
### markdown使用教程: https://www.jianshu.com/p/b1183c39ad88

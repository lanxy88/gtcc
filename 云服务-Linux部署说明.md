# 云服务Linux部署说明

## 一、部署环境
### 1.Java运行时
#### 推荐支持版本：Java8+

## 二、云应用部署
### 1.SSH远程连接Linux服务器
### 2.拷贝云服务部署包至服务器
### 3.定位至包目录，解压zip包，例如：unzip center-1.0-SNAPSHOT-release.zip -d center
### 4.更改cfg中application.xml配置，注册中心、数据库、rabbitmq等工程自定义配置
### 5.赋予start.sh可执行权限，chmod +x start.sh
### 6.直接启动：./start.sh，查看输出日志有误异常，确保部署正常，ctrl+c 退出
### 7.后台启动：nohup ./start.sh &


## 三、部署可能存在问题
### 1. start.sh启动不了，将启动脚本编码改成utf-8
### 2. 

# 一、安装Maven
> 环境：
> 
> centos：8
> 
> jdk：1.8
> 
> 

1.下载文件
> 选择版本：https://archive.apache.org/dist/maven/maven-3

2.配置环境变量
```shell
# 修改配置
vim /etc/profile

# 添加内容
export MAVEN_HOME=/opt/maven-3.6.3
export PATH=$MAVEN_HOME/bin:$PATH

#刷新环境变量
source /etc/profile

# 查看版本
mvn -v
```

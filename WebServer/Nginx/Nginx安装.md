[TOC]

# 一、安装
## 1.CentsOS8安装
安装依赖：`yum -y install gcc-c++ pcre pcre-devel zlib zlib-devel openssl openssl-devel`

下载安装包：`wget -c https://nginx.org/download/nginx-1.25.3.tar.gz`

解压：`tar -zxvf nginx-1.25.3.tar.gz`

安装编译：
`cd nginx-1.25.3`
`./configure --prefix=/opt/nginx-1.25.3 --with-http_stub_status_module --with-http_ssl_module --with-http_v2_module --with-http_sub_module --with-http_gzip_static_module --with-pcre`

编译安装：
`make`
`make install`
分开执行

执行：到安装路径(/opt/nginx-1.25.3/sbin)执行./nginx

开放端口：
`firewall-cmd --zone=public --add-port=80/tcp --permanent && firewall-cmd --reload`

```shell
# 操作：
./nginx    #  启动
./sbin/nginx -c ./conf/nginx.conf    #  启动
./nginx -s quit #（温和）此方式停止步骤是待nginx进程处理任务完毕进行停止。
./nginx -s stop #（强硬）此方式相当于先查出nginx进程id再使用kill命令强制杀掉进程。
./nginx -s reload # 重启nginx(不推荐此方法，推荐先停止在启动)
```

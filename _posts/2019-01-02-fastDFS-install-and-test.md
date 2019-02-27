---
layout: post
title: 'FastDFS分布式文件系统安装及使用指南'
tags: [code]
---

### 安装

#### 参考文章

- [分布式文件系统-FastDFS](http://blog.51cto.com/xinzong/1834466)
- [使用FastDFS搭建图片服务器单实例篇](http://blog.51cto.com/oldcat1981/1766810)
- [CentOS7使用firewalld打开关闭防火墙与端口](https://www.cnblogs.com/moxiaoan/p/5683743.html)

#### 服务器环境

- CentOS7
- IP: 120.78.237.137

#### 安装基本环境

```bash
yum groupinstall "Development Tools" "Server platform Development"
```

#### 安装 libfastcommon

```bash
cd /usr/local/
git clone https://github.com/happyfish100/libfastcommon.git
cd libfastcommon/
./make.sh
./make.sh install
```

#### 安装 fastdfs

```bash
cd /usr/local/
git clone https://github.com/happyfish100/fastdfs.git
cd fastdfs/
./make.sh
./make.sh install
```

#### 配置 tracker

```bash
cd /etc/fdfs
cp tracker.conf.sample tracker.conf
vim /etc/fdfs/tracker.conf

disabled=false（默认为false，表示是否无效）
port=22122（默认为22122）
base_path=/data/fdfs/tracker
```

#### 配置 client.conf

```bash
cd /etc/fdfs
cp client.conf.sample client.conf
vim /etc/fdfs/client.conf

base_path=/data/fdfs/tracker
tracker_server=120.78.237.137:22122
```

#### 创建 tracker 目录

```bash
mkdir -pv /data/fdfs/tracker
```

#### 启动 tracker

- *centos7 启动方式*

```bash
/etc/init.d/fdfs_trackerd start
```

#### 查看端口

```bash
ss -lntup|grep 22122
tcp LISTEN 0 128 :22122 :* users:((“fdfs_trackerd”,3785,5)) 
```

#### 关闭tracker

```bash
/etc/init.d/fdfs_trackerd stop
```

**注意：虽然FastDFS区分tracker和storage服务器，但是安装的软件及步骤均相同，只是不同的配置文件而已，因此以上安装适用tracker server和storage server**

#### 配置 storage

```bash
cd /etc/fdfs
cp storage.conf.sample storage.conf
vim /etc/fdfs/storage.conf

disabled=false（默认为false，表示是否无效）
port=23000（默认为23000）
group_name=group1 #指定组名
base_path=/data/fdfs/storage # 用于存储数据
store_path_count=2 # 设置设备数量
store_path0=/data/fdfs/storage/m0 #指定存储路径0
store_path1=/data/fdfs/storage/m1 #指定存储路径1
注意：同一组内存储路径不能冲突，例如：下一个节点的存储路径就是m2,m3….等
tracker_server=120.78.237.137:22122 #指定tracker
http.server_port=8888（默认为8888，nginx中配置的监听端口那之一致）

mkdir -pv /data/fdfs/storage/{m0,m1} # 创建数据目录
```

#### 启动 storage

**必须先启动tracker，再启动storage**

- centos7 启动方式

```bash
/etc/init.d/fdfs_storaged start
```

#### 查看端口

```bash
ss -lntup|grep 23000
LISTEN 0 128 :23000 :* 
```

#### 关闭storage

```bash
/etc/init.d/fdfs_storaged stop
```

#### 文件上传测试

**springboot 测试方案**

- 引入依赖

```xml
<dependency>
    <groupId>com.github.tobato</groupId>
    <artifactId>fastdfs-client</artifactId>
</dependency>
```

- 编写配置类

```java
@Configuration
@Import(FdfsClientConfig.class)
// 解决jmx重复注册bean的问题
@EnableMBeanExport(registration= RegistrationPolicy.IGNORE_EXISTING)
public class FastClientImporter {
}
```

- 编写测试类

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = LyUploadService.class)
public class FdfsTest {
    @Autowired
    private FastFileStorageClient storageClient;
    @Autowired
    private ThumbImageConfig thumbImageConfig;
    @Test
    public void testUpload() throws FileNotFoundException {
        File file=new File("D:\\my_project\\idea_pro_inuse\\leyouUpload\\logo3.jpg");
        //上传并且生成缩略图
        StorePath storePath= this.storageClient.uploadFile(
                new FileInputStream(file),file.length(),"jpg",null);
        // 带分组的路径
        System.out.println(storePath.getFullPath());
        // 不带分组的路径
        System.out.println(storePath.getPath());

    }

    @Test
    public void testUploadAndCreateThumb() throws FileNotFoundException {
        File file=new File("D:\\my_project\\idea_pro_inuse\\leyouUpload\\logo.jpg");
        // 上传并且生成缩略图
        StorePath storePath=this.storageClient.uploadImageAndCrtThumbImage(
                new FileInputStream(file),file.length(),"jpg",null);
        // 带分组的路径
        System.out.println(storePath.getFullPath());
        // 不带分组的路径
        System.out.println(storePath.getPath());
        // 获取缩略图路径
        String path=thumbImageConfig.getThumbImagePath(storePath.getPath());
        System.out.println(path);
    }
}
```

#### 测试效果

>group1/M01/00/00/rBKowFx2JGWADBRMAACARrBGE7w855.jpg
>M01/00/00/rBKowFx2JGWADBRMAACARrBGE7w855.jpg

#### 注意事项

- 改动以上配置文件当中的server服务器地址

- 开放21222和23000两个端口

#### 存储服务器（storage server）安装并配置nginx

- 安装 fastdfs-nginx-module 模块

```bash
cd /root
git clone https://github.com/happyfish100/fastdfs-nginx-module
cp /root/fastdfs-nginx-module/src/mod_fastdfs.conf /etc/fdfs/
vim /etc/fdfs/mod_fastdfs.conf

connect_timeout=10
base_path=/tmp（默认为/tmp）
tracker_server=120.78.237.137:22122
storage_server_port=23000（默认配置为23000）
url_have_group_name = true
store_path_count=2 # 设置设备数量
store_path0=/data/fdfs/storage/m0
store_path1=/data/fdfs/storage/m1
group_name=group1（默认配置为group1）
```

#### 安装 nginx 依赖库

```bash
yum -y install pcre-devel zlib-devel
yum -y install openssl openssl-devel
```

#### 安装 nginx

```bash
 cd /root 
 wget http://nginx.org/download/nginx-1.8.1.tar.gz 
 tar xf nginx-1.8.1.tar.gz 
 cd nginx-1.8.1 
 ./configure --prefix=/application/nginx/ --add-module=../fastdfs-nginx-module/src/ 
 make && make install
```

```bash
cp /usr/local/fastdfs/conf/http.conf /etc/fdfs/
cp /usr/local/fastdfs/conf/mime.types /etc/fdfs/
```

#### 配置 nginx

```bash
vim /application/nginx/conf/nginx.conf
```

```nginx
user root; 
worker_processes 1; events {
    worker_connections  1024;
} 
http {
        include       mime.types;
        default_type  application/octet-stream;
        sendfile        on;
        keepalive_timeout  65;
        server {
            listen       8888;
            server_name  localhost;
            location ~/group[0-9]/ {
                ngx_fastdfs_module;
            } error_page 500 502 503 504 /50x.html; location = /50x.html {
            root   html;
            } 
        } 
    }
```

#### 启动 nginx

```bash
cp /application/nginx/sbin/nginx /etc/init.d/
/etc/init.d/nginx

ss -lntup|grep 8888
tcp LISTEN 0 128 :8888 :* users:((“nginx”,7308,6),(“nginx”,7309,6))

```

#### 开启端口

开启8888端口

#### 远程访问

> http://120.78.237.137:8888/group1/M01/00/00/rBKowFx2JGWADBRMAACARrBGE7w855.jpg

![](../images/fastdfsTest.png)
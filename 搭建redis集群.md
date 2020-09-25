#### 架构介绍

伪集群：在一台机器上安装所有服务器

| 主库               | 从库               |
| ------------------ | ------------------ |
| 192.168.6.140:7001 | 192.168.6.140:7002 |
| 192.168.6.140:7003 | 192.168.6.140:7004 |
| 192.168.6.140:7005 | 192.168.6.140:7006 |
| 192.168.6.140:7006 | 192.168.6.140:7008 |

#### 搭建步骤（注意改为自己的安装目录）

- 下载压缩包

  ```
  wget http://download.redis.io/releases/redis-5.0.9.tar.gz
  ```

- 解压

  ```
  tar -zxvf redis-5.0.9.tar.gz
  ```

- 安装

  ```
  make install PREFIX=/root/software/redis_cluster/7001
  ```

- 拷贝redis.conf

  ```
  cp /root/software/redis-5.0.9/redis.conf /root/software/redis_cluster/7001/bin
  ```

- 修改redis.conf

  - 注释bind 127.0.0.1
  - protected-mode no
  - port 7001
  - daemonize yes
  - 开启cluster-enabled yes

- 拷贝7001，修改各自的端口号redis.conf

  ```
  cp -r 7001 7002
  cp -r 7001 7003
  cp -r 7001 7004
  cp -r 7001 7005
  cp -r 7001 7006
  ```

- 编写脚本start.sh，批量启动

  ```
  cd 7001/bin
  ./redis-server redis.conf
  cd ..
  cd ..
  
  cd 7002/bin
  ./redis-server redis.conf
  cd ..
  cd ..
  
  cd 7003/bin
  ./redis-server redis.conf
  cd ..
  cd ..
  
  cd 7004/bin
  ./redis-server redis.conf
  cd ..
  cd ..
  
  cd 7001/bin
  ./redis-server redis.conf
  cd ..
  cd ..
  
  cd 7005/bin
  ./redis-server redis.conf
  cd ..
  cd ..
  
  cd 7006/bin
  ./redis-server redis.conf
  cd ..
  cd ..
  ```

修改脚本权限：chmod u+x start.sh

创建集群

```
./redis-cli --cluster create 192.168.6.140:7001 192.168.6.140:7002 192.168.6.140:7003 192.168.6.140:7004 192.168.6.140:7005 192.168.6.140:7006 --cluster-replicas 1
```

连接

```
./redis-cli -h 127.0.0.1 -p 7001 -c
```

#### 扩容

- 创建主节点7007（无数据）

  ```
  mkdir 7007
  make install PREFIX=/root/software/redis_cluster/7007
  ```

- 向集群添加7007节点，并启动

  ```
  ./redis-cli --cluster add-node 192.168.6.140:7007 192.168.6.140:7001
  ```

- 拷贝redis.conf；配置redis.conf

- 重新分槽

  ```
  ./redis-cli --cluster reshard 192.168.6.140:7007
  ```

- 将7008置为7007的从节点

  ```
  ./redis-cli --cluster add-node 192.168.6.140:7008 192.168.6.140:7007 --cluster-slave --cluster-master-id 09254ff517b5d81b33c5fd1519279e901b1e1cfd
  ```

#### 安装错误

1. CC命令未找到

   ![image-20200925015856382](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200925015856382.png)

   解决方案：安装gcc

   ```
   yum install gcc
   ```

   

#### 程序遇见的bug

1. 连接超时

   ![image-20200925085024279](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200925085024279.png)

   具体原因具体分析，我的是因为端口号问题
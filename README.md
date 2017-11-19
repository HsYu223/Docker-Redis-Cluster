# encoding: UTF-8
# Docker-Redis-Cluster
測試架構,預計在CentOS7 Docker環境下建立6台測試機進行測試

![測試架構](https://github.com/HsYu223/Docker-Redis-Cluster/blob/master/Images%20File/redis%20server.jpg)

下載docker images (測試使用版本為3.2.9)
```bash
sudo docker pull redis:3.2.9
```

建立redis設定檔([範例](https://github.com/HsYu223/Docker-Redis-Cluster/blob/master/redis_sample.conf))
```bash
sudo mkdir -p /myredis/conf
sudo touch /myredis/conf/redis7001.conf
sudo touch /myredis/conf/redis7002.conf
sudo touch /myredis/conf/redis7003.conf
sudo touch /myredis/conf/redis7004.conf
sudo touch /myredis/conf/redis7005.conf
sudo touch /myredis/conf/redis7006.conf
sudo touch /myredis/conf/redis7007.conf
sudo touch /myredis/conf/redis7008.conf
```

安裝docker redis container
```bash
sudo docker run -v /myredis/conf/redis7001.conf:/usr/local/etc/redis/redis.conf --net=host -p 7001:7001 -p 17001:17001 --name redis01 -d  redis:3.2.9 redis-server /usr/local/etc/redis/redis.conf
sudo docker run -v /myredis/conf/redis7002.conf:/usr/local/etc/redis/redis.conf --net=host -p 7002:7002 -p 17002:17002 --name redis02 -d  redis:3.2.9 redis-server /usr/local/etc/redis/redis.conf
sudo docker run -v /myredis/conf/redis7003.conf:/usr/local/etc/redis/redis.conf --net=host -p 7003:7003 -p 17003:17003 --name redis03 -d  redis:3.2.9 redis-server /usr/local/etc/redis/redis.conf
sudo docker run -v /myredis/conf/redis7004.conf:/usr/local/etc/redis/redis.conf --net=host -p 7004:7004 -p 17004:17004 --name redis04 -d  redis:3.2.9 redis-server /usr/local/etc/redis/redis.conf
sudo docker run -v /myredis/conf/redis7005.conf:/usr/local/etc/redis/redis.conf --net=host -p 7005:7005 -p 17005:17005 --name redis05 -d  redis:3.2.9 redis-server /usr/local/etc/redis/redis.conf
sudo docker run -v /myredis/conf/redis7006.conf:/usr/local/etc/redis/redis.conf --net=host -p 7006:7006 -p 17006:17006 --name redis06 -d  redis:3.2.9 redis-server /usr/local/etc/redis/redis.conf
sudo docker run -v /myredis/conf/redis7007.conf:/usr/local/etc/redis/redis.conf --net=host -p 7007:7007 -p 17007:17007 --name redis07 -d  redis:3.2.9 redis-server /usr/local/etc/redis/redis.conf
sudo docker run -v /myredis/conf/redis7008.conf:/usr/local/etc/redis/redis.conf --net=host -p 7008:7008 -p 17008:17008 --name redis08 -d  redis:3.2.9 redis-server /usr/local/etc/redis/redis.conf
```
先把6台服務一起啟動
```bash
docker start redis01 redis02 redis03 redis05 redis06 redis07
```
確認啟動狀態
```bash
docker ps -a
```

啟動redis叢集 (redis-trib)
```<language>
redis-trib.rb create --replicas 1 10.101.1.110:7001 10.101.1.110:7002 10.101.1.110:7003 10.101.1.110:7005 10.101.1.110:7006 10.101.1.110:7007
```

確認叢集狀態
```bash
redis-trib.rb check 10.101.1.110:7001
```

叢集測試案例：
1. 加入master

```bash
add-node 10.101.1.110:7004 10.101.1.110:7001
```
![Add Master Node](https://github.com/HsYu223/Docker-Redis-Cluster/blob/master/Images%20File/add%20node%207007.jpg)
   
2. 加入slave

```<language>
add-node --slave --master-id b0f03581f787c991ef0ee641d06be9bb1aa6606f 10.101.1.110:7008 10.101.1.110:7004
```
![Add Slave Node](https://github.com/HsYu223/Docker-Redis-Cluster/blob/master/Images%20File/add%20slave%20node%207008.jpg)

3. 為新加入節點分配slot

```bash
reshard 10.101.1.110:7001 (之後會詢問要移動多少slots、接收節點是誰、要從哪些節點移動slot)
```
![Check Node](https://github.com/HsYu223/Docker-Redis-Cluster/blob/master/Images%20File/move%20slot.jpg)

![Move Result](https://github.com/HsYu223/Docker-Redis-Cluster/blob/master/Images%20File/move%20slot%20result.jpg)

4. master failover確認是否成為slave   

![Master Failover](https://github.com/HsYu223/Docker-Redis-Cluster/blob/master/Images%20File/redis04%20failover.jpg)

![Master Failover](https://github.com/HsYu223/Docker-Redis-Cluster/blob/master/Images%20File/redis08%20will%20be%20master.jpg)

![Master Failover](https://github.com/HsYu223/Docker-Redis-Cluster/blob/master/Images%20File/when%20redi04%20get%20back%20will%20be%20slave.jpg)

5. 延續3. 還原成master確認

![Master Failover](https://github.com/HsYu223/Docker-Redis-Cluster/blob/master/Images%20File/redis04%20reset%20to%20master.jpg)

![Master Failover](https://github.com/HsYu223/Docker-Redis-Cluster/blob/master/Images%20File/redis08%20restart%20get%20back%20will%20be%20slave.jpg)

6. slave failover

![Slave Failover](https://github.com/HsYu223/Docker-Redis-Cluster/blob/master/Images%20File/redis08%20slave%20fail%20over.jpg)

![Slave Failover](https://github.com/HsYu223/Docker-Redis-Cluster/blob/master/Images%20File/redis08%20slave%20get%20back.jpg)

7. 刪除master、slave

```bash
/// Master (移除master之前,請記得先清空已經分配到的slot,嘗試使用案例3的方式重新分配slot)
del-node 10.101.1.110:7004 8bc21197b94ed9ba3703d3643d5ed8a3399463ed

/// Slave (不需要重新分配slot)
del-node 10.101.1.110:7008 dc05c7005e664b24709475bb17114a2ecf6bca71
```

![Remove Master](https://github.com/HsYu223/Docker-Redis-Cluster/blob/master/Images%20File/remove%20redis04%20from%20cluster.jpg)

![Remove Slave](https://github.com/HsYu223/Docker-Redis-Cluster/blob/master/Images%20File/remove%20redis08%20slave%20node.jpg)

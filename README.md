# Docker-Redis-Cluster
���լ[�c,�w�p�bCentOS7 Docker���ҤU�إ�6�x���վ��i�����

![���լ[�c](https://github.com/HsYu223/Docker-Redis-Cluster/blob/master/Images%20File/redis%20server.jpg)

�U��docker images (���ըϥΪ�����3.2.9)
```bash
sudo docker pull redis:3.2.9
```

�إ�redis�]�w��([�d��](https://github.com/HsYu223/Docker-Redis-Cluster/blob/master/redis_sample.conf))
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

�w��docker redis container
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
����6�x�A�Ȥ@�_�Ұ�
```bash
docker start redis01 redis02 redis03 redis05 redis06 redis07
```
�T�{�Ұʪ��A
```bash
docker ps -a
```

�Ұ�redis�O�� (redis-trib)
```<language>
redis-trib.rb create --replicas 1 10.101.1.110:7001 10.101.1.110:7002 10.101.1.110:7003 10.101.1.110:7005 10.101.1.110:7006 10.101.1.110:7007
```

�T�{�O�����A
```bash
redis-trib.rb check 10.101.1.110:7001
```

�O�����ծרҡG
1. �[�Jmaster

```bash
add-node 10.101.1.110:7004 10.101.1.110:7001
```
![Add Master Node](https://github.com/HsYu223/Docker-Redis-Cluster/blob/master/Images%20File/add%20node%207007.jpg)
   
2. �[�Jslave

```<language>
add-node --slave --master-id b0f03581f787c991ef0ee641d06be9bb1aa6606f 10.101.1.110:7008 10.101.1.110:7004
```
![Add Slave Node](https://github.com/HsYu223/Docker-Redis-Cluster/blob/master/Images%20File/add%20slave%20node%207008.jpg)

3. ���s�[�J�`�I���tslot

```bash
reshard 10.101.1.110:7001 (����|�߰ݭn���ʦh��slots�B�����`�I�O�֡B�n�q���Ǹ`�I����slot)
```
![Check Node](https://github.com/HsYu223/Docker-Redis-Cluster/blob/master/Images%20File/move%20slot.jpg)

![Move Result](https://github.com/HsYu223/Docker-Redis-Cluster/blob/master/Images%20File/move%20slot%20result.jpg)

4. master failover�T�{�O�_����slave   

![Master Failover](https://github.com/HsYu223/Docker-Redis-Cluster/blob/master/Images%20File/redis04%20failover.jpg)

![Master Failover](https://github.com/HsYu223/Docker-Redis-Cluster/blob/master/Images%20File/redis08%20will%20be%20master.jpg)

![Master Failover](https://github.com/HsYu223/Docker-Redis-Cluster/blob/master/Images%20File/when%20redi04%20get%20back%20will%20be%20slave.jpg)

5. ����3. �٭즨master�T�{

![Master Failover](https://github.com/HsYu223/Docker-Redis-Cluster/blob/master/Images%20File/redis04%20reset%20to%20master.jpg)

![Master Failover](https://github.com/HsYu223/Docker-Redis-Cluster/blob/master/Images%20File/redis08%20restart%20get%20back%20will%20be%20slave.jpg)

6. slave failover

![Slave Failover](https://github.com/HsYu223/Docker-Redis-Cluster/blob/master/Images%20File/redis08%20slave%20fail%20over.jpg)

![Slave Failover](https://github.com/HsYu223/Docker-Redis-Cluster/blob/master/Images%20File/redis08%20slave%20get%20back.jpg)

7. �R��master�Bslave

```bash
/// Master (����master���e,�аO�o���M�Ťw�g���t�쪺slot,���ըϥήר�3���覡���s���tslot)
del-node 10.101.1.110:7004 8bc21197b94ed9ba3703d3643d5ed8a3399463ed

/// Slave (���ݭn���s���tslot)
del-node 10.101.1.110:7008 dc05c7005e664b24709475bb17114a2ecf6bca71
```

![Remove Master](https://github.com/HsYu223/Docker-Redis-Cluster/blob/master/Images%20File/remove%20redis04%20from%20cluster.jpg)

![Remove Slave](https://github.com/HsYu223/Docker-Redis-Cluster/blob/master/Images%20File/remove%20redis08%20slave%20node.jpg)
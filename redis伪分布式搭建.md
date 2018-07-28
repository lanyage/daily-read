#Redis

## redis是什么?
答:redis是一个基于内存的key、value高性能的数据库。支持多种数据结构,string,hash,list,set,zset.可以用来实现缓存,简单的消息队列,排行榜,计数器等需求。

## redis支持几种部署模式？
答：redis支持3种不同的部署模式。 <br>
	
* 单机部署
* 主从复制
* 集群模式

## redis集群模式如何搭建？
答：集群模式的搭建如下:

1. 在官方下载tar.gz的安装包,解压。
2. 在指定目录下创建redis-cluster目录。将前面解压了的文件夹拷贝6份到redis-cluster目录，修改这些redis的redis.conf文件
	
		port 7001
		cluster-enable yes
3. 将src/路径下的redis.trib.rb的ruby文件复制到redis-cluster下.
4. 执行ruby脚本之前,需要安装ruby环境。
	
		$ yum install ruby
		$ gem install redis-3.0.0.gem
		
5. 启动所有的实例 redis-server ./redis.conf
6. 然后创建集群。

	```
	$ ./redis-trib.rb create --replicas 1 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 127.0.0.1:7006
	```

  该操作会创建3个master,3个slave以备master挂掉。
  
  成功之后会出现。
  
 ```
 [WARNING] Some slaves are in the same host as their master
 ```

		M: 8657c9c991cab5a60e1a5b18763ead6083b15210 127.0.0.1:7001
		   slots:0-5460 (5461 slots) master
		M: d9ae1952f4d9ac472a12ee4fe0f1a0024b13248d 127.0.0.1:7002
		   slots:5461-10922 (5462 slots) master
		M: 2eeb840576bfa4544dfdded2070dff7355a8c157 127.0.0.1:7003
		   slots:10923-16383 (5461 slots) master
		S: 9ddc48c675538e80187e5ea31838de1d78026664 127.0.0.1:7004
		   replicates d9ae1952f4d9ac472a12ee4fe0f1a0024b13248d
		S: 9f3d3914b70f7bd85f812630a68ebf1e9d29a63f 127.0.0.1:7005
		   replicates 2eeb840576bfa4544dfdded2070dff7355a8c157
		S: ab86c4b2193337b41a1ff29f8610d2b9803a7849 127.0.0.1:7006
		   replicates 8657c9c991cab5a60e1a5b18763ead6083b15210
		Can I set the above configuration? (type 'yes' to accept): yes
		>>> Nodes configuration updated
		>>> Assign a different config epoch to each node
		>>> Sending CLUSTER MEET messages to join the cluster
		Waiting for the cluster to join........
		>>> Performing Cluster Check (using node 127.0.0.1:7001)
		M: 8657c9c991cab5a60e1a5b18763ead6083b15210 127.0.0.1:7001
		   slots:0-5460 (5461 slots) master
		   1 additional replica(s)
		S: 9f3d3914b70f7bd85f812630a68ebf1e9d29a63f 127.0.0.1:7005
		   slots: (0 slots) slave
		   replicates 2eeb840576bfa4544dfdded2070dff7355a8c157
		S: 9ddc48c675538e80187e5ea31838de1d78026664 127.0.0.1:7004
		   slots: (0 slots) slave
		   replicates d9ae1952f4d9ac472a12ee4fe0f1a0024b13248d
		S: ab86c4b2193337b41a1ff29f8610d2b9803a7849 127.0.0.1:7006
		   slots: (0 slots) slave
		   replicates 8657c9c991cab5a60e1a5b18763ead6083b15210
		M: 2eeb840576bfa4544dfdded2070dff7355a8c157 127.0.0.1:7003
		   slots:10923-16383 (5461 slots) master
		   1 additional replica(s)
		M: d9ae1952f4d9ac472a12ee4fe0f1a0024b13248d 127.0.0.1:7002
		   slots:5461-10922 (5462 slots) master
		   1 additional replica(s)
		[OK] All nodes agree about slots configuratio	n.
  说明集群已经搭好。
  
7.使用jedis开始编写程序。

	public class JedisDemo {
	    public static void main(String[] args) {
	        Set<HostAndPort> hostAndPorts = new HashSet<>();
	        hostAndPorts.add(new HostAndPort("127.0.0.1", 7001));
	        hostAndPorts.add(new HostAndPort("127.0.0.1", 7002));
	        hostAndPorts.add(new HostAndPort("127.0.0.1", 7003));
	        hostAndPorts.add(new HostAndPort("127.0.0.1", 7004));
	        hostAndPorts.add(new HostAndPort("127.0.0.1", 7005));
	        hostAndPorts.add(new HostAndPort("127.0.0.1", 7006));
	        JedisCluster jedisCluster = new JedisCluster(hostAndPorts);
	        jedisCluster.set("name", "shufang");        //不同的键会根据哈希算法存到不同的实例上
	        jedisCluster.set("age", "18");
	        String name = jedisCluster.get("name");
	        String age = jedisCluster.get("age");
	        System.out.println(name);
	        System.out.println(age);
	    }
	}		













	
# 初识java nio

## 核心类
Channel, Buffer, Selector

可以在单线程的情况下，处理成百上千个socket连接，底部采用非阻塞的io。

channel一般和selector一起使用，将自己注册到selector中，监听事件。

伪代码如下:

	while(true){
		1.检查selector中有无事件
		2.if(channel可读)...
		  if(channel可写)...
	}


## 基于nio的netty是一个container，威胁着servlet。

因为servlet只支持http请求，但是netty支持http,ftp,udp和一些用户自定义的协议，实现了高性能高可靠的网络服务器，而dubbo是基于netty的，因此netty的细节被隐藏起来了。

		 
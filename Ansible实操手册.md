# Ansible实操手册
## 安装
Ansible默认通过SSH管理机器,你只需要在一台机器上面安装Ansible就行了,这台安装Ansible的机器叫Controller。

#####	首先安装虚拟机，并且配置虚拟网络。
  
```
$ vi /etc/sysconfig/network-scripts/ifcfg-en33 -> ONBOOT=yes
```	
	
```
$ systemctl restart network OR service network restart
```
	
```
$ hostnamectl set-hostname ansiblectl
```

```
$ exit
```
		
	$ nmtui
	-------------------------------
	Edit a connection -> Edit
	IPv4 CONFIGURATION <MANUAL>
	Addresses: 192.168.8.131/24
	Gateway: 192.168.8.2(最后一位一般为2) 
	DNS servers: 8.8.8.8, 4.4.8.8
	------------------------------

```		
ip addr(查看网络)
```

#####	配置SSH,实现免密码登录  
	
```	
$ ssh-keygen
```
	
```
$ ssh-copy-id -i root@192.168.8.132
```
   
```
$ ssh root@192.168.8.132
```

##### 在`centos7`上安装Ansible必须保证Python版本在`2.5版本`以上。

```
$ ansible hostname --sudo -m raw -a "yum install -y python2"
```

```
$ python --version //查看python版本,cento7上必须是2.x.x版本的
```
	
##### 安装Ansible
```
$ yum install -y ansible
```

```
$ ansible --version //查看ansible版本
```

##### 配置Ansible host,默认的主机配置文件为`/etc/ansible/hosts`
```
$ vim /etc/ansible/hosts
```

```
[db]
192.168.8.132
[webserver]
192.168.8.133
```

```
$ ansible all -m ping //查看是不是机器都能连通
```

## 配置
Ansible支持多种配置变量的方式,有环境变量、命令行切换和初始化的ansible.cfg文件。

配置文件：

*	ANSIBLE_CONFIG(environment variable if set)
* 	ansible.cfg(if the current directory)
*  ~/.ansible.cfg(in the home directory)
*  /etc/ansible/ansible.cfg

Ansible会依次处理上面目录的文件,并且使用第一个找到的文件,其他的将被忽略。配置变量的详情请查看[Ansible Configuration Settings](https://docs.ansible.com/ansible/latest/reference_appendices/config.html)  
除了可以通过Environment来配置变量之外,还可以通过<mark>***Command line***</mark>来提供配置参数,在Command line中的配置将会覆盖配置文件和environment的配置。

##Get started

当与远程机器通信的时候,Ansible默认假设你使用的SSH密匙。SSH是Ansible官方推荐的方式,可以通过`--ask-pass`来使用密码.  

编辑`/etc/ansible/hosts`文件来配置管理的远端机器,这个文件叫做Inventory。

```
[db]
192.168.8.132
[webserver]
192.168.8.133
```
	
如果需要实现免SSH密码登录
	
	$ ssh-agent bash
	$ ssh-add ~/.ssh/id_rsa

现在开始ping你所有的远端机器

```
$ ansible all -m ping
```
会返回json结果。其他的ping方式如下。

	# 强ping
	$ ansible all -m ping -u lanyage
	# 强ping,用sudo
	$ ansible all -m ping -u lanyage --sudo
	# 强ping,用lanyage身份
	$ ansible all -m ping -u lanyage --sudo --sudo-user lanyage
	
	# With latest version of ansible `sudo` is deprecated so use become
	# as bruce, sudoing to root
	$ ansible all -m ping -u bruce -b
	# as bruce, sudoing to batman
	$ ansible all -m ping -u bruce -b --become-user batman

在所有机器上完成一条指令

```
# 使用原生的shell语句
$ ansible all -m raw -a "echo Hello"		
```
现在你就已经完成和节点之间的通信,你还可以在命令后面添加简单的额外参数。

```
$ ansible localhost -m ping -e 'ansible_python_interpreter="/usr/bin/env python"'
```

## Ad-Hoc Command(自组织命令)
1. 什么是自组织命令  
一个自组织命令往往是你希望快速的执行，并且不需要为下面的操作进行保存的命令。
2. 总的来说Ansible的真正力量是playbook


#### <a name="fenced-code-block">并行和shell命令</a>
使用命令关闭同组的所有机器。

	$ ssh-agent bash  
	$ ssh-add !/.ssh/id_rsa. //如果没有该目录的话就自己创建
并行的关闭电脑，每次10台，-f 10是同时操作10台机器,默认是5,你可以修改ansible的配置文件来避免以后需要再次设置.

```
$ ansible test -a "reboot" -f 10
```

ansible默认使用你当前的用户来执行，如果你不喜欢，你可以使用如下命令来指定远程机器上执行命令的用户。

```
$ ansible test -a "reboot" -u username
```
通常如果远程机器上面用户的权限不够的情况下,你需要使用--become来执行命令,如果使用-K说明要输入当前用户的密码，当然，这个密码不是强制输入的。

```
$ ansible test -i hosts -m command -a "echo hello" -u lanyage --become -k
```

当然也可以变成其他的用户去执行命令,默认的是become为root，你也可以变成其他的用户,比如说lanyage。

```
$ ansible test -i hosts -m command -a "echo hello" -u lanyage --become --become -user shufang -K
```

有时候，你可能需要选择模块去运行,比如说使用-m 来指定module name, 默认的module name是command 。比如说使用shell模块。

```
$ ansible test -m shell -a "echo hello"
```
#### <a name="fenced-code-block">文件传输</a>
ansible支持同时给多台机器传送文件,使用command line,如果说想要文件传送到远端的机器只需要使用:

```
$ transfer test -m copy -a "src=./test.txt dest=/home/lanyage/dump/test.txt"
```

如果你是用的是playbook,你可以使用模版。

file模块允许修改ownership和权限,如:

	$ ansible webservers -m file -a "dest=/srv/foo/a.txt mode=600"
	$ ansible webservers -m file -a "dest=/srv/foo/b.txt mode=600 owner=lanyage group=god"
	
file也可以创建文件夹,如:
	
	$ ansible webservers -m file -a "dest=/path/to/c mode=755 owner=mdehaan group=mdehaan state=directory"
	 
file还可以地柜的删除文件夹:
	
	$ ansible webservers -m file -a "dest=/path/to/c state=absent"

#### <a name="fenced-code-block">包管理</a>		
ansible也支持包管理模块,如yum和apt。  

如下载一个包,但是不更新.

```
$ ansible webservers -m yum -a "name=acme state=present"
```

下载一个包的版本

```
$ ansible webservers -m yum -a "name=acme-1.5 state=present"
```

写在一个包

```
$ ansible webservers -m yum -a "name=acme state=absent"
```
#### <a name="fenced-code-block">用户和组</a>		

	$ ansible all -m user -a "name=foo password=<crypted password here>"

	$ ansible all -m user -a "name=foo state=absent"	
	
#### <a name="fenced-code-block">直接从远端部署项目</a>	

```
$ ansible webservers -m git -a "repo=https://foo.example.org/repo.git dest=/srv/myapp version=HEAD"

```

#### <a name="fenced-code-block">管理服务</a>	

开启一个服务

```
$ ansible webservers -m service -a "name=httpd state=started"

```

重启一个服务

```
$ ansible webservers -m service -a "name=httpd state=restarted"
```

终止一个服务

```
$ ansible webservers -m service -a "name=httpd state=stopped"
```
#### <a name="fenced-code-block">时间限制的后台操作</a>
长时间的操作可以放在后台,然后后面在check状态,例如,long_running_operation 在后台异步操作,耗时3600秒,-P 0表示不轮询

```
$ ansible all -B 3600 -P 0 -a "/usr/bin/long_running_operation --do-stuff"
```

如果你想稍后再查看这个job的状态,你可以使用async_status模块,将job id传过去,这个jid是在你运行工作的时候返回的.

```
$ ansible web1.example.com -m async_status -a "jid=488359678239.2844"
```

如果需要轮询:

```
$ ansible all -B 1800 -P 60 -a "/usr/bin/long_running_operation --do-stuff"
```
上面的意思最大运行1800秒,并且每60秒轮询一次。超过时间时候进程会被终止。这样的长时间命令一般是软件升级和耗时的shell脚本。

#### <a name="fenced-code-block">获取系统参数</a>

facts是在playbook部分进行描述,代表着被解析的系统变量。这些facts能够被用作传统的任务执行的实现，通过这些命令来查看所有的facts.

```
$ ansible all -m setup
```

## Inventory

ansible是通过inventory来管理所有的机器的.默认的inventory文件在/etc/ansible/hosts下面。也可以通过`-i <path>`来指定inventory地址。


ansible支持一个时间内使用多个inventory文件。


#### <a name="fenced-code-block">主机组</a>

主机组一般像这样表示:
	
	[webservers]
	192.168.8.131
	[dbservers]
	192.168.8.132
wenservers和dbservers是组名, 组名下面的是该组下的所有的机器。

一个YAML可能看起来像这样:
	
	all:
	  hosts:
	    mail.example.com:
	  children:
	    webservers:
	      hosts:
	        foo.example.com:
	        bar.example.com:
	    dbservers:
	      hosts:
	        one.example.com:
	        two.example.com:
	        three.example.com:

可以将系统放在多个组中,比如server可以在webserver和dbserver两个组中, 变量来自他们所属的所有组。

如果你有主机运行在不标准的ssh端口,比如说主机运行端口不为22而为5309端口。建议将他们后面设置端口号。

```
badwolf.example.com:5309
```
加入你有静态IP并且你想设置一些别名,你可以这样设置。

```
jumper ansible_port=5555 ansible_host=192.168.2.50
```

并且在YAML中这样引用
···
  hosts:
    jumper: 
      ansible_port: 5555
      ansible_host: 192.0.2.50

上面就是尝试去搞jumper这个别名,也就是`192.0.2.50:5555`.但是这不是使用变量的最好的方法,最好的方法是使用模板。
`hello: Helloworld >hello =  Helloworld`

如果你需要设置很多主机,你可以这样设置:

	[webservers]
	www[01:50].example.com

	[databases]
	db-[a:f].example.com	

你也可以设置连接的类型和ansible的操作用户:
	
	[targets]
	localhost              ansible_connection=local
	other1.example.com     ansible_connection=ssh        	ansible_user=mpdehaan
	other2.example.com     ansible_connection=ssh        	ansible_user=mdehaan

这样在inventory文件里设置这些东西往往只是为了快速记忆,下面将讨论如何将这些变量分别存入host_vars目录下单独的文件中。

#### <a name="fenced-code-block">主机变量</a>

我们可以非常快速的指定变量到主机,这些变量往往后面会被用到。

	[atlanta]
	host1 http_port=80 maxRequestsPerChild=808
	host2 http_port=303 maxRequestsPerChild=909

#### <a name="fenced-code-block">group 变量</a>

变量也可以被应用到整个group中去。

INI方式:

	[atlanta]
	host1
	host2
	
	[atlanta:vars]
	ntp_server=ntp.atlanta.example.com
	proxy=proxy.atlanta.example.com

YAML方式:

	atlanta:
	  hosts:
	    host1:
	    host2:
	  vars:
	    ntp_server: ntp.atlanta.example.com
	    proxy: proxy.atlanta.example.com

这里你要意识到这只是一种方便的方式,即使你可以使用group variable,但是变量一般都是在play执行之前被扁平化到主机级别。

#### <a name="fenced-code-block">group中的group和group变量</a>

ansible允许你在组中定义子组,同样也支持两种形式的表现。
INI方式:
	
	[atlanta]
	host1
	host2
	
	[raleigh]
	host2
	host3
	
	[southeast:children]
	atlanta
	raleigh
	
	[southeast:vars]
	some_server=foo.southeast.example.com
	halon_system_timeout=30
	self_destruct_countdown=60
	escape_pods=2
	
	[usa:children]
	southeast
	northeast
	southwest
	northwest
	
YAML方式:
	
	all:
	  children:
	    usa:
	      children:
	        southeast:
	          children:
	            atlanta:
	              hosts:
	                host1:
	                host2:
	            raleigh:
	              hosts:
	                host2:
	                host3:
	          vars:
	            some_server: foo.southeast.example.com
	            halon_system_timeout: 30
	            self_destruct_countdown: 60
	            escape_pods: 2
	        northeast:
	        northwest:
	        southwest:
	       
如果你需要存储集合或者哈希数据,或者你喜欢将hosts和group variables分开在不同的inventory中,子组有很多值得注意的属性:

1. 任何子组中的主机都是父组的一部分
2. 子组的属性会覆盖父亲的属性
3. 组可以拥有多个父亲和孩子,但是没有环形依赖
4. 主机也可以处于多个组,但是将永远只有一个实例,将所有组中的变量进行了融合

#### <a name="fenced-code-block">默认组</a>
ansible有两个默认组,all和ungrouped。

#### <a name="fenced-code-block">主机和数据分离</a>

ansible推荐将主机和变量分离。可以使用group_vars目录用来存放变量。

	group_vars/all 表示所有组共享
	group_vars/webserver 表示webserver共享
	host_vars/localhost 表示某一host独享的变量
	
例如,如果你有一群主机,如果dbservers想使用某中变量,那么group_vars/dbservers文件就应该看起来是这样的。

	---
	ntp_server: acme.example.org
	database_server: storage.example.org
	
类似于这种文件是可选的,是可以不需要存在的。还有就是你也可以这样配置,将组变量分的更加细致,如:
	
	/etc/ansible/group_vars/raleigh/db_settings
	/etc/ansible/group_vars/raleigh/cluster_settings
	
playbook目录和host目录都可以存在group_vars,前者会覆盖后者。

#### <a name="fenced-code-block">变量是如何进行覆盖的</a>

变量默认的被扁平化到特定的主机在play开始执行之前。这样使得ansible专注于主机和任务, 这样主机组在inventory和host匹配之外是不会存活的。组的优先级排序如下:
	
	1. all group
	2. parent group
	3. child group
	4. host

后面的变量会覆盖前面的变量。


在ansible 2.4以后,用户可以设置变量优先级`ansible_group_priority`来改变merge同级组的顺序。数字越大,merge的越晚,优先级越高,默认为1。

	a_group:
	    testvar: a
	    ansible_group_priority: 10
	b_group
	    testvar: 
	    
在这个例子中,如果两个组都有同样的优先级,最终的结果会是`testvar == b`, 但是我们给了a_group一个更高的优先级,所以结果将会是`testvar == a`。

## Dynamic Inventory

通常一个用户需要将配置放在不同的软件系统, 如频繁的从云端, LDAP, Cobbler或者一系列暗鬼的CMDB软件。Ansible能够非常简单的支持这些操作通过一个外部的inventory系统,这些目录包括EC2/Eucalyptus,Rackspace Cloud,OpenStack等等的操作。

Ansible Tower提供用于存储inventory 结果的数据库,支持Restful风格。Tower与所有的动态inventory资源同步，并且支持图像inventory编辑器。有了这样的数据库记录,你就能够非常将过去的事件历史关联起来,并且知道哪一个play在他们过去的执行中失败了。

##  Playbooks
playbook是Ansible最强大的功能了,能够非常方便的进行配置,部署。是ansible的演奏语言。playbook能够描述一种你将要强加在远端机器上的策略。并且以一系列的流程顺序而执行。

用一个形象的比喻就是playbook是指导手册,而inventory就是你的原材料。

#### <a name="fenced-code-block">playbook介绍</a>

playbook不同于ad-hoc命令,playbook非常的适合复杂应用的部署。playbook可以声明配置也可以以人为顺序去进行安装部署。可同步可异步。

#### <a name="fenced-code-block">playbook语言案例</a>

playbook的格式是YAML格式+许多plays,playbook就是将hosts映射成为定义好的角色.这个角色执行一个任务,这个任务就是使用ansible去执行module。

plays是类似的,你可以在不同的时间执行不同的play。初识playbook,我们从single play开始吧。
	
	- hosts: webservers
	  vars:
	    http_port: 80
	    max_clients: 200
	  remote_user: root
	  tasks:
	  - name: ensure apache is at the latest version
	    yum:
	      name: httpd
	      state: latest
	  - name: write the apache config file
	    template:
	      src: /srv/httpd.j2
	      dest: /etc/httpd.conf
	    notify:
	    - restart apache
	  - name: ensure apache is running
	    service:
	      name: httpd
	      state: started
	  handlers:
	    - name: restart apache
	      service:
	        name: httpd
	        state: restarted

playbooks能包含多个plays,你可能需要一个playbook for webserver还需要一个for dbservers,例如:

	---
	- hosts: webservers
	  remote_user: root
	  become: yes
	  become_method: sudo
	  tasks:
	  - name: ensure apache is at the latest version
	    yum:
	      name: httpd
	      state: latest
	  - name: write the apache config file
	    template:
	      src: /srv/httpd.j2
	      dest: /etc/httpd.conf
	
	- hosts: databases
	  remote_user: root
	
	  tasks:
	  - name: ensure postgresql is at the latest version
	    yum:
	      name: postgresql
	      state: latest
	  - name: ensure that postgresql is started
	    service:
	      name: postgresql
	      state: started
	 
你可以快速切换在不同的group之间,确定登陆的用户,用户是不是需要sudo权限等等。

接下来,咱们对这些东西分开进行讲解。

#### <a name="fenced-code-block">Hosts and users</a>

对于每一个play,你需要选择你要操作的机器是哪台, 登录机器的用户用哪个从而来执行tasks。

hosts是一个或多个主机组组成的,通过冒号隔开。remote_user就是用户的账户名。在ansible1.4之前是使用的user,现在用remote_user是为了和module user区分开来.

	---
	- hosts: webservers
	  remote_user: root

remote_user也可以为每个task定义都定义一个。

	---
	- hosts: webservers
	  remote_user: root
	  tasks:
	    - name: test connection
	      ping:
	      remote_user: yourname
如果说你想变成另外一个用户也是允许的。使用become: yes

	---
	- hosts: webservers
	  remote_user: yourname
	  become: yes
一样的,你也可以为每个task都使用一个become而不是整个play

	---
	- hosts: webservers
	  remote_user: yourname
	  tasks:
	    - service:
	        name: nginx
	        state: started
	      become: yes
	      become_method: sudo
你也可以用你的账户登录然后变成另外一个不是root的用户.
	
	---
	- hosts: webservers
	  remote_user: yourname
	  become: yes
	  become_user: postgres
你还可以使用其他的权限提升的方法,比如：
	
	---
	- hosts: webservers
	  remote_user: yourname
	  become: yes
	  become_method: su
如果同时你需要输入密码的时候, 使用ansible-playbook的时候在后面加上--ask-become-pass或者使用旧的语法 --ask-sudo-pass `(-K)`,当你是用这个的时候,playbook可能会被挂起。

在2.4版本之后你可以控制主机执行的顺序,默认的顺序是inventory中定义的顺序.

	- hosts: all
	  order: sorted
	  gather_facts: False
	  tasks:
	    - debug:
	        var: inventory_hostname

order可取的值为:`inventory`,`reverse_inventory`,`sorted`,`reverse_sorted`,`shuffle`，分别按照顺序,反顺序,字母顺序,反字母顺序,随机排序的方式进行执行。


#### <a name="fenced-code-block">Tasks列表</a>
每一个play都包含一系列列的任务。任务按照一定顺序执行,一次执行一个任务.task的目标就是去执行一个模块,包含一些指定的参数。变量能够被用在模块的参数中。模块的执行都是幂等的。也就是说执行一个模块多次和他运行一次的效果是一样的。一种实现模块幂等的方式就是检查模块的最终状态是不是达到,如果最终状态达到了,那么就退出不执行任何操作。如果所有的模块的使用都是幂等的,那么这个playbook本身就是幂等的,因此re-running是安全的。

**module**和**shell**模块返回相同的指令,这对于chmod和setsebool等命令都是ok的。尽管有creates标签可以使这些操作也幂等。

每一个`task`都必须有一个`name`,如果没有提供name,那么action的将会作为任务的输出。使用name是有意义的,因为任务执行后的返回需要保证时人类可读的。

任务可以使用以前的老格式`action: module options`, 但是建议使用更加方便的`module: options`格式,接下来是一个标准的`tasks`的格式。

	tasks:
	  - name: make sure apache is running
	    service:
	      name: httpd
	      state: started

**command**和**shell**是两个不使用`key=value`格式的module,这使他们的操作更加的简单。

	tasks:
	  - name: enable selinux
	    command: /sbin/setenforce 1

**command**和**shell**执行的时候,如果出错了你可以通过这样来屏蔽结果,你可以这样做:

	tasks:
	  - name: run this command and ignore the result
	    shell: /usr/bin/somecommand || /bin/true
	    
或者这样:
	
	tasks:
	  - name: run this command and ignore the result
	    shell: /usr/bin/somecommand
	    ignore_errors: True
如果你命令变长了,你可以使用一个space键和indent来分行。

同样变量也可以使用在task中,比如。
	
	tasks:
	  - name: create a virtual host file for {{ vhost }}
	    template:
	      src: somefile.j2
	      dest: /etc/httpd/conf.d/{{ vhost }}
	
#### <a name="fenced-code-block">Action速记</a>
ansible推荐罗列modules像这样:

	template:
	    src: templates/foo.j2
	    dest: /etc/foo.conf

当然也可以使用以前的语法：
	
	action: template src=templates/foo.j2 dest=/etc/foo.conf

	
#### <a name="fenced-code-block">Handler:执行变化中的操作</a>

playbooks有一个基础的事件系统用来响应变化。这些通知在task块的结尾被触发,并且只会被触发一次。

比如,多个资源表示apache需要被重启因为他们修改了配置文件,但是apache将只会重启一次来避免不必要的重启。

下面是一个一个文件变化后两个服务重启的案例：

	- name: template configuration file
	  template:
	    src: template.j2
	    dest: /etc/foo.conf
	  notify:
	     - restart memcached
	     - restart apache
`notify`会触发事件,重启缓存和apache服务器。

**Handlers**是多个任务的列表,如果没有事件通知handler,它将不会运行。不管多少次通知，handler都只会执行一次, 在所有task都完成之后。

接下来是一个handlers的例子：
	
	handlers:
	    - name: restart memcached
	      service:
	        name: memcached
	        state: restarted
	    - name: restart apache
	      service:
	        name: apache
	        state: restarted
在ansible2.2的时候,handler同样能监听原生topics, tasks也能够通知那些topics。

	handlers:
    - name: restart memcached
      service:
        name: memcached
        state: restarted
      listen: "restart web services"
    - name: restart apache
      service:
        name: apache
        state:restarted
      listen: "restart web services"

	tasks:
	    - name: restart everything
	      command: echo "this task will restart the web services"
	      notify: "restart web services"

如果你希望flush所有的handler命令,你需要这样的命令：
	tasks:
	   - shell: some tasks go here
	   - meta: flush_handlers
	   - shell: some other tasks

#### <a name="fenced-code-block">执行一个playbook</a>

在学习玩playbook语法之后,你可以使用命令来执行playbook了。使用10的并行度。

```
ansible-playbook playbook.yml -f 10
```
#### <a name="fenced-code-block">技巧:使用技巧</a>
1. 使用--syntaxt-check来检查playbook是不是语法正确。
2. 如果你想看细节输出,可以是用--verbose

## 创建可充用的playbooks

在Ansible中,有三种方法来构建可充用的组件:includes,imports,roles。其中includes和imports允许用户去拆分大的playbooks到小的文件,这样就能够被用在多个父类playbook中了。Roles就更加灵活了,可以通过roles文件夹来自动匹配子playbook。并且还能被ansible galaxy共享和上传。


#### <a name="fenced-code-block">import和include</a>
import为静态包含,如`import_playbook, import_tasks`,include为动态包含,include被认为是过时的了。

include是不能notify其他的handler的。
import不能用于循环引用，并host/group vars不能使用。

可以在一个master playbook中include playbooks,比如:
	
	- import_playbook: webservers.yml
	- import_playbook: databases.yml


你能使用import_tasks或者include_tasks去执行tasks在一个main.yml的tasks list中：

	tasks:
	- import_tasks: common_tasks.yml

or
	
	- include_tasks: common_tasks.yml
	
你还可传递参数到imports和includes:

	tasks:
	- import_tasks: wordpress.yml
	  vars:
	    wp_user: timmy
	- import_tasks: wordpress.yml
	  vars:
	    wp_user: alice
	- import_tasks: wordpress.yml
	  vars:
	    wp_user: bob

static和dynamic可以混用,但是不建议这样使用。

handlers也支持import和include。

<mark>more_handlers.yml</mark>

	- name: restart apache
	  service: name=apache state=restarted

如何引用？

	handlers:
	- include_tasks: more_handlers.yml
	- import_tasks: more_handlers.yml

## Roles
#### <a name="fenced-code-block">Role目录结构</a>

项目的结构为:
	
	site.yml
	webservers.yml
	fooservers.yml
	roles/
	   common/
	     tasks/
	     handlers/
	     files/
	     templates/
	     vars/
	     defaults/
	     meta/
	   webservers/
	     tasks/
	     defaults/
	     meta/
正常的目录如下:

* `tasks` - contains the main list of tasks to be executed by the role.
* `handlers` - contains handlers, which may be used by this role or even anywhere outside this role.
* `defaults` - default variables for the role (see Variables for more information).
* `vars` - other variables for the role (see Variables for more information).
* `files` - contains files which can be deployed via this role.
* `templates` - contains templates which can be deployed via this role.
* `meta` - defines some meta data for this role. See below for more details.

其他的YAML file可能被包含在特定的目录下,例如, 这样是非常正常的:

roles/example/tasks/main.yml

	- name: added in 2.4, previously you used 'include'
	  import_tasks: redhat.yml
	  when: ansible_os_platform|lower == 'redhat'
	- import_tasks: debian.yml
	  when: ansible_os_platform|lower == 'debian'

roles/example/tasks/

	- yum:
	    name: "httpd"
	    state: present

roles/example/tasks/

	debian.yml
	- apt:
	    name: "apache2"
	    state: present

#### <a name="fenced-code-block">Roles的使用</a>

经典使用roles的方法如下:

	---
	- hosts: webservers
	  roles:
	     - common
	     - webservers	
在ansible2.4之后,你可以像这样使用roles.比如:

	---
	
	- hosts: webservers
	  tasks:
	  - debug:
	      msg: "before we run our role"
	  - import_role:
	      name: example
	  - include_role:
	      name: example
	  - debug:
	      msg: "after we ran our role"

如果是以经典的方式去定义roles,那么他们将以静态的方式被导入。

roles的名字可以是简单的名字也可以是全限路径名：

	---
	- hosts: webservers
	  roles:
	    - role: '/path/to/my/roles/common'
roles能接受其他的关键字:

	---
	- hosts: webservers
	  roles:
	    - common
	    - role: foo_app_instance
	      vars:
	         dir: '/opt/a'
	         app_port: 5000
	    - role: foo_app_instance
	      vars:
	         dir: '/opt/b'
	         app_port: 5001
或者使用新的语法:

	---
	- hosts: webservers
	  tasks:
	  - include_role:
	       name: foo_app_instance
	    vars:
	      dir: '/opt/a'
	      app_port: 5000
	  ...
你能在条件的状态下引入一个角色,并且执行它的任务:

	---	
	- hosts: webservers
	  tasks:
	  - include_role:
	      name: some_role
	    when: "ansible_os_family == 'RedHat'"
你可能希望打标签在你定义的role中:

	---
	- hosts: webservers
	  tasks:
	  - import_role:
	      name: foo
	    tags:
	    - bar
	    - baz
	
#### <a name="fenced-code-block">Roles去重和执行</a>	
Ansible只会允许role执行一次,即使你定义了多次。

	---
	- hosts: webservers
	  roles:
	  - foo
	  - foo
如果像让role执行多次,有两个选择:
1. 给role传递不同的参数。
2. 添加`allow_duplicates: true`到`meta/main.yml`

例：

	---
	- hosts: webservers
	  roles:
	  - role: foo
	    vars:
	         message: "first"
	  - { role: foo, vars: { message: "second" } }
	

	*playbook.yml
	---
	- hosts: webservers
	  roles:
	  - foo
	  - foo
	
	*roles/foo/meta/main.yml
	---
	allow_duplicates: true
这样foo就能够运行两次了。

#### <a name="fenced-code-block">Roles的默认参数</a>

ansible允许你在dafaults/main.yml中定义默认变量。

#### <a>Roles依赖</a>
role依赖允许你自动pull in其他的roles当你使用role的时候,Role依赖存在meta/main.yml中,例子如下:

	---
	dependencies:
	  - role: common
	    vars:
	      some_parameter: 3
	  - role: apache
	    vars:
	      apache_port: 80
	  - role: postgres
	    vars:
	      dbname: blarg
	      other_parameter: 12

比如,一个叫car的role需要一个叫wheel的role:

	---
	dependencies:
	- role: wheel
	  vars:
	     n: 1
	- role: wheel
	  vars:
	     n: 2
	- role: wheel
	  vars:
	     n: 3
	- role: wheel
	  vars:
	     n: 4
然后wheel又依赖tire和brake.wheel的`meta/main.yml`就需要包含:

	---
	dependencies:
	- role: tire
	- role: brake
然后tire和brake就需要包含:

	---
	allow_duplicates: true
然后最终的结果是

	tire(n=1)
	brake(n=1)
	wheel(n=1)
	tire(n=2)
	brake(n=2)
	wheel(n=2)
	...
	car
考虑到我们不需要使用`allow_duplicates: true`对`wheel`,因为每一个实例car使用不同的参数值。
#### <a>Roles的搜索路径</a>
ansible将会通过以下两种方式来搜索roles.

1. 一个`roles/`目录,与playbook相关
2. 默认,`/etc/ansible/roles

## 变量
虽然自动化非常好,但是不是所有的系统都是一样的,一些需要与其他稍微不同的配置。此时变量variables就派上了用场。

#### <a>变量速记</a>

YAML支持字典将key映射到value,例如:
	
	foo:
	  field1: one
	  field2: two
然后你就可以这样去引用这个变量了:

	foo['field1']
	foo.field1	
python的保留字不要用。

在inventory中定义变量:

	- hosts: webservers
	  vars:
	    http_port: 80
ansible允许使用jinja2的格式系统。如:

```
My amp goes to {{ max_amp_value }}
```
或者你也像这样使用：

```
template: src=foo.cfg.j2 dest={{ remote_install_path }}/foo.cfg
```

<a>在template中你可以获取所有host的scope中的主机。事实上,不仅如此,你还可以获取其他hosts的变量。</mark>

#### <a>变量速记</a>
这样是不起作用的:

	- hosts: app_servers
	  vars:
	      app_path: {{ base_path }}/22
这样才是正确的:

	- hosts: app_servers
	  vars:
	       app_path: "{{ base_path }}/22"
#### <a>Facts:从系统抓去的参数</a>
facts是和系统交互获取的信息,比如:
```
ansible hostname -m setup
```
如果你对你的机器足够了解并且不需要系统参数的情况下,你可以关闭facts

	- hosts: whatever
	  gather_facts: no
#### <a>本地参数</a>	

通常的话,用户可以自己写通用的facts module,但是如果你需要一个简单的方式去提供系统或者用户提供的数据来作为ansible参数,在不使用fact module的情况下。

·facts.d`是一个机制,使用户能够控制一些点关于他们的系统是如何管理的。

在远端机器又一个目录,`/etc/ansible/facts.d'目录,这个目录下所有的文件都是一fact结尾,可以是json也可以是ini,或者是可执行的返回json的文件.这些能够返回本地参数,一个可替换的目录可以是fact_path关键字，如,假设`/etc/ansible/facts.d/preferences.fact`存在。

	[general]
	asdf=1
	bar=2
这个将会提供一个哈希变量fact,叫做general,而asdf和bar作为成员.为了验证这个,接下来可以这样:

```
ansible <hostname> -m setup -a "filter=ansible_local"
```
然后你将会看到：
	
	"ansible_local": {
	        "preferences": {
	            "general": {
	                "asdf" : "1",
	                "bar"  : "2"
	            }
	        }
	 }
然后这个数据可以在`template/playbook`这样获取:

```
{{ ansible_local.preferences.general.asdf }}
```
任何用户都不能覆盖系统facts。

为了更好的使用ansible的version,一个名叫`ansible_version`的变量是可以用的,类似下面的结构:

	"ansible_version": {
	    "full": "2.0.0.2",
	    "major": 2,
	    "minor": 0,
	    "revision": 0,
	    "string": "2.0.0.2"
	}

#### <a>Facts缓存</a>	
可以获取其他机器的参数,如:

```
{{ hostvars['asdf.example.com']['ansible_os_family'] }}
```
Fact caching是默认关闭的,为了拿到上面的参数,ansible必须确保能够talk to asdf.example.com.为了避免这个,ansible1.8之后提供了可以存储facts在playbook执行的过程中,但是这个特性需要手动被开启。为什么这个会有用呢？

当一个集群有上千个主机的时候,facts缓存可被配置在夜间运行.小数量的集群可以配置运行ad-hoc命令,周期性的,当fact caching开启之后,将所有的服务器都去引用fact变量.当fact caching开启之后, 机器可以去引用其他机器的变量,尽管这些变量不属于这些组。

要想收益的话,你会将`gathering`设置到`smart`或`explicit`或者设置`gather_facts`为False在大多数plays中.

目前Ansible支持两种持久化缓存插件,redis和jsonfile.

当你需要配置redis的时候,可以将`ansible.cfg`配置成这样:

	[defaults]
	gathering = smart
	fact_caching = redis
	fact_caching_timeout = 86400
	//#seconds
要想将redis运行起来,使用以下命令的等价操作:

	$ yum install redis
	$ service redis start
	$ pip install redis
考虑到python redis库应该用pip安装,EPEL选择的redis版本太老了。暂时redis是不支持端口和密码的。

如果需要配置jsonfile,可以使用如下的命令：

	[defaults]
	gathering = smart
	fact_caching = jsonfile
	fact_caching_connection = /path/to/cachedir
	fact_caching_timeout = 86400
	//# seconds
`fact_caching_connection`是一个本地的文件系统路径,是可写的,ansible将会试图去创建目录如果这个目录不存在。
`fact_caching_timeout`表示存储缓存的时间,时间一到就不再缓存。
#### <a>注册变量</a>	
另一个变量的作用就是执行一个命令,然后使用这个命令的结果保存到一个变量中.使用-v当执行的playbook将会有返回结果。

任务执行的结果能被保存起来供后面的使用。

进行一个快速的预览:

	- hosts: web_servers
	  tasks:
	
	     - shell: /usr/bin/foo
	       register: foo_result
	       ignore_errors: True
	
	     - shell: /usr/bin/bar
	       when: foo_result.rc == 5
		
注册变量一直有效,和永久变量facts是一样的。高效的注册变量就想facts。

当使用`register`进行循环的时候,数据结构将会包含一个`results`属性,这就是module的一系列响应。

#### <a>获取复杂的变量数据</a>	
当你想获取ip地址的时候,简单的`{{foo}}`不再高效,但是要想获取还是不难的：
`{{ansible_eth0["ipv4"]["address"]}}`
或
`{{ansible_etho0.ipv4.address}}`
同样的,要想获取数组的第一个元素,{{foo[0]}}是可行的。
#### <a>魔术变量,如何去获取其他主机的信息</a>	
ansible给我们提供了非常重要的变量,如`hostvars`,`group_names`,`groups`用户不应该再自己定义这些变量.`environment`也是保留的。

`hostvars`让你能访问其他主机的变量,包括他们获取的facts。如果你没有和那些机器通讯过,你是不能访问的facts的。

如果你的数据库服务器想使用其他服务器的fact,或者一个inventory变量由其他节点指定的,我们可以使用模版来完成:

```
{{ hostvars['test.example.com']['ansible_distribution'] }}
```
`group_names`是这个主机所属的所有组的数组。这些也能使用Jinja2语法去创建模版源文件：

	{% if 'webserver' in group_names %}
	   # some part of a configuration file that only applies to webservers
	{% endif %}

`groups` 是所有groups的集合,可以用这来美剧一个group下的所有主机。例如:

	{% for host in groups['app_servers'] %}
	   # something that applies to all app servers.
	{% endfor %}
一个高频的用法如下,找出一个group中的所有ip address:

	{% for host in groups['app_servers'] %}
	   {{ hostvars[host]['ansible_eth0']['ipv4']['address'] }}
	{% endfor %}
另外`inventory_hostname`是一个配置在ansible inventory主机文件下的。这将变的非常有用当你不想去依靠发现的hostname`ansible_host`或其他的奇怪的原因。还有很多有用的变量都在[variables](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#id27).
	
####<a>变量文件分离</a>
将你的playbooks保持在资源控制下是非常好的想法,但是你可能想使你的playbook公有在使部分变量私有的情况下。相似的,有些时候你只想保持确定的信息在不同的文件而不是在`main playbook`中。
如果你想这样做,你完全可以使用一个外部的变量文件。

	---
	- hosts: all
	  remote_user: root
	  vars:
	    favcolor: blue
	  vars_files:
	    - /vars/external_vars.yml
	  tasks:
	  - name: this is just a placeholder
	    command: /bin/echo foo
这就避免了当你共享你的playbook的时候敏感数据的问题。

这些内容也都是这样定义的。

	---
	//# in the above example, this would be vars/external_vars.yml
	somevar: somevalue
	password: magic
####<a>命令行传递变量</a>	
除了`vars_prompt`和`vars_file`,可以通过使用命令行来设置变量。还可以使用--`extra-vars`或`-e`来设置额外的参数。
	
如kv对格式:

```
nsible-playbook release.yml --extra-vars "version=1.23.45 other_variable=foo"
```
或Json格式

	ansible-playbook release.yml --extra-vars '{"version":"1.23.45","other_variable":"foo"}'
	ansible-playbook arcade.yml --extra-vars '{"pacman":"mrs","ghosts":["inky","pinky","clyde","sue"]}'
或yaml格式:

	ansible-playbook release.yml --extra-vars '
	version: "1.23.45"
	other_variable: foo'
	
	ansible-playbook arcade.yml --extra-vars '
	pacman: mrs
	ghosts:
	- inky
	- pinky
	- clyde
	- sue'
或json或yaml文件:

```
ansible-playbook release.yml --extra-vars "@some_file.json"
```
####<a>变量优先级</a>	
role defaults最小,
最高的是命令行的额外变量。

####<a>变量的作用域</a>
1. `global`:由配置文件,环境变量, 命令行的参数。
2. `play`:role defaults和vars和var files,vars_prompt等等。
3. `host`:直接与host相关的变量,想inventory,include_vars或者注册的任务输出。

变量案例:
`group_vars/all`非常强大：

	---
	//# file: /etc/ansible/group_vars/all
	//# this is the site wide default
	ntp_server: default-time.example.com

`group_vars/region`一般是`all`的子组,将会覆盖更加普通的组:

	---
	//# file: /etc/ansible/group_vars/boston
	ntp_server: boston-time.example.com

一些crazy的人可能还需要指定主机对应的变量。

	---
	//# file: /etc/ansible/host_vars/xyz.boston.example.com
	ntp_server: override.example.com
	
更多细节请参考[variables of playbook](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html)

## 条件式
####<a>Conditionals</a>
通常一个play的结果依赖于一个变量的值,fact或者前一个任务的结果。在某些情况下,这些变量的值依赖于其他的变量。接下来讲如何使用`when`标签。快速记忆:

	tasks:
	  - name: "shut down Debian flavored systems"
	    command: /sbin/shutdown -t now
	    when: ansible_os_family == "Debian"
	    # note that Ansible facts and vars like ansible_os_family can be used
	    # directly in conditionals without double curly braces

	
使用圆括号来拼装条件:

	tasks:
	  - name: "shut down CentOS 6 and Debian 7 systems"
	    command: /sbin/shutdown -t now
	    when: (ansible_distribution == "CentOS" and ansible_distribution_major_version == "6") or
	          (ansible_distribution == "Debian" and ansible_distribution_major_version == "7")

多条件查询我们还可以这样:

	tasks:
	  - name: "shut down CentOS 6 systems"
	    command: /sbin/shutdown -t now
	    when:
	      - ansible_distribution == "CentOS"
	      - ansible_distribution_major_version == "6"
	
如果书哦我们需要忽略错误,跟觉状态的结果是成功还是失败去决定是不是要做某些事。这是非常有用的。

当你想看在特定的机器上能不能做什么:

```
ansible hostname.example.com -m setup
```
有时候你需要给一个string做一个数学运算,你可以这么做。

	tasks:
	  - shell: echo "only on Red Hat 6, derivatives, and later"
	    when: ansible_os_family == "RedHat" and ansible_lsb.major_release|int >= 6
	
定义在playbook或inventory中的变量也能被使用,例子如下:

	vars:
	  epic: true

接下来就可以对这个变量进行判断:

	tasks:
	    - shell: echo "This certainly is epic!"
	      when: epic	
或:

	tasks:
	    - shell: echo "This certainly isn't epic!"
	      when: not epic	
		
如果当一个变量还没有定义的话,你可以这样提醒。

	tasks:
	    - shell: echo "I've got '{{ foo }}' and am not afraid to use it!"
	      when: foo is defined
	
	    - fail: msg="Bailing out. this play requires 'bar'"
	      when: bar is undefined
	
####<a>Loops和Conditions</a>
`loop`的基本用法是这样的:

	tasks:
	    - command: echo {{ item }}
	      loop: [ 0, 2, 4, 6, 8, 10 ]
	      when: item > 5
如果你需要根据变量是否被定义去跳过任务,使用|default([])过滤器。
	
	- command: echo {{ item }}
	  loop: "{{ mylist|default([]) }}"
	  when: item > 5
如果是遍历字典的话,可以是这样的:

	- command: echo {{ item.key }}
	  loop: "{{ query('dict', mydict|default({})) }}"
	  when: item.value > 5
####<a>加载通用facts</a>
如果你想加载外部的facts的话，可以是这样的:

	tasks:
	    - name: gather site specific fact data
	      action: site_facts
	    - command: /usr/bin/thingy
	      when: my_custom_fact_just_retrieved_from_the_remote_system == '1234'
####<a>使用when去决定是否加载其他任务</a>

	- import_tasks: tasks/sometasks.yml
	  when: "'reticulating splines' in output"

或

	- hosts: webservers
	  roles:
	     - role: debian_stock_config
	       when: ansible_os_family == 'Debian'
如果你想在一个变量没有定义的时候引入其他的文件:

	//# include a file to define a variable when it is not already defined
	
	//# main.yml
	- include_tasks: other_tasks.yml
	  when: x is not defined
	
	//# other_tasks.yml
	- set_fact:
	    x: foo
	- debug:
	    var: x

比如你需要根据操作系统的不同.执行不同任务:

	---
	- hosts: all
	  remote_user: root
	  vars_files:
	    - "vars/common.yml"
	    - [ "vars/{{ ansible_os_family }}.yml", "vars/os_defaults.yml" ]
	  tasks:
	  - name: make sure apache is started
	    service: name={{ apache }} state=started	    
上面的例子就是加载readhat.yml,如果加载不到的话,就加载os_default.yml文件,如果还是找不到的话就报错。

####<a>注册变量</a>		    
通常在playbook中需要保存task的结果，那么就使用register吧,字符串的匹配也被包含了:

	- name: test play
	  hosts: all
	
	  tasks:
	
	      - shell: cat /etc/motd
	        register: motd_contents
	
	      - shell: echo "motd contains the word hi"
	        when: motd_contents.stdout.find('hi') != -1
在上面看到了可以通过stout来获取一个变量的内容。可以和loop配合使用:

	- name: registered variable usage as a loop list
	  hosts: all
	  tasks:
	
	    - name: retrieve the list of home directories
	      command: ls /home
	      register: home_dirs
	
	    - name: add home dirs to the backup spooler
	      file:
	        path: /mnt/bkspool/{{ item }}
	        src: /home/{{ item }}
	        state: link
	      loop: "{{ home_dirs.stdout_lines }}"
	      # same as loop: "{{ home_dirs.stdout.split() }}"
当你想查看variable的内容是不是为空的时候:
	
	- name: check registered variable for emptiness
	  hosts: all
	
	  tasks:
	
	      - name: list contents of directory
	        command: ls mydir
	        register: contents
	
	      - name: check contents for emptiness
	        debug:
	          msg: "Directory is empty"
	        when: contents.stdout == ""
     
####<a>常用的facts</a>		          
* `ansible_distribution`

可能的取值：

	Alpine
	Altlinux
	Amazon
	Archlinux
	ClearLinux
	Coreos
	Debian
	Gentoo
	Mandriva
	NA
	OpenWrt
	OracleLinux
	RedHat
	Slackware
	SMGL
	SUSE
	VMwareESX     
* `ansible_distribution_major_version`
操作系统的主版本

* `ansible_os_family`

可能的取值:

	AIX
	Alpine
	Altlinux
	Archlinux
	Darwin
	Debian
	FreeBSD
	Gentoo
	HP-UX
	Mandrake
	RedHat
	SGML
	Slackware
	Solaris
	Suse    
      
## Loops
      
通常你希望在一个任务中做很多事情,比如说创建大量的用户,安装很多包,执行轮训的任务知道一个任务达到。

####<a>标准的循环</a>
如果想节省代码量,创建用户可以这样来做。

	- name: add several users
	  user:
	    name: "{{ item }}"
	    state: present
	    groups: "wheel"
	  loop:
	     - testuser1
	     - testuser2     
      
如果你在YMAL文件中定义了一个变量,或者是`vars`区域,你还可以这样做：

```
loop: "{{ somelist }}"
```
上面的例子等价于：

	- name: add user testuser1
	  user:
	    name: "testuser1"
	    state: present
	    groups: "wheel"
	- name: add user testuser2
	  user:
	    name: "testuser2"
	    state: present
	    groups: "wheel"      
像yum和apt等包管理module可以直接将循环引入它们的option。如:

	- name: optimal yum
	  yum:
	    name: "{{list_of_packages}}"
	    state: present
	
	- name: non optimal yum, not only slower but might cause issues with interdependencies
	  yum:
	    name: "{{item}}"
	    state: present
	  loop: "{{list_of_packages}}"
 考虑到你遍历的对象不一定是简单的字符串,如果你有一个hash的集合.那么你可以这样:
 
	 - name: add several users
	  user:
	    name: "{{ item.name }}"
	    state: present
	    groups: "{{ item.groups }}"
	  loop:
	    - { name: 'testuser1', groups: 'wheel' }
	    - { name: 'testuser2', groups: 'root' }     
 注意当conditional的when和loop配合的时候,when是针对的每一个item.
 
如果你想遍历一个字典的话,使用`dict2items`过滤器:

	- name: create a tag dictionary of non-empty tags
	  set_fact:
	    tags_dict: "{{ (tags_dict|default({}))|combine({item.key: item.value}) }}"
	  loop: "{{ tags|dict2items }}"
	  vars:
	    tags:
	      Environment: dev
	      Application: payment
	      Another: "{{ doesnotexist|default() }}"
	  when: item.value != ""
####<a>复杂的循环</a>      
当你想遍历多个集合的时候,比如,你可以使用jinja2表达式来创建复杂的lists:例如,使用`nested`,你可以组合列表:

	- name: give users access to multiple databases
	  mysql_user:
	    name: "{{ item[0] }}"
	    priv: "{{ item[1] }}.*:ALL"
	    append_privs: yes
	    password: "foo"
	  loop: "{{ ['alice', 'bob'] |product(['clientdb', 'employeedb', 'providerdb'])|list }}"      
####<a>使用lookup vs query</a>          
在Ansible2.5中,新的jinja2语法叫做`query`,这个语法提供了很多新的特性相比lookup来说。接下来的两种表达式是等价的:

	loop: "{{ query('inventory_hostnames', 'all') }}"
	
	loop: "{{ lookup('inventory_hostnames', 'all', wantlist=True) }}"      
     
####<a>Do-until循环</a>         
有时候你想retry一个任务知道某个条件达到了,这里是一个例子:

	- shell: /usr/bin/foo
	  register: result
	  until: result.stdout.find("all systems go") != -1
	  retries: 5
	  delay: 10
默认的retries是3并且delay是5。如果until没有定义的话,那么retries参数被强制为1。

####<a>当在循环中使用register的时候</a>      
当在loop中使用register的时候,结果会包含一个result的属性。当在循环中使用register的时候和不在循环中使用register的时候是不一样的,如:

	- shell: "echo {{ item }}"
	  loop:
	    - "one"
	    - "two"
	  register: echo
结果如下:

	{
	    "changed": true,
	    "msg": "All items completed",
	    "results": [
	        {
	            "changed": true,
	            "cmd": "echo \"one\" ",
	            "delta": "0:00:00.003110",
	            "end": "2013-12-19 12:00:05.187153",
	            "invocation": {
	                "module_args": "echo \"one\"",
	                "module_name": "shell"
	            },
	            "item": "one",
	            "rc": 0,
	            "start": "2013-12-19 12:00:05.184043",
	            "stderr": "",
	            "stdout": "one"
	        },
	        {
	            "changed": true,
	            "cmd": "echo \"two\" ",
	            "delta": "0:00:00.002920",
	            "end": "2013-12-19 12:00:05.245502",
	            "invocation": {
	                "module_args": "echo \"two\"",
	                "module_name": "shell"
	            },
	            "item": "two",
	            "rc": 0,
	            "start": "2013-12-19 12:00:05.242582",
	            "stderr": "",
	            "stdout": "two"
	        }
	    ]
	}	  
如果想查看echo的results.

	- name: Fail if return code is not 0
	  fail:
	    msg: "The command ({{ item.cmd }}) did not have a 0 return code"
	  when: item.rc != 0
	  loop: "{{ echo.results }}"

在迭代的时候，当前item的结果将会被存进变量。

	- shell: echo "{{ item }}"
	  loop:
	    - one
	    - two
	  register: echo
	  changed_when: echo.stdout != "one"

####<a>遍历inventory</a> 
如果你希望遍历inventory,或者遍历它的一个子集。

遍历所有主机:

	//# show all the hosts in the inventory
	- debug:
	    msg: "{{ item }}"
	  loop: "{{ groups['all'] }}"
	
	//# show all the hosts in the current play
	- debug:
	    msg: "{{ item }}"
	  loop: "{{ ansible_play_batch }}"
遍历所有主机名:

	//# show all the hosts in the inventory
	- debug:
	    msg: "{{ item }}"
	  loop: "{{ query('inventory_hostnames', 'all') }}"
	
	//# show all the hosts matching the pattern, ie all but the group www
	- debug:
	    msg: "{{ item }}"
	  loop: "{{ query('inventory_hostnames', 'all!www') }}"	
####<a>Loop control</a> 	
`loop_control`用来指定`item`的名字,避免变量值的覆盖。

		//# main.yml
		- include_tasks: inner.yml
		  loop:
		    - 1
		    - 2
		    - 3
		  loop_control:
		    loop_var: outer_item
		
		//# inner.yml
		- debug:
		    msg: "outer item={{ outer_item }} inner item={{ item }}"
		  loop:
		    - a
		    - b
		    - c	
当你遍历复杂的对象时,label标签就派上用场了:

	- name: create servers
	  digital_ocean:
	    name: "{{ item.name }}"
	    state: present
	  loop:
	    - name: server1
	      disks: 3gb
	      ram: 15Gb
	      network:
	        nic01: 100Gb
	        nic02: 10Gb
	        ...
	  loop_control:
	    label: "{{ item.name }}"
这样就只会遍历`label`字段而不是整个`item`。

你还可以控制执行loop的时间间隔。

	//# main.yml
	- name: create servers, pause 3s before creating next
	  digital_ocean:
	    name: "{{ item }}"
	    state: present
	  loop:
	    - server1
	    - server2
	  loop_control:
	    pause: 3
如果你想track你的loop,你可以使用`index_var`选项来获取当前的index.

	- name: count our fruit
	  debug:
	    msg: "{{ item }} with index {{ my_idx }}"
	  loop:
	    - apple
	    - banana
	    - pear
	  loop_control:
	    index_var: my_idx
在2.5之后建议使用`loop`命令来代替`with_X`

## Block
block允许逻辑任务组合,并且提供error handling.
例:

	tasks:
	   - name: Install Apache
	     block:
	       - yum:
	           name: "{{ item }}"
	           state: installed
	         with_items:
	           - httpd
	           - memcached
	       - template:
	           src: templates/src.j2
	           dest: /etc/foo.conf
	       - service:
	           name: bar
	           state: started
	           enabled: True
	     when: ansible_distribution == 'CentOS'
	     become: true
	     become_user: root
####<a>Error handling</a> 	            

	tasks:
	 - name: Attempt and graceful roll back demo
	   block:
	     - debug:
	         msg: 'I execute normally'
	     - command: /bin/false
	     - debug:
	         msg: 'I never execute, due to the above task failing'
	   rescue:
	     - debug:
	         msg: 'I caught an error'
	     - command: /bin/false
	     - debug:
	         msg: 'I also never execute :-('
	   always:
	     - debug:
	         msg: "This always executes"
这个就类似于`try-catch-finally`。

当收到异常之后,如何处理?

	tasks:
	   - name: Attempt and graceful roll back demo
	     block:
	       - debug:
	           msg: 'I execute normally'
	         notify: run me even after an error
	       - command: /bin/false
	     rescue:
	       - name: make sure all handlers run
	         meta: flush_handlers
	 handlers:
	    - name: run me even after an error
	      debug:
	        msg: 'This handler runs even on error'

## 最终实践

####<a>ansible目录结构</a> 	  		         

	production                # inventory file for production servers
	staging                   # inventory file for staging environment
	
	group_vars/
	   group1.yml             # here we assign variables to particular groups
	   group2.yml
	host_vars/
	   hostname1.yml          # here we assign variables to particular systems
	   hostname2.yml
	
	library/                  # if any custom modules, put them here (optional)
	module_utils/             # if any custom module_utils to support modules, put them here (optional)
	filter_plugins/           # if any custom filter plugins, put them here (optional)
	
	site.yml                  # master playbook
	webservers.yml            # playbook for webserver tier
	dbservers.yml             # playbook for dbserver tier
	
	roles/
	    common/               # this hierarchy represents a "role"
	        tasks/            #
	            main.yml      #  <-- tasks file can include smaller files if warranted
	        handlers/         #
	            main.yml      #  <-- handlers file
	        templates/        #  <-- files for use with the template resource
	            ntp.conf.j2   #  <------- templates end in .j2
	        files/            #
	            bar.txt       #  <-- files for use with the copy resource
	            foo.sh        #  <-- script files for use with the script resource
	        vars/             #
	            main.yml      #  <-- variables associated with this role
	        defaults/         #
	            main.yml      #  <-- default lower priority variables for this role
	        meta/             #
	            main.yml      #  <-- role dependencies
	        library/          # roles can also include custom modules
	        module_utils/     # roles can also include custom module_utils
	        lookup_plugins/   # or other types of plugins, like lookup in this case
	
	    webtier/              # same kind of structure as "common" was above, done for the webtier role
	    monitoring/           # ""
	    fooapp/               # ""
你还可以将group_vars和host_vars分开,如果两者没有什么必然的联系的话。

	inventories/
	   production/
	      hosts               # inventory file for production servers
	      group_vars/
	         group1.yml       # here we assign variables to particular groups
	         group2.yml
	      host_vars/
	         hostname1.yml    # here we assign variables to particular systems
	         hostname2.yml
	
	   staging/
	      hosts               # inventory file for staging environment
	      group_vars/
	         group1.yml       # here we assign variables to particular groups
	         group2.yml
	      host_vars/
	         stagehost1.yml   # here we assign variables to particular systems
	         stagehost2.yml
	
	library/
	module_utils/
	filter_plugins/
	
	site.yml
	webservers.yml
	dbservers.yml
	
	roles/
	    common/
	    webtier/
	    monitoring/
	    fooapp/
这种布局为更大的环境提供了更好的灵活性.

####<a>顶层的playbook被role分开</a>

在`site.yml`中,你可以这样引入其他的role

	---
	//# file: site.yml
	- import_playbook: webservers.yml
	- import_playbook: dbservers.yml

然后在webservers.yml文件中这样。

	---
	//# file: webservers.yml
	- hosts: webservers
	  roles:
	    - common
	    - webtier
我们在执行playbook的时候还可以指定执行的yml文件。

	ansible-playbook site.yml --limit webservers
	ansible-playbook webservers.yml
####<a>Role的Task和Handler组织</a>
下面是一个的common role的例子,这个common role仅仅是确保ntp被开启了。

	---
	//# file: roles/common/tasks/main.yml
	
	- name: be sure ntp is installed
	  yum:
	    name: ntp
	    state: installed
	  tags: ntp
	
	- name: be sure ntp is configured
	  template:
	    src: ntp.conf.j2
	    dest: /etc/ntp.conf
	  notify:
	    - restart ntpd
	  tags: ntp
	
	- name: be sure ntpd is running and enabled
	  service:
	    name: ntpd
	    state: started
	    enabled: yes
	  tags: ntp
接下来是一个handlers文件，handlers仅仅当task报告改变了才会被触发,并且在每个play的结尾运行。

	---
	//# file: roles/common/handlers/main.yml
	- name: restart ntpd
	  service:
	    name: ntpd
	    state: restarted
既然基本的格局已经写好了,那么我们有哪些使用场景可以使用当这种布局设置好了之后?

1. 如果我们想重新哦诶只我们的结构。

```
ansible-playbook -i production site.yml
```
2.如果我们想重新配置NTP

```
ansible-playbook -i production site.yml --tags ntp
```
3.如果我们只想配置webservers

```
ansible-playbook -i production webservers.yml
```
4.希望只配置webservers in boston

```
ansible-playbook -i production webservers.yml --limit boston
```

5.如果仅仅是前10台,机器.

	ansible-playbook -i production webservers.yml --limit boston[0:9]
	ansible-playbook -i production webservers.yml --limit boston[10:19]

6.还有很多有用的命令。

	//# confirm what task names would be run if I ran this command and said "just ntp tasks"
	ansible-playbook -i production webservers.yml --tags ntp --list-tasks
	
	//# confirm what hostnames might be communicated with if I said "limit to boston"
	ansible-playbook -i production webservers.yml --limit boston --list-hosts

当你想给指定的机器设置特定的值的时候。

	---
	//# talk to all hosts just so we can learn about them
	 - hosts: all
	   tasks:
	     - group_by:
	         key: os_{{ ansible_distribution }}
	
	//# now just on the CentOS hosts...
	
	 - hosts: os_CentOS
	   gather_facts: False
	   tasks:
	     - //# tasks that only happen on CentOS go here

如果组特定设置需要的话,这也能做到:

	---
	//# file: group_vars/all
	asdf: 10
	
	---
	//# file: group_vars/os_CentOS
	asdf: 42



























	         
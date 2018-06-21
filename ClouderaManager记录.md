---


---

<h1 id="前言">前言</h1>
<p>以下记录是笔者在使用ClouderaManager的过程中的一些经验心得和踩过的坑，希望能对部门的小伙伴们有所帮助。由于笔者刚开始接触大数据，水平有限，肯定会有不当之处，望各位小伙伴指正。联系方式：<a href="fuqq@itaojin.cn">fuqq@itaojin.cn</a></p>
<h1 id="hive">Hive</h1>
<p>使用ClouderaManager安装Hive。<br>
这里的ClouderaManager版本是5.14，支持的Hive只有2.x系列。</p>
<h2 id="环境准备">环境准备</h2>
<p>默认至少已经具备以下条件：</p>
<ul>
<li>HDFS集群，最好是HA（高可用）</li>
<li>YARN集群，最好是HA</li>
<li>Zookeeper集群</li>
</ul>
<h2 id="安装">安装</h2>
<ul>
<li>
<p>添加新服务<br>
<img src="https://i.imgur.com/YMsmFRX.png" alt="enter image description here"></p>
</li>
<li>
<p>选择Hive<br>
<img src="https://i.imgur.com/FCXorla.png" alt="enter image description here"></p>
</li>
<li>
<p>选择一组依赖<br>
<img src="https://i.imgur.com/ypGHbsR.png" alt="enter image description here"></p>
</li>
<li>
<p>为Hive生态中的每种服务分配节点资源。这里列举了 <code>Gateway</code> , <code>Metastore Server</code> , <code>WebHCat Server</code> , <code>HiveServer2</code> 四种服务。</p>
<ol>
<li>Gateway。目前对其了解的不多，其用途大概是提供修改Hive配置文件的客户端，在选择节点时，笔者是在HiveServer2节点上都创建了该节点，理由是对Hive的配置修改其实就是对HiveServer2服务的配置的修改。</li>
<li>Metastore Service。元数据服务，即Hive存放所有表格，试图，函数等等元数据的服务。由于该服务存储了大量元数据，故而对内存和稳定性要求较高，理论上该服务可以放置于任何节点，最好该节点配置要稍好。</li>
<li>WebHCat Server。未知。</li>
<li>HiveServer2。Hive提供服务的服务。支持JDBC，shell等方式访问服务。对该服务的探索还在继续中，该服务不需要在每个节点上部署，视业务需求而定。		<br>
<img src="https://i.imgur.com/rneuDVI.png" alt="enter image description here"></li>
</ol>
</li>
<li>
<p>为Metastore Server节点配置Mysql数据库。需要事先在某一台节点上的Mysql中创建数据库和用户，推荐跟<code>scm</code>在同一个Mysql。</p>
<blockquote>
<p>create database hive;</p>
</blockquote>
<blockquote>
<p>create user ‘hive’ identified by ‘hive’;</p>
</blockquote>
<blockquote>
<p>grant all privileges on hive.* to ‘hive’@‘localhost’ identified by ‘hive’;</p>
</blockquote>
<blockquote>
<p>grant all privileges on hive.* to ‘hive’@’%’ identified by ‘hive’;</p>
</blockquote>
<blockquote>
<p>flush privileges;</p>
</blockquote>
<p><strong>在这里需要注意，如果是首次安装，还需要完成重要的一步：将Mysql驱动jar添加到<code>/usr/share/java</code>目录下，且改名为<code>mysql-connector-java.jar</code>，即去掉版本号。这是至关重要的一步，不然Hive将不能连接Mysql，也可以推出，Hive是通过JDBC的方式访问Mysql。</strong></p>
<p>驱动下载地址：<a href="http://192.168.1.105:8081/nexus/content/groups/public/mysql/mysql-connector-java/5.1.38/mysql-connector-java-5.1.38.jar">下载地址</a></p>
<p>添加完成后，点击<code>测试连接</code>按钮完成连接测试。</p>
<p><img src="https://i.imgur.com/FssVckP.png" alt="enter image description here"></p>
</li>
<li>
<p>为Hive指定在HDFS中的数据目录，保持默认状态，下一步<br>
<img src="https://i.imgur.com/NVZkItG.png" alt="enter image description here"></p>
</li>
<li>
<p>开始安装<br>
<img src="https://i.imgur.com/pE7kmSV.png" alt="enter image description here"></p>
</li>
<li>
<p>完成<br>
<img src="https://i.imgur.com/di7ar5B.png" alt="enter image description here"></p>
</li>
<li>
<p>回到CM首页即可看到安装完毕的Hive服务<br>
<img src="https://i.imgur.com/wWcJR6I.png" alt="enter image description here"></p>
</li>
</ul>
<h2 id="测试">测试</h2>
<p>在安装了<code>HiveServer2</code>服务的节点上执行以下shell脚本：</p>
<blockquote>
<p>beeline</p>
</blockquote>
<p>调出Hive客户端，连接HiveServer2服务：</p>
<blockquote>
<p>beeline&gt; !connect jdbc:hive2://localhost:10000</p>
</blockquote>
<p>回车两次即可<br>
<img src="https://i.imgur.com/7l3yiL3.png" alt="enter image description here"></p>
<p>展示所有database</p>
<blockquote>
<p>show databases;</p>
</blockquote>
<p>展现出默认<code>default</code>数据库，测试完毕！</p>
<h2 id="注意">注意</h2>
<h3 id="mysql驱动jar">MySQL驱动jar</h3>
<p>这个问题已经在前面讲过，不再赘述。</p>
<h3 id="mysql中user表默认记录导致的一个错误">MySQL中user表默认记录导致的一个错误</h3>
<ul>
<li>
<p>业务场景：ClouderaManager安装Hive时需要一个MySQL数据库存放元数据，于是创建hive库和hive用户，并且授权，操作命令如下：</p>
<blockquote>
<p>create database hive;</p>
</blockquote>
<blockquote>
<p>create user ‘hive’  identified by ‘hive’;</p>
</blockquote>
<blockquote>
<p>grant all privileges on hive.* to ‘hive’@‘localhost’ identified by ‘hive’;</p>
</blockquote>
<blockquote>
<p>grant all privileges on hive.* to ‘hive’@’%’ identified by ‘hive’;</p>
</blockquote>
<blockquote>
<p>flush privileges;</p>
</blockquote>
<p>通过远程连接hive库ok，但是在ClouderaManager上安装Hive时就悲剧了，直接报错：</p>
<pre><code>Logon denied for user/password. Able to find the database server and database, but the login reques was rejected
</code></pre>
<p><img src="https://i.imgur.com/s5VkTDV.png" alt="enter image description here"></p>
</li>
<li>
<p>解决：以root用户登录MySQL，查询user表得知：<br>
<img src="https://i.imgur.com/ZHTLzth.png" alt="enter image description here"></p>
<p>可以看到有两行记录的User列为空，没错，就是因为这个报的错，直接将这两行记录删除即可：</p>
<blockquote>
<p>delete from user where User = ‘’;</p>
</blockquote>
<blockquote>
<p>flush privileges;</p>
</blockquote>
<p><strong>别忘了执行最后一句SQL刷新权限哦</strong></p>
</li>
</ul>
<h3 id="权限问题">权限问题</h3>
<p>CDH根目录下的<code>/user</code>目录权限是用户<code>hdfs</code>所有，hive客户端在HDFS中对应的用户是<code>anonymous</code>，如果使用hive客户端插入数据，会报权限不足的问题：</p>
<pre><code>0: jdbc:hive2://localhost:10000&gt; insert into student values (1, 'zhangsan', 23);
INFO  : Compiling command(queryId=hive_20180620114040_16eb72bd-3d66-46a5-802a-695a5d4963b3): insert into student values (1, 'zhangsan', 23)
INFO  : Semantic Analysis Completed
INFO  : Returning Hive schema: Schema(fieldSchemas:[FieldSchema(name:_col0, type:int, comment:null), FieldSchema(name:_col1, type:string, comment:null), FieldSchema(name:_col2, type:int, comment:null)], properties:null)
INFO  : Completed compiling command(queryId=hive_20180620114040_16eb72bd-3d66-46a5-802a-695a5d4963b3); Time taken: 1.142 seconds
INFO  : Executing command(queryId=hive_20180620114040_16eb72bd-3d66-46a5-802a-695a5d4963b3): insert into student values (1, 'zhangsan', 23)
INFO  : Query ID = hive_20180620114040_16eb72bd-3d66-46a5-802a-695a5d4963b3
INFO  : Total jobs = 3
INFO  : Launching Job 1 out of 3
INFO  : Starting task [Stage-1:MAPRED] in serial mode
INFO  : Number of reduce tasks is set to 0 since there's no reduce operator
ERROR : Job Submission failed with exception 'org.apache.hadoop.security.AccessControlException(Permission denied: user=anonymous, access=WRITE, inode="/user":hdfs:supergroup:drwxr-xr-x
	at org.apache.hadoop.hdfs.server.namenode.DefaultAuthorizationProvider.checkFsPermission(DefaultAuthorizationProvider.java:279)
	at org.apache.hadoop.hdfs.server.namenode.DefaultAuthorizationProvider.check(DefaultAuthorizationProvider.java:260)
	at org.apache.hadoop.hdfs.server.namenode.DefaultAuthorizationProvider.check(DefaultAuthorizationProvider.java:240)
	at org.apache.hadoop.hdfs.server.namenode.DefaultAuthorizationProvider.checkPermission(DefaultAuthorizationProvider.java:162)
	at org.apache.hadoop.hdfs.server.namenode.FSPermissionChecker.checkPermission(FSPermissionChecker.java:152)
	at org.apache.hadoop.hdfs.server.namenode.FSDirectory.checkPermission(FSDirectory.java:3877)
	at org.apache.hadoop.hdfs.server.namenode.FSDirectory.checkPermission(FSDirectory.java:3860)
	at org.apache.hadoop.hdfs.server.namenode.FSDirectory.checkAncestorAccess(FSDirectory.java:3842)
	at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.checkAncestorAccess(FSNamesystem.java:6817)
	at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.mkdirsInternal(FSNamesystem.java:4559)
	at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.mkdirsInt(FSNamesystem.java:4529)
	at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.mkdirs(FSNamesystem.java:4502)
	at org.apache.hadoop.hdfs.server.namenode.NameNodeRpcServer.mkdirs(NameNodeRpcServer.java:884)
	at org.apache.hadoop.hdfs.server.namenode.AuthorizationProviderProxyClientProtocol.mkdirs(AuthorizationProviderProxyClientProtocol.java:328)
	at org.apache.hadoop.hdfs.protocolPB.ClientNamenodeProtocolServerSideTranslatorPB.mkdirs(ClientNamenodeProtocolServerSideTranslatorPB.java:641)
	at org.apache.hadoop.hdfs.protocol.proto.ClientNamenodeProtocolProtos$ClientNamenodeProtocol$2.callBlockingMethod(ClientNamenodeProtocolProtos.java)
	at org.apache.hadoop.ipc.ProtobufRpcEngine$Server$ProtoBufRpcInvoker.call(ProtobufRpcEngine.java:617)
	at org.apache.hadoop.ipc.RPC$Server.call(RPC.java:1073)
	at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:2281)
	at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:2277)
	at java.security.AccessController.doPrivileged(Native Method)
	at javax.security.auth.Subject.doAs(Subject.java:422)
	at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1920)
	at org.apache.hadoop.ipc.Server$Handler.run(Server.java:2275)
</code></pre>
<p>目前还不知道如何根治这个问题，但有一个解决方案：</p>
<p>切换<code>hdfs</code>用户</p>
<blockquote>
<p>su hdfs</p>
</blockquote>
<p>授权</p>
<blockquote>
<p>hdfs dfs -chmod -R 777 /user</p>
</blockquote>
<p>这样做不好的地方就是改变了HDFS文件目录的权限，目前还不知道有没有后遗症，但是解决权限问题。</p>


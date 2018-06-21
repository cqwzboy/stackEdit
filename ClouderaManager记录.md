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
<h3 id="mysql中user表中的默认记录导致的一个错误">MySQL中user表中的默认记录导致的一个错误</h3>
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
<p>ClouderaManager安装HDFS后会在操作系统里创建一个hdfs用户，且HDFS文件系统中的根目录拥有者也是hdfs，例如根目录下的<code>/user</code>目录权限是用户<code>hdfs</code>所有，hive客户端在HDFS中对应的用户是<code>anonymous</code>，如果使用hive客户端插入数据，会报权限不足的问题：</p>
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
<h3 id="安装hive的一个坑">安装Hive的一个坑</h3>
<p>当ClouderaManager第一次安装Hive是好使的，如果将Hive删除后重新安装，会出现一些问题：<strong>Hive会使用内置的derby数据库，即便在创建Hive时指定了外部MySQL数据库</strong>，直接影响到注入sqoop从关系型数据导数据到Hive的成功性，这究竟是为什么呢？</p>
<p>还得先从ClouderaManager环境下Hive的安装目录说起。Hive的安装目录在外部提供的parcels包里，具体位置是：<code>/opt/cloudera/parcels/CDH-5.15.0-1.cdh5.15.0.p0.21/lib/hive</code>，这里将外部parcels放到了 <code>/opt/cloudea</code>文件夹下，根据自己环境的实际安装路径而定。<br>
<img src="https://i.imgur.com/AHvXf1o.png" alt="enter image description here"></p>
<p>从图中可以看到，hive安装目录下的<code>conf</code>是引用了 <code>/etc/hive/conf</code> 的软连接（<em>软连接相关知识百度下，大概意思就是创建一个对某个文件夹的引用，并指向另一个虚拟文件夹，物理文件夹的变动实时反映到虚拟文件夹</em>），我们直接到该目录下一探究竟。<br>
<img src="https://i.imgur.com/c7VYDaa.png" alt="enter image description here"><br>
原来<code>/etc/hive/conf</code>又是同目录下<code>conf.cloudera.hive</code>目录的软连接。其实，刚开始的模样并非如此，这也是这个坑的出处。</p>
<p>当安装好ClouderaManager后，在 <code>/etc/hive</code> 只存在一个软连接文件夹 <code>conf</code>，并且指向 <code>/etc/alternatives/hive-conf</code>，</p>
<p><img src="https://i.imgur.com/iu6H99n.png" alt="enter image description here"><br>
<code>/etc/alternatives/hive-conf</code>目录下的内容是：<br>
<img src="https://i.imgur.com/22gbGm5.png" alt="enter image description here"><br>
查看 <code>hive-site.xml</code> 文件：</p>
<pre><code>&lt;property&gt;
  &lt;name&gt;javax.jdo.option.ConnectionURL&lt;/name&gt;
  &lt;value&gt;jdbc:derby:;databaseName=/var/lib/hive/metastore/metastore_db;create=true&lt;/value&gt;
  &lt;description&gt;JDBC connect string for a JDBC metastore&lt;/description&gt;
&lt;/property&gt;

&lt;property&gt;
  &lt;name&gt;javax.jdo.option.ConnectionDriverName&lt;/name&gt;
  &lt;value&gt;org.apache.derby.jdbc.EmbeddedDriver&lt;/value&gt;
  &lt;description&gt;Driver class name for a JDBC metastore&lt;/description&gt;
&lt;/property&gt;
</code></pre>
<p>该配置文件只定义了Hive的内置derby数据库，这也是Hive在ClouderaManager环境下的初始配置。</p>
<p>当第一次安装并启动Hive时，<code>/etc/hive/</code>目录下会产生一个文件夹 <code>conf.cloudera.hive</code>，该文件夹推测是ClouderaManager结合当前环境整合的一个Hive配置集合，进入目录：<br>
<img src="https://i.imgur.com/Y3XQ2QD.png" alt="enter image description here"><br>
可以看到，新产生的目录下存放着hadoop的一些配置文件和一个<code>hive-site.xml</code>文件，查看<code>hive-site.xml</code>：</p>
<pre><code>&lt;configuration&gt;
  &lt;property&gt;
    &lt;name&gt;hive.metastore.uris&lt;/name&gt;
    &lt;value&gt;thrift://node5:9083&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.metastore.client.socket.timeout&lt;/name&gt;
    &lt;value&gt;300&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.metastore.warehouse.dir&lt;/name&gt;
    &lt;value&gt;/user/hive/warehouse&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.warehouse.subdir.inherit.perms&lt;/name&gt;
    &lt;value&gt;true&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.auto.convert.join&lt;/name&gt;
    &lt;value&gt;true&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.auto.convert.join.noconditionaltask.size&lt;/name&gt;
    &lt;value&gt;20971520&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.optimize.bucketmapjoin.sortedmerge&lt;/name&gt;
    &lt;value&gt;false&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.smbjoin.cache.rows&lt;/name&gt;
    &lt;value&gt;10000&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.server2.logging.operation.enabled&lt;/name&gt;
    &lt;value&gt;true&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.server2.logging.operation.log.location&lt;/name&gt;
    &lt;value&gt;/var/log/hive/operation_logs&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;mapred.reduce.tasks&lt;/name&gt;
    &lt;value&gt;-1&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.exec.reducers.bytes.per.reducer&lt;/name&gt;
    &lt;value&gt;67108864&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.exec.copyfile.maxsize&lt;/name&gt;
    &lt;value&gt;33554432&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.exec.reducers.max&lt;/name&gt;
    &lt;value&gt;1099&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.vectorized.groupby.checkinterval&lt;/name&gt;
    &lt;value&gt;4096&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.vectorized.groupby.flush.percent&lt;/name&gt;
    &lt;value&gt;0.1&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.compute.query.using.stats&lt;/name&gt;
    &lt;value&gt;false&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.vectorized.execution.enabled&lt;/name&gt;
    &lt;value&gt;true&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.vectorized.execution.reduce.enabled&lt;/name&gt;
    &lt;value&gt;false&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.merge.mapfiles&lt;/name&gt;
    &lt;value&gt;true&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.merge.mapredfiles&lt;/name&gt;
    &lt;value&gt;false&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.cbo.enable&lt;/name&gt;
    &lt;value&gt;false&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.fetch.task.conversion&lt;/name&gt;
    &lt;value&gt;minimal&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.fetch.task.conversion.threshold&lt;/name&gt;
    &lt;value&gt;268435456&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.limit.pushdown.memory.usage&lt;/name&gt;
    &lt;value&gt;0.1&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.merge.sparkfiles&lt;/name&gt;
    &lt;value&gt;true&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.merge.smallfiles.avgsize&lt;/name&gt;
    &lt;value&gt;16777216&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.merge.size.per.task&lt;/name&gt;
    &lt;value&gt;268435456&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.optimize.reducededuplication&lt;/name&gt;
    &lt;value&gt;true&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.optimize.reducededuplication.min.reducer&lt;/name&gt;
    &lt;value&gt;4&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.map.aggr&lt;/name&gt;
    &lt;value&gt;true&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.map.aggr.hash.percentmemory&lt;/name&gt;
    &lt;value&gt;0.5&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.optimize.sort.dynamic.partition&lt;/name&gt;
    &lt;value&gt;false&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.execution.engine&lt;/name&gt;
    &lt;value&gt;mr&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;spark.executor.memory&lt;/name&gt;
    &lt;value&gt;3039297536&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;spark.driver.memory&lt;/name&gt;
    &lt;value&gt;966367641&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;spark.executor.cores&lt;/name&gt;
    &lt;value&gt;4&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;spark.yarn.driver.memoryOverhead&lt;/name&gt;
    &lt;value&gt;102&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;spark.yarn.executor.memoryOverhead&lt;/name&gt;
    &lt;value&gt;511&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;spark.dynamicAllocation.enabled&lt;/name&gt;
    &lt;value&gt;true&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;spark.dynamicAllocation.initialExecutors&lt;/name&gt;
    &lt;value&gt;1&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;spark.dynamicAllocation.minExecutors&lt;/name&gt;
    &lt;value&gt;1&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;spark.dynamicAllocation.maxExecutors&lt;/name&gt;
    &lt;value&gt;2147483647&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.metastore.execute.setugi&lt;/name&gt;
    &lt;value&gt;true&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.support.concurrency&lt;/name&gt;
    &lt;value&gt;true&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.zookeeper.quorum&lt;/name&gt;
    &lt;value&gt;node2,node6,node5&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.zookeeper.client.port&lt;/name&gt;
    &lt;value&gt;2181&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.zookeeper.namespace&lt;/name&gt;
    &lt;value&gt;hive_zookeeper_namespace_hive&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.cluster.delegation.token.store.class&lt;/name&gt;
    &lt;value&gt;org.apache.hadoop.hive.thrift.MemoryTokenStore&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.server2.enable.doAs&lt;/name&gt;
    &lt;value&gt;true&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;hive.server2.use.SSL&lt;/name&gt;
    &lt;value&gt;false&lt;/value&gt;
  &lt;/property&gt;
  &lt;property&gt;
    &lt;name&gt;spark.shuffle.service.enabled&lt;/name&gt;
    &lt;value&gt;true&lt;/value&gt;
  &lt;/property&gt;
&lt;/configuration&gt;
</code></pre>
<p>ClouderaManager已经为我们生成了一份Hive配置文件，并且 <code>/etc/hive/conf</code> 目录的软连接指向已经变成了这个新增的目录 <code>/etc/hive/conf.cloudera.hive</code>，显而易见，Hive已经将配置文件路径指向了新生成的目录下。当我们自己搭建源生Hive环境时，以上配置也可以作为有力参考。配置中着重要注意第一个配置<code>hive.metastore.uris</code>，此配置是指定元数据服务器路径。</p>
<p>讲了这么多，以上内容只是内容铺垫，下面正式进入主题。</p>
<p>当把已经装好的Hive服务从ClouderaManager平台移除，且重新安装时，问题就来了，<code>/etc/hive/conf</code>软连接地址又指回了初始值 <code>/etc/alternatives/hive-conf</code>，使得Hive服务从内置数据库derby寻找数据源，从而报找不到数据库的错误，其本质就是找不到元数据服务了，所以解决办法就是手动创建软连接，重新指向同目录下的<code>conf.cloudera.hive</code>目录：</p>
<blockquote>
<p>cd /etc/hive/conf;</p>
</blockquote>
<blockquote>
<p>rm -rf conf;</p>
</blockquote>
<blockquote>
<p>ln -s conf.cloudera.hive conf;</p>
</blockquote>
<p>重启Hive服务即可。</p>
<p>找不到数据库报错会影响以下两方面流程：</p>
<ul>
<li>不能从外部直接导数据到Hive，例如sqoop</li>
<li>当Hive和HBase存在关联表时，也不能从外部导数据到HBase</li>
</ul>


---


---

<h1 id="原理和架构">原理和架构</h1>
<p>原文链接：<br>
<a href="https://blog.csdn.net/matthewei6/article/details/50554472">https://blog.csdn.net/matthewei6/article/details/50554472</a><br>
<a href="http://www.infoq.com/cn/articles/introductionOozie/">http://www.infoq.com/cn/articles/introductionOozie/</a><br>
<a href="https://www.cnblogs.com/juncaoit/p/6122187.html">https://www.cnblogs.com/juncaoit/p/6122187.html</a></p>
<h2 id="oozie是什么">Oozie是什么</h2>
<p>Oozie众多大数据工作流调度框架中的一种。</p>
<p>大数据工作流调度概念：一组复杂的计算需要多个环节组成，每个环节用到大数据生态中的一个或者多个组件配合完成，工作流调度工作就是将这些纷繁复杂的组件有序的组合并完成任务。</p>
<h2 id="同类">同类</h2>
<ul>
<li>
<p>Oozie （已经成为hadoop标配）<br>
Yahoo！开源，基于xml表达作业依赖关系；</p>
</li>
<li>
<p>Azkaban<br>
Linkedin开源，通过Java property配置作业依赖关系</p>
</li>
<li>
<p>Zeus（宙斯） （据说不再更新）<br>
阿里开源，通过界面配置作业依赖关系</p>
</li>
<li>
<p>其他开源系统<br>
Cascading（通过Java API编程实现作业依赖关系）</p>
</li>
</ul>
<h2 id="oozie三大功能">Oozie三大功能</h2>
<ul>
<li><strong>Oozie Workflow jobs</strong> ：工作流任务，可以生成DAG图</li>
<li><strong>Oozie Coordinator jobs</strong>：可以定时调度</li>
<li><strong>Oozie Bundle</strong>：多个coordinator的集合，或者多个workflow的集合</li>
</ul>
<h2 id="oozie节点">Oozie节点</h2>
<p>节点，即完成每一步工作的具体执行者。<br>
分类：</p>
<ul>
<li>控制流节点：起始，分支，并发，汇合，结束</li>
<li>动作节点（action）：执行的job。例如，mapreduce action，hive action ，shell action</li>
</ul>
<p><img src="https://i.imgur.com/Yct0J3Y.png" alt="enter image description here"></p>
<h2 id="架构">架构</h2>
<p><img src="https://i.imgur.com/bb7UYwx.png" alt="enter image description here"></p>
<ul>
<li>左边：Oozie提供完整的API接口，包括java API，REST API，webUI（只读），Oozie CLI 等等</li>
<li>中间：Oozie服务器是一个java的tomcat服务，可以直线<strong>三大功能</strong></li>
<li>右边：直接操作HDFS和MR</li>
</ul>
<h2 id="运行流程">运行流程</h2>
<p><img src="https://i.imgur.com/od4WA5K.jpg" alt="enter image description here"></p>
<h2 id="hpdl">hPDL</h2>
<p>Oozie工作流是放置在控制依赖DAG（有向无环图 Direct Acyclic Graph）中的一组动作（例如，Hadoop的Map/Reduce作业、Pig作业等），其中指定了动作执行的顺序。我们会使用hPDL（一种XML流程定义语言）来描述这个图。</p>
<p>hPDL是一种很简洁的语言，只会使用少数流程控制和动作节点。控制节点会定义执行的流程，并包含工作流的起点和终点（start、end和fail节点）以及控制工作流执行路径的机制（decision、fork和join节点）。动作节点是一些机制，通过它们工作流会触发执行计算或者处理任务。Oozie为以下类型的动作提供支持： Hadoop map-reduce、Hadoop文件系统、Pig、Java和Oozie的子工作流（SSH动作已经从Oozie schema 0.2之后的版本中移除了）。</p>
<p>所有由动作节点触发的计算和处理任务都不在Oozie之中——它们是由Hadoop的Map/Reduce框架执行的。这种方法让Oozie可以支持现存的Hadoop用于负载平衡、灾难恢复的机制。这些任务主要是异步执行的（只有文件系统动作例外，它是同步处理的）。这意味着对于大多数工作流动作触发的计算或处理任务的类型来说，在工作流操作转换到工作流的下一个节点之前都需要等待，直到计算或处理任务结束了之后才能够继续。Oozie可以通过两种不同的方式来检测计算或处理任务是否完成，也就是回调和轮询。当Oozie启动了计算或处理任务的时候，它会为任务提供唯一的回调URL，然后任务会在完成的时候发送通知给特定的URL。在任务无法触发回调URL的情况下（可能是因为任何原因，比方说网络闪断），或者当任务的类型无法在完成时触发回调URL的时候，Oozie有一种机制，可以对计算或处理任务进行轮询，从而保证能够完成任务。</p>
<h1 id="安装">安装</h1>
<p>原文链接：<br>
<a href="https://www.cnblogs.com/duanxingxing/p/5015709.html">https://www.cnblogs.com/duanxingxing/p/5015709.html</a><br>
<a href="https://blog.csdn.net/u014729236/article/details/47188631">https://blog.csdn.net/u014729236/article/details/47188631</a></p>
<p>对比大数据生态的其他组件的安装过程，Oozie的安装过程是略显复杂的，最突出的一点就是Apache官方只提供Oozie的源码，不提供已经打好的tar包，而且编译过程有很多不确定性因素，这就抬高了门槛。</p>
<p>经验之谈：在安装各个组件时，建议不要安装最新的版本，不管是从兼容性考虑，还是从参考资料的完整性考虑皆是如此，秉承<strong>够用原则</strong>。</p>
<h2 id="环境介绍：">环境介绍：</h2>
<ul>
<li>JDK1.8</li>
<li>Maven  3.5.3</li>
<li>Hadoop 2.6.0集群</li>
<li>Oozie 4.3.0</li>
</ul>
<h2 id="下载源码">下载源码</h2>
<p>直接到官网下载：<a href="http://archive.apache.org/dist/oozie/4.3.0/">http://archive.apache.org/dist/oozie/4.3.0/</a></p>
<blockquote>
<p>cd /usr/local/share/applications;<br>
wget <a href="http://archive.apache.org/dist/oozie/4.3.0/oozie-4.3.0.tar.gz;">http://archive.apache.org/dist/oozie/4.3.0/oozie-4.3.0.tar.gz;</a><br>
tar -zxvf oozie-4.3.0.tar.gz;<br>
mv oozie-4.3.0 oozie-4.3.0-main;</p>
</blockquote>
<p>之所以要将oozie-4.3.0重命名，是因为后面会重新编译并解压出一个oozie-4.3.0文件夹，为了避免覆盖。</p>
<h2 id="检查maven配置">检查Maven配置</h2>
<p>如果机器上安装的Maven配置了第三方源或者私服，请屏蔽，因为Oozie的源码里已经指定了maven中央仓库源，且Oozie的很多依赖jar只有中央仓库才有。<strong>如果没有配置第三方源或者私服，跳过此步。</strong></p>
<p>修改命令：</p>
<blockquote>
<p>vim $MAVEN_HOME/conf/settings.xml</p>
</blockquote>
<ul>
<li>将<strong>mirrors</strong>节点下镜像屏蔽</li>
<li>将<strong>activeProfiles</strong>开关屏蔽</li>
</ul>
<h2 id="编译">编译</h2>
<blockquote>
<p>cd ./oozie-4.3.0-main/bin;<br>
./mkdistro.sh -DskipTests;</p>
</blockquote>
<p>进入源码根目录下的bin目录，执行编译，<strong>-DskipTests</strong>为跳过编译。<br>
编译的过程很漫长，大概要持续<strong>15~20</strong>分钟。并且编译过程中会报找不到一些jar，<strong>只要编译过程没有中断，可以不理</strong>。</p>
<p>Tips：</p>
<blockquote>
<p>刚开始Hadoop的版本是很新的2.9.0，在编译oozie的过程中发现很多jar找不到，于是我将Hadoop的版本降为2.6.0才正常了。</p>
</blockquote>
<h2 id="安装oozie-server">安装Oozie Server</h2>
<h3 id="解压">解压</h3>
<p>完成<strong>编译</strong>后，会在 <code>/usr/local/share/applications/oozie-4.3.0-main/distro/target</code> 下生成一个tar包：<strong>oozie-4.3.0-distro.tar.gz</strong>，没错，这就是编译好的安装包，将其解压到与oozie-4.3.0-main平级的目录下：</p>
<blockquote>
<p>cd /usr/local/share/applications/oozie-4.3.0-main/distro/target tar;<br>
-zxvf oozie-4.3.0-distro.tar.gz -C /usr/local/share/applications/;</p>
</blockquote>
<p>我们在<code>/usr/local/share/applications/</code>目录下可以看到有一个<strong>oozie-4.3.0</strong> 的文件夹</p>
<h3 id="环境变量">环境变量</h3>
<p>将Oozie的根目录加到/etc/profile文件中</p>
<blockquote>
<p>vim /etc/profile<br>
<img src="https://i.imgur.com/SozCIWJ.png" alt="enter image description here"><br>
source /etc/profile</p>
</blockquote>
<h3 id="新建libext">新建libext</h3>
<p>进入<strong>oozie-4.3.0</strong>，新建文件夹<strong>libext</strong>，此文件夹存放Oozie Server依赖的jar，且名称只能是<strong>libext</strong>，不可更改，应该是源码中会依据此名称文件夹进行解析。</p>
<blockquote>
<p>cd /usr/local/share/applications/oozie-4.3.0 ;<br>
mkdir libext;</p>
</blockquote>
<p>jar依赖一共分为三种类型：</p>
<h4 id="ext-2.2.zip">ext-2.2.zip</h4>
<p>为Oozie的webUI提供JS<br>
下载地址：<a href="https://ext4all.com/post/how-to-download-extjs-2.html">https://ext4all.com/post/how-to-download-extjs-2.html</a></p>
<blockquote>
<p>cd libext;<br>
wget <a href="https://ext4all.com/ext/download/ext-2.2.zip">https://ext4all.com/ext/download/ext-2.2.zip</a> ;<br>
unzip ext-2.2.zip;</p>
</blockquote>
<p>如果报不识别<strong>unzip</strong>，先安装unzip，centOS命令如下：</p>
<blockquote>
<p>yum install zip unzip -y;</p>
</blockquote>
<h4 id="mysql-connector-java-5.1.32.jar">mysql-connector-java-5.1.32.jar</h4>
<p>由于Oozie的任务数据存放在数据库中，这里我们采用Mysql数据库存放，所以需要导入驱动jar</p>
<pre><code>wget http://itaojin105:8081/nexus/content/groups/public/mysql/mysql-connector-java/5.1.32/mysql-connector-java-5.1.32.jar;
</code></pre>
<p>这里是从<strong>itaojin105</strong>的Maven私服上获取，也可以到mysql官网下载，或者到本地maven库中拷贝一份。</p>
<h4 id="hadoop-jar">Hadoop jar</h4>
<p>将Hadoop安装目录下的<strong>share</strong>目录下的所有jar拷贝过来</p>
<blockquote>
<p>cp  ${HADOOP_HOME}/share/hadoop/<em>/</em>.jar  . ;<br>
cp ${HADOOP_HOME}/share/hadoop/<em>/lib/</em>.jar  .;</p>
</blockquote>
<h4 id="坑">坑</h4>
<p>这里有个大坑，oozie server默认使用<code>tomcat 6.0.41</code>，而hadoop也有内置的server，如果按照上面两个命令把hadoop依赖的jar包都拷贝过去，有可能出现冲突，这两个server使用的servlet、jsp版本很可能不一样。</p>
<p>这里需要把这几个jar包删除，不要放到libext中</p>
<blockquote>
<p>rm jasper-compiler-5.5.23.jar ;<br>
rm  jasper-runtime-5.5.23.jar ;<br>
rm  jsp-api-2.1.jar;</p>
</blockquote>
<h3 id="配置">配置</h3>
<p>前面说到，Oozie会把除了运行日志以外的所有日志全部存放在第三方数据库中，这里我们选用MySQL，所以需要再配置文件中配置MySQL。</p>
<h4 id="创建数据库用户">创建数据库用户</h4>
<p>注意：oozie-site.xml中有个配置项<code>oozie.service.JPAService.create.db.schema</code>，默认值为false，表示非自动创建数据库，所以我们需要自己创建oozie数据库。</p>
<p>用root用户登录mysql</p>
<blockquote>
<p>create user ‘oozie’ identified by ‘oozie’;<br>
create database oozie;<br>
grant all privileges on oozie.* to ‘oozie’@’%’ identified by ‘oozie’;<br>
grant all privileges on oozie.* to ‘oozie’@‘localhost’ identified by ‘oozie’;<br>
flush privileges;</p>
</blockquote>
<h4 id="配置-1">配置</h4>
<blockquote>
<p>cd /usr/local/share/applications/oozie-4.3.0/conf;<br>
vim oozie-site.xml;</p>
</blockquote>
<p><strong>oozie-site.xml</strong></p>
<pre><code>&lt;configuration&gt;
	&lt;!--mysql作为元数据存放的数据库--&gt;
	&lt;property&gt;
		&lt;name&gt;oozie.service.JPAService.jdbc.driver&lt;/name&gt;
		&lt;value&gt;com.mysql.jdbc.Driver&lt;/value&gt;
		&lt;description&gt;
			JDBC driver class.
		&lt;/description&gt;
	&lt;/property&gt;
	&lt;property&gt;
		&lt;name&gt;oozie.service.JPAService.jdbc.url&lt;/name&gt;
		&lt;value&gt;jdbc:mysql://itaojin101:3306/oozie&lt;/value&gt;
		&lt;description&gt;
			JDBC URL.
		&lt;/description&gt;
	&lt;/property&gt;
	&lt;property&gt;
		&lt;name&gt;oozie.service.JPAService.jdbc.username&lt;/name&gt;
		&lt;value&gt;oozie&lt;/value&gt;
		&lt;description&gt;
			DB user name.
		&lt;/description&gt;
	&lt;/property&gt;
	&lt;property&gt;
		&lt;name&gt;oozie.service.JPAService.jdbc.password&lt;/name&gt;
		&lt;value&gt;oozie&lt;/value&gt;
		&lt;description&gt;
			DB user password.
			IMPORTANT: if password is emtpy leave a 1 space string, the service trims the value,
			if empty Configuration assumes it is NULL.
		&lt;/description&gt;
	&lt;/property&gt;
	&lt;!--设置Hadoop的配置文件的路径--&gt;
	&lt;property&gt;
		&lt;name&gt;oozie.service.HadoopAccessorService.hadoop.configurations&lt;/name&gt;
		&lt;value&gt;*=/usr/local/share/applications/hadoop-2.6.0/etc/hadoop&lt;/value&gt;
		&lt;description&gt;
			Comma separated AUTHORITY=HADOOP_CONF_DIR, where AUTHORITY is the HOST:PORT of
			the Hadoop service (JobTracker, YARN, HDFS). The wildcard '*' configuration is
			used when there is no exact match for an authority. The HADOOP_CONF_DIR contains
			the relevant Hadoop *-site.xml files. If the path is relative is looked within
			the Oozie configuration directory; though the path can be absolute (i.e. to point
			to Hadoop client conf/ directories in the local filesystem.
		&lt;/description&gt;
	&lt;/property&gt;
	&lt;!--设置Spark的配置文件的路径--&gt;
	&lt;!--
	&lt;property&gt;
		&lt;name&gt;oozie.service.SparkConfigurationService.spark.configurations&lt;/name&gt;
		&lt;value&gt;*=/opt/spark-1.4.0-bin-hadoop2.6-hive/conf&lt;/value&gt;
		&lt;description&gt;
			Comma separated AUTHORITY=SPARK_CONF_DIR, where AUTHORITY is the HOST:PORT of
			the ResourceManager of a YARN cluster. The wildcard '*' configuration is
			used when there is no exact match for an authority. The SPARK_CONF_DIR contains
			the relevant spark-defaults.conf properties file. If the path is relative is looked within
			the Oozie configuration directory; though the path can be absolute.  This is only used
			when the Spark master is set to either "yarn-client" or "yarn-cluster".
		&lt;/description&gt;
	&lt;/property&gt;
	--&gt;
	&lt;!--
	设置系统库存放在hdfs中，注意只有在job.properties中将设置oozie.use.system.libpath=true才会引用系统库
	。注意，下面mycluster是namenode的逻辑名称，根据自己集群的情况进行更改即可--&gt;
	&lt;property&gt;
		&lt;name&gt;oozie.service.WorkflowAppService.system.libpath&lt;/name&gt;
		&lt;value&gt;hdfs://mycluster/user/${user.name}/share/lib&lt;/value&gt;
		&lt;description&gt;
			System library path to use for workflow applications.
			This path is added to workflow application if their job properties sets
			the property 'oozie.use.system.libpath' to true.
		&lt;/description&gt;
	&lt;/property&gt;
&lt;/configuration&gt;
</code></pre>
<p>重点说一下最后两项配置（不含被注释掉的Spark配置）</p>
<ul>
<li>设置Hadoop的配置文件的路径，需要将Hadoop的配置文件路径改成自己的。</li>
<li>该配置项是配置HDFS中的共享jar，这里先明白有这么回事儿，后面步骤会配置。注意，<strong>${<a href="http://user.name">user.name</a>}</strong>，这个用户名是Oozie在HDFS中的执行用户，可配置的，这里我采用<strong>root</strong>账户，需要配置Hadoop中的<strong>core-site.xml</strong>文件。</li>
</ul>
<p><strong>${HADOOP_HOME}/etc/hadoop/core-site.xml</strong><br>
新增：</p>
<pre><code>&lt;!-- OOZIE --&gt;
&lt;property&gt;
	&lt;name&gt;hadoop.proxyuser.root.hosts&lt;/name&gt;
	&lt;value&gt;itaojin101&lt;/value&gt;
&lt;/property&gt;
&lt;property&gt;
	&lt;name&gt;hadoop.proxyuser.root.groups&lt;/name&gt;
	&lt;value&gt;root&lt;/value&gt;
&lt;/property&gt;
</code></pre>
<p><strong>配置完成后，一定要重启Hadoop使配置生效</strong></p>
<h2 id="打war包">打war包</h2>
<p>经过以上的配置后就可以将Oozie Server打包运行了。</p>
<blockquote>
<p>cd /usr/local/share/applications/oozie-4.3.0/bin;<br>
<a href="http://oozie-setup.sh">oozie-setup.sh</a>  prepare-war;</p>
</blockquote>
<h2 id="初始化mysql">初始化Mysql</h2>
<blockquote>
<p><a href="http://ooziedb.sh">ooziedb.sh</a> create -sqlfile oozie.sql -run;</p>
</blockquote>
<p>完成此步后会在MySQL的oozie数据库下生成一些表格，这些表格就是用来存放OozieServer运行数据的。<br>
<img src="https://i.imgur.com/VXbatBO.png" alt="enter image description here"></p>
<h2 id="安装oozie-sharelib">安装oozie-sharelib</h2>
<ul>
<li>在oozie-4.3.0下有一个<strong>oozie-sharelib-4.3.0.tar.gz</strong>包，将其解压，生成<strong>share</strong>目录。</li>
</ul>
<blockquote>
<p>tar -zxvf oozie-sharelib-4.3.0.tar.gz;</p>
</blockquote>
<ul>
<li>将Mysql的驱动jar添加到<code>../share/lib/sqoop</code>下，如果没有该jar，Oozie的sqoop作业将受限。当然，如果导入导出的数据库是oracle就导入oracle的驱动jar。</li>
<li>将<strong>share</strong>文件夹上传到HDFS的<code>/user/root</code>目录下，因为前面配置的Oozie在HDFS中的操作用户是<strong>root</strong>，所以放在root目录下。</li>
</ul>
<blockquote>
<p>hdfs dfs -put /usr/local/share/applications/oozie-4.3.0/share  /user/root;</p>
</blockquote>
<h2 id="启动oozie-server">启动Oozie Server</h2>
<blockquote>
<p><a href="http://oozie-start.sh">oozie-start.sh</a>;</p>
</blockquote>
<p>这时在/opt/oozie-4.2.0/oozie-server/webapps目录下多了oozie这个目录。如果缺少jar包或者jar包冲突了可以对$OOZIE_HOME/oozie-server/webapps/oozie/WEB-INF/lib中的进行添加或删除jar包</p>
<h2 id="验证">验证</h2>
<ul>
<li>使用以下命令验证成功与否</li>
</ul>
<blockquote>
<p>oozie admin -oozie <a href="http://localhost:11000/oozie">http://localhost:11000/oozie</a> -status;</p>
</blockquote>
<p>如果是<strong>System model:Normal</strong>，表明启动成功，否则失败。</p>
<ul>
<li>访问<strong><a href="http://itaojin101:11000/oozie">http://itaojin101:11000/oozie</a></strong><br>
<img src="https://i.imgur.com/J3zAzHr.png" alt="enter image description here"></li>
</ul>
<h2 id="注意">注意</h2>
<p>在运行job之前，最好开启Hadoop的jobhistory。它可以帮助你查看job调度时产生的日志。启动命令：</p>
<blockquote>
<p><a href="http://mr-jobhistory-daemon.sh">mr-jobhistory-daemon.sh</a> start historyserver;</p>
</blockquote>
<p>使用jps命令查看是否启动成功，如果出现了 JobHistoryServer进程，表示启动成功。</p>
<p><strong>在上面修改了Hadoop中的core-site.xml配置文件后还未重启Hadoop，请速速重启。</strong></p>
<h2 id="client安装">Client安装</h2>
<p>Oozie server 安装中已经包括了Oozie client。如果想要在其他机子上也使用Oozie，那么只要在那些机子上安装Oozei的client即可。</p>
<ul>
<li>将<code>/usr/local/share/applications/oozie-4.3.0</code>目录下的<strong>oozie-client-4.3.0.tar.gz</strong>远程拷贝到 itaojin102 机器上</li>
</ul>
<blockquote>
<p>scp -r $OOZIE_HOME/oozie-client-4.3.0.tar.gz root@itaojin102:/usr/local/share/applications/;</p>
</blockquote>
<ul>
<li>切换到itaojin102机器上，解压</li>
</ul>
<blockquote>
<p>cd /usr/local/share/applications/; 	<br>
tar -zxvf oozie-client-4.3.0.tar.gz;</p>
</blockquote>
<ul>
<li>
<p>设置环境变量<br>
<img src="https://i.imgur.com/N2RIBvr.png" alt="enter image description here"><br>
<strong>注意</strong>：上面还可以添加一个环境变量，export OOZIE_URL=http://itaojin101:11000/oozie这样在后面的oozie job这个命令中就不需要加 -oozie了</p>
</li>
<li>
<p>就可以通过<code>oozie job -oozie http://itaojin101:11000/oozie -config ....../job.properties -run</code>开启一个任务了。<br>
这里的任务只是占位符，想要真正运行一个任务，往下看。</p>
</li>
</ul>
<h1 id="oozie-examples">Oozie Examples</h1>
<p>Oozie自带了一些示例job，我们可以来尝试一下。</p>
<h2 id="解压-1">解压</h2>
<p>在<code>/usr/local/share/applications/oozie-4.3.0</code>目录下有一个<strong>oozie-examples.tar.gz</strong>包，将其解压到任务目录</p>
<blockquote>
<p>tar -zxvf oozie-examples.tar.gz;</p>
</blockquote>
<h2 id="组成">组成</h2>
<p>进入examples/apps，发现有很多示例，我们以 shell 为例<br>
<img src="https://i.imgur.com/xzUdIMI.png" alt="enter image description here"></p>
<p>在 …/examples/apps/shell 目录下有两个文件</p>
<ul>
<li>job.properties：job任务的环境变量等参数</li>
<li>workflow.xml：具体执行的任务，通过xml的形式描述流程，简称 <strong>hPDL</strong></li>
</ul>
<p><strong>一个完整的任务就是由以上两部分构成</strong>。</p>
<h2 id="配置-2">配置</h2>
<p>先编辑 <strong>job.properties</strong></p>
<pre><code>&lt;!--hadoop1的配置--&gt;
&lt;!--
nameNode=hdfs://localhost:8020
jobTracker=localhost:8021
--&gt;

&lt;!--hadoop2的配置，由于我的Hadoop是HA，mycluster是Hadoop的nameservices，rm1和rm2是yarn的两个节点--&gt;
nameNode=hdfs://mycluster
jobTracker=rm1,rm2

queueName=default
examplesRoot=examples

&lt;!--true-job任务使用HDFS中share下的共享jar--&gt;
oozie.use.system.libpath=true

&lt;!--examples的HDFS目录，需要上传，稍后会讲到--&gt;
oozie.wf.application.path=${nameNode}/user/${user.name}/${examplesRoot}/apps/shell
</code></pre>
<p>Tips：</p>
<blockquote>
<p>以上配置的备注和注释是#，但是由于#在mackdown中是标题申明符，所以改用xml的形式。</p>
</blockquote>
<h2 id="坑-1">坑</h2>
<p>由于oozie的job是通过调用<strong>yarn.resourcemanager.address</strong>配置的端口进行通讯，所以我们需在$HADOOP_HOME/etc/hadoop下对<strong>yarn-site.xml</strong>追加，由于我的yarn集群是HA，所以配置如下：</p>
<pre><code>&lt;!--RM的applications manager(ASM)端口--&gt;
&lt;property&gt;
	&lt;name&gt;yarn.resourcemanager.address.rm1&lt;/name&gt;
	&lt;value&gt;itaojin101:8032&lt;/value&gt;
&lt;/property&gt;
&lt;property&gt;
	&lt;name&gt;yarn.resourcemanager.address.rm2&lt;/name&gt;
	&lt;value&gt;itaojin102:8032&lt;/value&gt;
&lt;/property&gt;
</code></pre>
<p><strong>配置完成后，记得一定要重启yarn集群使配置生效</strong></p>
<h2 id="上传hdfs">上传HDFS</h2>
<p>将examples这个文件上传到hdfs中的/user/${<a href="http://user.name">user.name</a>} 中，我采用的是root这个用户，所以是/user/root</p>
<blockquote>
<p>hdfs dfs -put examples /user/root;</p>
</blockquote>
<h2 id="第一个job">第一个job</h2>
<blockquote>
<p>oozie job -oozie <a href="http://itaojin101:11000/oozie">http://itaojin101:11000/oozie</a> -config<br>
/usr/local/share/applications/oozie-4.3.0/examples/apps/shell/job.properties<br>
-run;</p>
</blockquote>
<p><img src="https://i.imgur.com/6DasZv5.png" alt="enter image description here"></p>
<p><strong>注意</strong>： -oozie 后面跟的是oozie server的地址，-config后面跟的是执行的脚本，除了在hdfs上要有一份examples，在本地也需要一份。这个命令中的/data/installers/examples/apps/shell/job.properties 是本地路径的job.properties，不是hdfs上的。</p>
<ul>
<li>根据上面的job Id ，可以使用下列命令进行查看：</li>
</ul>
<blockquote>
<p>oozie job -oozie <a href="http://itaojin101:11000/oozie">http://itaojin101:11000/oozie</a> -info 0000004-180530210648410-oozie-root-W;</p>
</blockquote>
<p><img src="https://i.imgur.com/KXqDWpD.png" alt="enter image description here"></p>
<ul>
<li>根据上面的job Id ，可以使用下列命令进行日志
<blockquote>
<p>oozie job -oozie <a href="http://itaojin101:11000/oozie">http://itaojin101:11000/oozie</a> -log 0000004-180530210648410-oozie-root-W;</p>
</blockquote>
</li>
</ul>
<p><img src="https://i.imgur.com/GaOu8pI.png" alt="enter image description here"></p>
<ul>
<li>可以访问 <strong><a href="http://itaojin101:11000/oozie">http://itaojin101:11000/oozie</a></strong> 查看提交的job的情况：<br>
<img src="https://i.imgur.com/V5ggJ3n.png" alt="enter image description here"></li>
</ul>
<p>webUI可以嫌多信息</p>
<ol>
<li>任务的整个生命周期都能看到</li>
<li>工作流的有向无环图（DAG）</li>
<li>正常日志和错误日志</li>
</ol>
<p>等等</p>
<p><strong>参考网站</strong>：</p>
<p><a href="https://hadooptutorial.info/oozie-share-lib-does-not-exist-error/">https://hadooptutorial.info/oozie-share-lib-does-not-exist-error/</a></p>
<p><a href="http://blog.csdn.net/teddeyang/article/details/16339533">http://blog.csdn.net/teddeyang/article/details/16339533 </a></p>
<p><a href="http://oozie.apache.org/docs/4.2.0/AG_Install.html">http://oozie.apache.org/docs/4.2.0/AG_Install.html </a></p>
<p><a href="http://www.cloudera.com/content/cloudera/en/documentation/cdh4/v4-2-0/CDH4-Installation-Guide/cdh4ig_topic_17_6.html">http://www.cloudera.com/content/cloudera/en/documentation/cdh4/v4-2-0/CDH4-Installation-Guide/cdh4ig_topic_17_6.html</a></p>
<p><a href="http://stackoverflow.com/questions/11555344/sqoop-export-fail-through-oozie">http://stackoverflow.com/questions/11555344/sqoop-export-fail-through-oozie</a></p>


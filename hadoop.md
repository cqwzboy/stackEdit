---


---

<p><strong>Hadoop2.9.0 主要由HDFS，Yarn和MapReduce三部分构成</strong></p>
<h1 id="hdfs">HDFS</h1>
<h2 id="几种hdfs集群">几种HDFS集群</h2>
<h3 id="单机环境">单机环境</h3>
<p>默认情况下，Hadoop被配置为以非分布式模式运行，作为单个Java进程。这对于调试非常有用。</p>
<p>搭建教程：<a href="http://hadoop.apache.org/docs/r2.9.1/hadoop-project-dist/hadoop-common/SingleCluster.html#Standalone_Operation">http://hadoop.apache.org/docs/r2.9.1/hadoop-project-dist/hadoop-common/SingleCluster.html#Standalone_Operation</a></p>
<h3 id="伪分布式集群">伪分布式集群</h3>
<p>Hadoop也可以在一个单节点上运行，在一个伪分布模式下，每个Hadoop守护进程都在一个单独的Java进程中运行。</p>
<p>搭建教程：<a href="http://hadoop.apache.org/docs/r2.9.1/hadoop-project-dist/hadoop-common/SingleCluster.html#Standalone_Operation">http://hadoop.apache.org/docs/r2.9.1/hadoop-project-dist/hadoop-common/SingleCluster.html#Standalone_Operation</a></p>
<h3 id="分布式集群">分布式集群</h3>
<p>在分布式模式下，Hadoop的每个节点可以运行在不同的服务器上，真正的实现了可扩展的分布式集群。<br>
搭建教程：<br>
<a href="http://hadoop.apache.org/docs/r2.9.1/hadoop-project-dist/hadoop-common/ClusterSetup.html">http://hadoop.apache.org/docs/r2.9.1/hadoop-project-dist/hadoop-common/ClusterSetup.html</a><br>
<a href="https://blog.csdn.net/a237428367/article/details/50462858">https://blog.csdn.net/a237428367/article/details/50462858</a></p>
<p><strong>分布式集群又分为好几种模式：</strong></p>
<h4 id="含secondarynamenode节点">含SecondaryNameNode节点</h4>
<p>在0.x系列的Hadoop版本中，分布式集群中含有SecondaryNameNode节点。</p>
<p>有个让人疑惑的概念：Secondary NameNode，也叫辅助namenode。从命名看，好像是第二个namenode，用于备份主namenode，在主namenode失败后启动。其实不然，这是一个误区。<strong>SecondaryNameNode的重要作用是定期通过编辑日志文件合并命名空间镜像，以防止编辑日志文件过大</strong>。</p>
<h4 id="含checkpointnode和backupnode">含CheckpointNode和BackupNode</h4>
<p>在2.x系列的Hadoop产品中，包含这两个节点，且完全可以取代0.x系列遗留下来的SecondaryNameNode，当然，可以选择依然使用后者。</p>
<p>CheckpointNode和SecondaryNameNode的作用以及配置完全相同。</p>
<p>BackupNode提供了一个真正意义上的备用节点。它在内存中维护了一份从NameNode同步过来的fsimage，同时它还从namenode接收edits文件的日志流，并把它们持久化硬盘。BackupNode在内存中维护与NameNode一样的Matadata数据。</p>
<h4 id="ha-high-availability-高可用集群">HA (High Availability) 高可用集群</h4>
<p>由于NameNode的在集群中充当着大脑中枢的作用，所以一旦宕机整个集群随即瘫痪，HA机制应运而生，HA便是解决以上问题的一种机制。在2.x系列的Hadoop产品中得到支持，但是只能搭建2个NameNode。</p>
<h4 id="多个namenode的ha高可用集群">多个NameNode的HA高可用集群</h4>
<p>在3.x系列的Hadoop产品中，NameNode已经支持部署大于2个的情况。</p>
<h2 id="构成">构成</h2>
<p>HDFS大致由NameNode, SecondaryNameNode, CheckpointNode, BackupNode, DataNode, JournalNode, ZKFC等节点构成。</p>
<h3 id="namenode">NameNode</h3>
<p>HDFS集群有两类节点以管理者和工作者的工作模式运行，namenode就是其中的管理者。它管理着文件系统的命名空间，维护着文件系统树及整棵树的所有文件和目录。这些信息以两个文件的形式保存于内存或者磁盘，这两个文件是：命名空间镜像文件fsimage和编辑日志文件edit logs ，同时namenode也记录着每个文件中各个块所在的数据节点信息。</p>
<p><strong>namenode对元数据的操作过程</strong><br>
<img src="https://img-blog.csdn.net/20170503091946427?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveWFuZ2pqdWFu/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" alt="è¿™é‡Œå†™å›¾ç‰‡æè¿°"><br>
图中有两个文件：<br>
（1）fsimage:文件系统映射文件，也是元数据的镜像文件（磁盘中），存储某段时间namenode内存元数据信息。<br>
（2）edits log:操作日志文件。<br>
这种工作方式的特点：<br>
（1）namenode始终在内存中存储元数据（metedata）,使得“读操作”更加快、<br>
（2）有“写请求”时，向edits文件写入日志，成功返回后才修改内存，并向客户端返回。<br>
（3）fsimage文件为metedata的镜像，不会随时同步，与edits合并生成新的fsimage。</p>
<p>从以上特点可以知道，edits文件会在集群运行的过程中不断增多，占用更多的存储空间，虽然有合并，但是只有在namenode重启时才会进行。并且在实际工作环境很少重启namenode，<br>
这就带来了一下问题：<br>
（1）edits文件不断增大，如何存储和管理？<br>
（2）因为需要合并大量的edits文件生成fsimage，导致namenode重启时间过长。<br>
（3）一旦namenode宕机，用于恢复的fsiamge数据很旧，会造成大量数据的丢失。</p>
<h3 id="secondarynamenode">SecondaryNameNode</h3>
<p>上述问题的解决方案就是运行辅助namenode–Secondary NameNode，为主namenode内存中的文件系统元数据创建检查点，Secondary NameNode所做的不过是在文件系统中设置一个检查点来帮助NameNode更好的工作。它不是要取代掉NameNode也不是NameNode的备份，<br>
SecondaryNameNode有两个作用，一是镜像备份，二是日志与镜像的定期合并。两个过程同时进行，称为checkpoint（检查点）。<br>
镜像备份的作用：备份fsimage(fsimage是元数据发送检查点时写入文件)；<br>
日志与镜像的定期合并的作用：将Namenode中edits日志和fsimage合并,防止如果Namenode节点故障，namenode下次启动的时候，会把fsimage加载到内存中，应用edits log,edits log往往很大，导致操作往往很耗时。<strong>（这也是namenode容错的一套机制）</strong></p>
<p><strong>Secondary NameNode创建检查点过程</strong></p>
<p><img src="https://img-blog.csdn.net/20170503103823737?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveWFuZ2pqdWFu/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" alt="enter image description here"></p>
<p><strong>Secondarynamenode工作过程</strong><br>
（1）SecondaryNameNode通知NameNode准备提交edits文件，此时主节点将新的写操作数据记录到一个新的文件edits.new中。<br>
（2）SecondaryNameNode通过HTTP GET方式获取NameNode的fsimage与edits文件（在SecondaryNameNode的current同级目录下可见到 temp.check-point或者previous-checkpoint目录，这些目录中存储着从namenode拷贝来的镜像文件）。<br>
（3）SecondaryNameNode开始合并获取的上述两个文件，产生一个新的fsimage文件fsimage.ckpt。<br>
（4）SecondaryNameNode用HTTP POST方式发送fsimage.ckpt至NameNode。<br>
（5）NameNode将fsimage.ckpt与edits.new文件分别重命名为fsimage与edits，然后更新fstime，整个checkpoint过程到此结束。<br>
SecondaryNameNode备份由三个参数控制fs.checkpoint.period控制周期（以秒为单位，默认3600秒），fs.checkpoint.size控制日志文件超过多少大小时合并（以字节为单位，默认64M）， dfs.http.address表示http地址，这个参数在SecondaryNameNode为单独节点时需要设置。</p>
<p><strong>从工作过程可以看出，SecondaryNameNode的重要作用是定期通过编辑日志文件合并命名空间镜像，以防止编辑日志文件过大。SecondaryNameNode一般要在另一台机器上运行，因为它需要占用大量的CPU时间与namenode相同容量的内存才可以进行合并操作。它会保存合并后的命名空间镜像的副本，并在namenode发生故障时启用。</strong></p>
<h3 id="checkpointnode">CheckpointNode</h3>
<p>CheckpointNode和SecondaryNameNode的作用以及配置完全相同。<br>
<strong>启动命令:</strong><code>hdfs namenode -checkpoint</code></p>
<p>配置文件：<strong>core-site.xml</strong></p>
<ul>
<li>fs.checkpoint.period</li>
<li>fs.checkpoint,size</li>
<li>fs.checkpoint.dir</li>
<li>fs.checkpoint.edits.dir</li>
</ul>
<h3 id="backupnode">BackupNode</h3>
<p>提供了一个真正意义上的备用节点。<br>
BackupNode在内存中维护了一份从NameNode同步过来的fsimage，同时它还从namenode接收edits文件的日志流，并把它们持久化硬盘。<br>
BackupNode在内存中维护与NameNode一样的Matadata数据。<br>
<strong>启动命令:</strong><code>hdfs namenode -backup</code></p>
<p>配置文件：<strong>hdfs-site.xml</strong></p>
<ul>
<li>dfs.backup.address</li>
<li>dfs.backup.http.address</li>
</ul>
<h3 id="datanode">DataNode</h3>
<p>提供真实文件数据的存储服务，</p>
<ul>
<li>Block：最基本的存储单位，默认Block大小是128MB。</li>
<li>Replication：多复本。默认是三个。（hdfs-site.xml的dfs.replication属性）</li>
</ul>
<h3 id="journalnode">JournalNode</h3>
<p>两个NameNode为了数据同步，会通过一组称作JournalNodes的独立进程进行相互通信。当active状态的NameNode的命名空间有任何修改时，会告知大部分的JournalNodes进程。standby状态的NameNode有能力读取JNs中的变更信息，并且一直监控edit log的变化，把变化应用于自己的命名空间。standby可以确保在集群出错时，命名空间状态已经完全同步了。</p>
<h3 id="zkfailovercontroller-zkfc">ZKFailoverController (ZKFC)</h3>
<p>当Active的NameNode宕机后，ZKFC会使Standby状态的NameNode自动升级为Active状态，从而保证集群的正常运行。<strong>ZKFC是HDFS HA自动切换机制的核心</strong>。</p>
<h2 id="搭建hadoop集群非ha">搭建Hadoop集群(非HA)</h2>
<h3 id="准备工作">准备工作</h3>
<ol>
<li>服务器说明：</li>
</ol>
<p>itaojin101: 192.168.1.101 DataNode</p>
<p>itaojin102: 192.168.1.102 DataNode</p>
<p>itaojin103: 192.168.1.103 NamNode</p>
<p>itaojin106: 192.168.1.106 SecondaryNamNode<br>
2. 每台机器安装SSH，且实现免密登录<br>
3. 每台机器安装JDK1.8</p>
<h3 id="搭建步骤">搭建步骤</h3>
<h4 id="下载">下载</h4>
<p>下载Hadoop2.9并解压，将/bin和/sbin加入path环境变量</p>
<h4 id="配置">配置</h4>
<p>在…/etc/hadoop路径下配置文件：</p>
<p><strong>无特殊说明，以下配置在每个节点上都需要配置</strong></p>
<p><strong>code-site.xml</strong></p>
<pre><code>&lt;configuration&gt;  
&lt;!--数据存储路径--&gt;  
&lt;property&gt;  
&lt;name&gt;hadoop.tmp.dir&lt;/name&gt;  
&lt;value&gt;/hadoop/tmp&lt;/value&gt;  
&lt;description&gt;Abase for other temporary directories.&lt;/description&gt;  
&lt;/property&gt;  
&lt;!--默认的主节点--&gt;  
&lt;property&gt;  
&lt;name&gt;fs.defaultFS&lt;/name&gt;  
&lt;value&gt;hdfs://itaojin103:9000&lt;/value&gt;  
&lt;/property&gt;  
&lt;property&gt;  
&lt;name&gt;io.file.buffer.size&lt;/name&gt;  
&lt;value&gt;4096&lt;/value&gt;  
&lt;/property&gt;  
&lt;/configuration&gt; 
</code></pre>
<p><strong>hdfs-site.xml</strong></p>
<pre><code>&lt;configuration&gt;  
&lt;!--副本数--&gt;  
&lt;property&gt;  
&lt;name&gt;dfs.replication&lt;/name&gt;  
&lt;value&gt;2&lt;/value&gt;  
&lt;description&gt;nodes total count&lt;/description&gt;  
&lt;/property&gt;  
  
&lt;!--只需在NameNode上配置，SecondaryNameNode节点服务器--&gt;  
&lt;property&gt;  
&lt;name&gt;dfs.namenode.secondary.http-address&lt;/name&gt;  
&lt;value&gt;itaojin106:50090&lt;/value&gt;  
&lt;/property&gt;  
&lt;/configuration&gt;
</code></pre>
<p>新建<strong>mapred-site.xml</strong></p>
<pre><code>&lt;configuration&gt;  
&lt;property&gt;  
&lt;name&gt;mapreduce.framework.name&lt;/name&gt;  
&lt;value&gt;yarn&lt;/value&gt;  
&lt;final&gt;true&lt;/final&gt;  
&lt;/property&gt;  
&lt;property&gt;  
&lt;name&gt;mapreduce.jobtracker.http.address&lt;/name&gt;  
&lt;value&gt;itaojin103:50030&lt;/value&gt;  
&lt;/property&gt;  
&lt;property&gt;  
&lt;name&gt;mapreduce.jobhistory.address&lt;/name&gt;  
&lt;value&gt;itaojin103:10020&lt;/value&gt;  
&lt;/property&gt;  
&lt;property&gt;  
&lt;name&gt;mapreduce.jobhistory.webapp.address&lt;/name&gt;  
&lt;value&gt;itaojin103:19888&lt;/value&gt;  
&lt;/property&gt;  
&lt;property&gt;  
&lt;name&gt;mapred.job.tracker&lt;/name&gt;  
&lt;value&gt;http://itaojin103:9001&lt;/value&gt;  
&lt;/property&gt;  
&lt;/configuration&gt;
</code></pre>
<p>新建<strong>masters</strong></p>
<pre><code>&lt;!--SecondaryNameNode服务器--&gt;  
itaojin106  
</code></pre>
<p><strong>slaves</strong></p>
<pre><code>&lt;!--DataNode节点，一行一个--&gt;  
itaojin101  
itaojin102  
</code></pre>
<p><strong>yarn-site.xml</strong></p>
<pre><code>&lt;configuration&gt;  
&lt;!-- Site specific YARN configuration properties --&gt;  
&lt;property&gt;  
&lt;name&gt;yarn.resourcemanager.hostname&lt;/name&gt;  
&lt;value&gt;itaojin103&lt;/value&gt;  
&lt;/property&gt;  
&lt;property&gt;  
&lt;name&gt;yarn.nodemanager.aux-services&lt;/name&gt;  
&lt;value&gt;mapreduce_shuffle&lt;/value&gt;  
&lt;/property&gt;  
&lt;property&gt;  
&lt;name&gt;yarn.resourcemanager.address&lt;/name&gt;  
&lt;value&gt;itaojin103:8032&lt;/value&gt;  
&lt;/property&gt;  
&lt;property&gt;  
&lt;name&gt;yarn.resourcemanager.scheduler.address&lt;/name&gt;  
&lt;value&gt;itaojin103:8030&lt;/value&gt;  
&lt;/property&gt;  
&lt;property&gt;  
&lt;name&gt;yarn.resourcemanager.resource-tracker.address&lt;/name&gt;  
&lt;value&gt;itaojin103:8031&lt;/value&gt;  
&lt;/property&gt;  
&lt;property&gt;  
&lt;name&gt;yarn.resourcemanager.admin.address&lt;/name&gt;  
&lt;value&gt;itaojin103:8033&lt;/value&gt;  
&lt;/property&gt;  
&lt;property&gt;  
&lt;name&gt;yarn.resourcemanager.webapp.address&lt;/name&gt;  
&lt;value&gt;itaojin103:8088&lt;/value&gt;  
&lt;/property&gt;  
&lt;/configuration&gt;  
</code></pre>
<p><strong><a href="http://hadoop-env.sh">hadoop-env.sh</a></strong></p>
<pre><code>&lt;!--配置本机JDK路径--&gt;  
export JAVA_HOME=/usr/local/java/jdk1.8.0_152  
</code></pre>
<h4 id="启动">启动</h4>
<p>只需在NameNode节点上启动即可</p>
<pre><code>start-all.sh 启动HDFS和MapReduce  
stop-all.sh  
start-dfs.sh 只启动HDFS  
stop-dfs.sh  
</code></pre>
<h4 id="验证">验证</h4>
<p>·NameNode的IP:50070·，进入HDFS管理页面</p>
<h2 id="搭建hadoop集群ha">搭建Hadoop集群(HA)</h2>
<h3 id="环境概要">环境概要</h3>
<p><strong>HOST</strong></p>

<table>
<thead>
<tr>
<th>域名</th>
<th>IP</th>
</tr>
</thead>
<tbody>
<tr>
<td>itaojin101</td>
<td>192.168.1.101</td>
</tr>
<tr>
<td>itaojin102</td>
<td>192.168.1.102</td>
</tr>
<tr>
<td>itaojin103</td>
<td>192.168.1.103</td>
</tr>
<tr>
<td>itaojin105</td>
<td>192.168.1.105</td>
</tr>
<tr>
<td>itaojin106</td>
<td>192.168.1.106</td>
</tr>
<tr>
<td>itaojin107</td>
<td>192.168.1.107</td>
</tr>
</tbody>
</table><p><strong>节点规划</strong></p>

<table>
<thead>
<tr>
<th></th>
<th>NameNode</th>
<th>JournalNode</th>
<th>DataNode</th>
<th>Zookeeper</th>
<th>ZKFC</th>
</tr>
</thead>
<tbody>
<tr>
<td>itaojin101</td>
<td>√</td>
<td>√</td>
<td></td>
<td></td>
<td>√</td>
</tr>
<tr>
<td>itaojin102</td>
<td>√</td>
<td>√</td>
<td></td>
<td></td>
<td>√</td>
</tr>
<tr>
<td>itaojin103</td>
<td></td>
<td>√</td>
<td>√</td>
<td></td>
<td></td>
</tr>
<tr>
<td>itaojin106</td>
<td></td>
<td></td>
<td>√</td>
<td>√</td>
<td></td>
</tr>
<tr>
<td>itaojin107</td>
<td></td>
<td></td>
<td>√</td>
<td>√</td>
<td></td>
</tr>
<tr>
<td>itaojin105</td>
<td></td>
<td></td>
<td></td>
<td>√</td>
<td></td>
</tr>
</tbody>
</table><h3 id="环境准备">环境准备</h3>
<ol>
<li>6台centos7虚拟机</li>
<li>每台服务器上安装好JDK，且保证JDK的安装路径相同，方便后面copy操作。</li>
<li>实现6台服务器root用的SSH免密登录。采用root用户的原因很简单：省事儿。</li>
</ol>
<h3 id="环境搭建">环境搭建</h3>
<p><strong>一下操作先只在一台单机上操作</strong></p>
<h4 id="下载并解压">下载并解压</h4>
<p>地址：<a href="http://apache.claz.org/hadoop/common/hadoop-2.9.0/hadoop-2.9.0.tar.gz">http://apache.claz.org/hadoop/common/hadoop-2.9.0/hadoop-2.9.0.tar.gz</a><br>
目录：<strong>cd /usr/local/share/applications</strong> （自定）<br>
下载：<strong>wget <a href="http://apache.claz.org/hadoop/common/hadoop-2.9.0/hadoop-2.9.0.tar.gz">http://apache.claz.org/hadoop/common/hadoop-2.9.0/hadoop-2.9.0.tar.gz</a></strong><br>
解压：<strong>tar -zxvf  hadoop-2.9.0.tar.gz</strong></p>
<h4 id="目录介绍">目录介绍</h4>
<p><img src="https://i.imgur.com/rLQEltm.png" alt="enter image description here"></p>
<ul>
<li>bin:  可执行脚本</li>
<li>sbin:  可执行脚本</li>
<li>etc: 配置文件</li>
<li><strong>将 …/share/doc删除，这是说明文档，删除后方便后面服务器之间拷贝</strong></li>
</ul>
<h4 id="环境变量">环境变量</h4>
<p>推荐将 <strong>bin</strong> 和 <strong>sbin</strong> 目录添加到path下</p>
<h4 id="配置文件">配置文件</h4>
<p>不推荐直接使用xshell等客户端通过vim修改配置文件，首先很麻烦，再其次易出错。推荐使用EditPlus或者Nodepad++等文本编辑器连接服务器的FTP服务，然后在文本编辑器里进行修改，很方便。</p>
<h5 id="hadoop-env.sh"><a href="http://hadoop-env.sh">hadoop-env.sh</a></h5>
<p>export JAVA_HOME=/usr/local/java/jdk1.8.0_152</p>
<h5 id="core-site.xml">core-site.xml</h5>
<pre><code>&lt;configuration&gt;
	&lt;!--指定HDFS的命名空间--&gt;
	&lt;property&gt;
	  &lt;name&gt;fs.defaultFS&lt;/name&gt;
	  &lt;value&gt;hdfs://mycluster&lt;/value&gt;
	&lt;/property&gt;

	&lt;!--指定HDFS数据存储路径，默认在/tmp下，机器重启后自动删除该目录下的所有数据--&gt;
	&lt;property&gt;
		&lt;name&gt;hadoop.tmp.dir&lt;/name&gt;
		&lt;value&gt;/hadoop/tmp&lt;/value&gt;
	&lt;/property&gt;

	&lt;!--自动切换备用NameNode所依赖的Zookeeper集群--&gt;
	&lt;property&gt;
	   &lt;name&gt;ha.zookeeper.quorum&lt;/name&gt;
	   &lt;value&gt;itaojin105:2181,itaojin106:2181,itaojin107:2181&lt;/value&gt;
	 &lt;/property&gt;
&lt;/configuration&gt;
</code></pre>
<h5 id="hdfs-site.xml">hdfs-site.xml</h5>
<pre><code>&lt;configuration&gt;
	&lt;!--副本数量，默认3--&gt;
	&lt;property&gt;
		&lt;name&gt;dfs.replication&lt;/name&gt;
		&lt;value&gt;3&lt;/value&gt;
	&lt;/property&gt;

	&lt;!--块大小，默认134217728 byte--&gt;
	&lt;property&gt;
		&lt;name&gt;dfs.blocksize&lt;/name&gt;
		&lt;value&gt;134217728&lt;/value&gt;
	&lt;/property&gt;

	&lt;!--NameNode的源数据存储路径，默认路径是“file://${hadoop.tmp.dir}/dfs/name”--&gt;
	&lt;property&gt;
		&lt;name&gt;dfs.namenode.name.dir&lt;/name&gt;
		&lt;value&gt;/hadoop/tmp/dfs/name&lt;/value&gt;
	&lt;/property&gt;
	
	&lt;!--DataNode存储数据的路径，默认路径是“file://${hadoop.tmp.dir}/dfs/data”--&gt;
	&lt;property&gt;
		&lt;name&gt;dfs.datanode.data.dir&lt;/name&gt;
		&lt;value&gt;/hadoop/tmp/dfs/data&lt;/value&gt;
	&lt;/property&gt;

	&lt;!--自定义虚拟服务名称，需要与core-site.xml中的对应上--&gt;
	&lt;property&gt;
		&lt;name&gt;dfs.nameservices&lt;/name&gt;
		&lt;value&gt;mycluster&lt;/value&gt;
	&lt;/property&gt;

	&lt;!--指定虚拟服务下的NameNode名字，随便取--&gt;
	&lt;property&gt;
	  &lt;name&gt;dfs.ha.namenodes.mycluster&lt;/name&gt;
	  &lt;value&gt;nn1,nn2&lt;/value&gt;
	&lt;/property&gt;

	&lt;!--NameNode内部节点间的RPC通讯地址--&gt;
	&lt;property&gt;
	  &lt;name&gt;dfs.namenode.rpc-address.mycluster.nn1&lt;/name&gt;
	  &lt;value&gt;itaojin101:9000&lt;/value&gt;
	&lt;/property&gt;
	&lt;property&gt;
	  &lt;name&gt;dfs.namenode.rpc-address.mycluster.nn2&lt;/name&gt;
	  &lt;value&gt;itaojin102:9000&lt;/value&gt;
	&lt;/property&gt;

	&lt;!--NameNode对外开放的Http地址--&gt;
	&lt;property&gt;
	  &lt;name&gt;dfs.namenode.http-address.mycluster.nn1&lt;/name&gt;
	  &lt;value&gt;itaojin101:50070&lt;/value&gt;
	&lt;/property&gt;
	&lt;property&gt;
	  &lt;name&gt;dfs.namenode.http-address.mycluster.nn2&lt;/name&gt;
	  &lt;value&gt;itaojin102:50070&lt;/value&gt;
	&lt;/property&gt;

	&lt;!--指定JournalNode节点的数据存放目录--&gt;
	&lt;property&gt;
	  &lt;name&gt;dfs.namenode.shared.edits.dir&lt;/name&gt;
	  &lt;value&gt;qjournal://itaojin101:8485;itaojin102:8485;itaojin103:8485/mycluster&lt;/value&gt;
	&lt;/property&gt;

	&lt;!--指定JournalNode存储本地状态的文件路径--&gt;
	&lt;property&gt;
	  &lt;name&gt;dfs.journalnode.edits.dir&lt;/name&gt;
	  &lt;value&gt;/hadoop/tmp/journal/node/local/data&lt;/value&gt;
	&lt;/property&gt;

	&lt;!--当NameNode失败时，执行自动切换的主类，能自动将宕掉的NameNode撤下，将standby状态的NameNode切换成Active--&gt;
	&lt;property&gt;
	  &lt;name&gt;dfs.client.failover.proxy.provider.mycluster&lt;/name&gt;
	  &lt;value&gt;org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider&lt;/value&gt;
	&lt;/property&gt;

	&lt;!--当“脑裂”时，采用“sshfence”方式，且免密地杀死其中一个。什么是“脑裂”：就是两个NameNode都处于Active状态，此种情况Hadoop是不被允许的--&gt;
	&lt;property&gt;
      &lt;name&gt;dfs.ha.fencing.methods&lt;/name&gt;
      &lt;value&gt;sshfence&lt;/value&gt;
    &lt;/property&gt;
    &lt;property&gt;
      &lt;name&gt;dfs.ha.fencing.ssh.private-key-files&lt;/name&gt;
      &lt;value&gt;/root/.ssh/id_rsa&lt;/value&gt;
    &lt;/property&gt;

	&lt;!--使用SSH免密处理脑裂时的超时时间--&gt;
	&lt;property&gt;
      &lt;name&gt;dfs.ha.fencing.ssh.connect-timeout&lt;/name&gt;
      &lt;value&gt;30000&lt;/value&gt;
    &lt;/property&gt;

	&lt;!--当NameNode宕掉后，启用自动切换备用NameNode--&gt;
	&lt;property&gt;
	   &lt;name&gt;dfs.ha.automatic-failover.enabled&lt;/name&gt;
	   &lt;value&gt;true&lt;/value&gt;
	 &lt;/property&gt;
&lt;/configuration&gt;
</code></pre>
<h5 id="slaves">slaves</h5>
<pre><code>itaojin103
itaojin106
itaojin107
</code></pre>
<h4 id="拷贝">拷贝</h4>
<p>将hadoop整个目录远程拷贝到其他服务器上<br>
如：我在itaojin101上做的以上修改，想将文件夹拷贝到itaojin102上，在itaojin101上执行：</p>
<pre><code>scp -r /usr/local/share/applications/hadoop-2.9.0 root@itaojin102:/usr/local/share/applications/
</code></pre>
<h4 id="初始化">初始化</h4>
<h5 id="启动zookeeper">启动Zookeeper</h5>
<p>如果Zookeeper集群已经启动则忽略</p>
<pre><code>cd &lt;zookeeper_home&gt;/bin
zkServer.sh start
</code></pre>
<p>以上操作需在zookeeper集群中的每个节点上执行，运用jps验证是否启动成功。</p>
<h5 id="启动journalnode节点">启动JournalNode节点</h5>
<p>通过配置文件可以看出，在itaojin101, itaojin102, itaojin103上部署JournalNode。有两种方式启动JournalNode：</p>
<ul>
<li>
<p><a href="http://hadoop-daemons.sh">hadoop-daemons.sh</a> start journalnode  将注册的全部journalnode启动，<strong>但是遇到一个坑–没在itaojin101，itaojin102，itaojin103上启动，而是在itaojin103，itaojin106，itaojin107上，对，跟slaves的配置保持了一致，但是slaves是配置DataNode节点的，目前还不知道为什么，推荐使用第二种方式启动。</strong></p>
</li>
<li>
<p>在itaojin101，itaojin102，itaojin103上分别执行：</p>
<p><a href="http://hadoop-daemons.sh">hadoop-daemons.sh</a> start journalnode</p>
</li>
</ul>
<p>使用jps验证是否启动成功</p>
<h5 id="格式化hdfs">格式化HDFS</h5>
<ul>
<li>
<p>在两个NameNode中任意挑选一台来格式化，执行：</p>
<p>hdfs namenode -format</p>
</li>
<li>
<p>启动NameNode</p>
<p><a href="http://hadoop-daemon.sh">hadoop-daemon.sh</a> start namenode</p>
</li>
<li>
<p>在另一台NameNode上去拉取元数据（也可以使用复制）</p>
<p>hdfs namenode -bootstrapStandby</p>
</li>
</ul>
<h5 id="格式化zkfc">格式化ZKFC</h5>
<p>在任意一台NameNode上执行：</p>
<pre><code>hdfs zkfc -formatZK
</code></pre>
<p><img src="https://i.imgur.com/iMZtbVX.png" alt="enter image description here"></p>
<p>在zookeeper集群中验证结果：</p>
<pre><code>&lt;zookeeper_home&gt;/bin/zkCli.sh
ls /
</code></pre>
<p><img src="https://i.imgur.com/KTc9kbR.png" alt="enter image description here"><br>
<strong>可以看到，zookeeper中多了hadoop-ha</strong></p>
<h3 id="启动-1">启动</h3>
<pre><code>start-dfs.sh
</code></pre>
<h3 id="验证-1">验证</h3>
<ul>
<li>
<p>用浏览器访问： <code>http://itaojin101:50070</code></p>
</li>
<li>
<p>自动切换<br>
首先访问两个NameNode的50070端口，知道Active和Standby的对应关系，然后到Active的服务器上执行</p>
<p><strong>jps | grep NameNode | awk {‘print $1’} | xargs kill -9</strong><br>
杀死NameNode，此时再访问50070，发现之前的Standby变成了Active，从而验证了自动切换。<br>
<strong>在自动切换时有一个坑：将Active节点kill掉后，Standby节点不能自动切换，究其原因，原来是最小化的CentOS7没有fuser，需要安装，执行：<br>
yum install psmisc -y<br>
执行以上命令可能报错：“Could not resolve host: <a href="http://centos.ustc.edu.cn">centos.ustc.edu.cn</a>; 未知的错误”，原因是没有配置DNS：<br>
vim /etc/resolv.conf<br>
添加：<br>
nameserver 8.8.8.8</strong></p>
</li>
</ul>
<h2 id="java客户端访问hdfs">Java客户端访问HDFS</h2>
<ol>
<li>导入jar依赖</li>
</ol>
<blockquote>
  
</blockquote>
<pre><code> &lt;groupId&gt;org.apache.hadoop&lt;/groupId&gt;  
 &lt;artifactId&gt;hadoop-client&lt;/artifactId&gt;  
 &lt;version&gt;2.9.0&lt;/version&gt;  
</code></pre>

<ol start="2">
<li></li>
</ol>
<pre><code>public static void main(String[] args)  {  
    try{  
        Long start = System.currentTimeMillis();  
  Configuration configuration = new Configuration();  
  FileSystem fileSystem = FileSystem.get(new URI("hdfs://itaojin102:9000"), configuration, "root");  
  fileSystem.copyFromLocalFile(new Path("E:\\Maven\\apache-maven-3.5.3"), new Path("/"));  
  fileSystem.close();  
  System.out.println(System.currentTimeMillis()-start + " 毫秒");  
  }catch (Exception ex){  
        ex.printStackTrace();  
  }  
}
</code></pre>
<h2 id="shell脚本">Shell脚本</h2>
<p><a href="http://hadoop.apache.org/docs/r2.9.1/hadoop-project-dist/hadoop-hdfs/HDFSCommands.html">http://hadoop.apache.org/docs/r2.9.1/hadoop-project-dist/hadoop-hdfs/HDFSCommands.html</a></p>


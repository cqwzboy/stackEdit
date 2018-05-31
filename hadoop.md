---


---

<p><strong>Hadoop2.9.0 主要由HDFS，Yarn和MapReduce三部分构成</strong></p>
<h1 id="一览表">一览表</h1>

<table>
<thead>
<tr>
<th></th>
<th>NameNode</th>
<th>JournalNode</th>
<th>DataNode</th>
<th>Zookeeper</th>
<th>ZKFC</th>
<th>ResourceManager</th>
<th>NodeManager</th>
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
<td>√</td>
<td></td>
</tr>
<tr>
<td>itaojin102</td>
<td>√</td>
<td>√</td>
<td></td>
<td></td>
<td>√</td>
<td>√</td>
<td></td>
</tr>
<tr>
<td>itaojin103</td>
<td></td>
<td>√</td>
<td>√</td>
<td></td>
<td></td>
<td></td>
<td>√</td>
</tr>
<tr>
<td>itaojin106</td>
<td></td>
<td></td>
<td>√</td>
<td>√</td>
<td></td>
<td></td>
<td>√</td>
</tr>
<tr>
<td>itaojin107</td>
<td></td>
<td></td>
<td>√</td>
<td>√</td>
<td></td>
<td></td>
<td>√</td>
</tr>
<tr>
<td>itaojin105</td>
<td></td>
<td></td>
<td></td>
<td>√</td>
<td></td>
<td></td>
<td></td>
</tr>
</tbody>
</table><h1 id="hdfs">HDFS</h1>
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
<img src="https://i.imgur.com/TcJLN9W.png" alt="enter image description here"><br>
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
<p><img src="https://i.imgur.com/LumHha1.png" alt="enter image description here"></p>
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
<h1 id="yarn">Yarn</h1>
<h2 id="yarn基本服务组件">Yarn基本服务组件</h2>
<p>YARN是Hadoop 2.0中的资源管理<a href="http://www.2cto.com/os/">系统</a>，它的基本设计思想是将MRv1中的JobTracker拆分成了两个独立的服务：一个全局的资源管理器ResourceManager和每个应用程序特有的ApplicationMaster。其中ResourceManager负责整个<a href="http://www.2cto.com/os/">系统</a>的资源管理和分配，而ApplicationMaster负责单个应用程序的管理。<br>
<img src="https://i.imgur.com/GVl9Atu.png" alt="enter image description here"><br>
YARN总体上仍然是master/slave结构，在整个资源管理框架中，resourcemanager为master，nodemanager是slave。Resourcemanager负责对各个nademanger上资源进行统一管理和调度。当用户提交一个应用程序时，需要提供一个用以跟踪和管理这个程序的ApplicationMaster，它负责向ResourceManager申请资源，并要求NodeManger启动可以占用一定资源的任务。由于不同的ApplicationMaster被分布到不同的节点上，因此它们之间不会相互影响。</p>
<p>YARN的基本组成结构，YARN主要由ResourceManager、NodeManager、ApplicationMaster和Container等几个<a href="http://www.2cto.com/kf/all/zujian/">组件</a>构成。</p>
<p>ResourceManager是Master上一个独立运行的进程，负责集群统一的资源管理、调度、分配等等；NodeManager是Slave上一个独立运行的进程，负责上报节点的状态；App Master和Container是运行在Slave上的组件，Container是yarn中分配资源的一个单位，包涵内存、CPU等等资源，yarn以Container为单位分配资源。</p>
<p>Client向ResourceManager提交的每一个应用程序都必须有一个Application Master，它经过ResourceManager分配资源后，运行于某一个Slave节点的Container中，具体做事情的Task，同样也运行与某一个Slave节点的Container中。RM，NM，AM乃至普通的Container之间的通信，都是用RPC机制。</p>
<p>YARN的架构设计使其越来越像是一个云操作系统，数据处理操作系统。<br>
<img src="https://i.imgur.com/Qqu4AFY.jpg" alt="enter image description here"></p>
<h3 id="resourcemanager">Resourcemanager</h3>
<p>RM是一个全局的资源管理器，集群只有一个，负责整个系统的资源管理和分配，包括处理客户端请求、启动/监控APP master、监控nodemanager、资源的分配与调度。它主要由两个<a href="http://www.2cto.com/kf/all/zujian/">组件</a>构成：调度器（Scheduler）和应用程序管理器（Applications Manager，ASM）。</p>
<p>（1） <strong>调度器</strong></p>
<p>调度器根据容量、队列等限制条件（如每个队列分配一定的资源，最多执行一定数量的作业等），将系统中的资源分配给各个正在运行的应用程序。需要注意的是，该调度器是一个“纯调度器”，它不再从事任何与具体应用程序相关的工作，比如不负责监控或者跟踪应用的执行状态等，也不负责重新启动因应用执行失败或者硬件故障而产生的失败任务，这些均交由应用程序相关的ApplicationMaster完成。调度器仅根据各个应用程序的资源需求进行资源分配，而资源分配单位用一个抽象概念“资源容器”（Resource Container，简称Container）表示，Container是一个动态资源分配单位，它将内存、CPU、磁盘、网络等资源封装在一起，从而限定每个任务使用的资源量。此外，该调度器是一个可插拔的组件，用户可根据自己的需要设计新的调度器，YARN提供了多种直接可用的调度器，比如Fair Scheduler和Capacity Scheduler等。</p>
<p>（2） <strong>应用程序管理器</strong></p>
<p>应用程序管理器负责管理整个系统中所有应用程序，包括应用程序提交、与调度器协商资源以启动ApplicationMaster、监控ApplicationMaster运行状态并在失败时重新启动它等。</p>
<h3 id="applicationmaster（am）">ApplicationMaster（AM）</h3>
<p>管理YARN内运行的应用程序的每个实例。</p>
<p>功能：数据切分</p>
<p>为应用程序申请资源并进一步分配给内部任务。</p>
<p>任务监控与容错</p>
<p>负责协调来自resourcemanager的资源，并通过nodemanager监视容易的执行和资源使用情况。</p>
<h3 id="nodemanager（nm）">NodeManager（NM）</h3>
<p>Nodemanager整个集群有多个，负责每个节点上的资源和使用。</p>
<p>功能：单个节点上的资源管理和任务。</p>
<p>处理来自于resourcemanager的命令。</p>
<p>处理来自域app master的命令。</p>
<p>Nodemanager管理着抽象容器，这些抽象容器代表着一些特定程序使用针对每个节点的资源。</p>
<p>Nodemanager定时地向RM汇报本节点上的资源使用情况和各个Container的运行状态（cpu和内存等资源）</p>
<h3 id="container">Container</h3>
<p>Container是YARN中的资源抽象，它封装了某个节点上的多维度资源，如内存、CPU、磁盘、网络等，当AM向RM申请资源时，RM为AM返回的资源便是用Container表示的。YARN会为每个任务分配一个Container，且该任务只能使用该Container中描述的资源。需要注意的是，Container不同于MRv1中的slot，它是一个动态资源划分单位，是根据应用程序的需求动态生成的。目前为止，YARN仅支持CPU和内存两种资源，且使用了轻量级资源隔离机制Cgroups进行资源隔离。</p>
<p>功能：对task环境的抽象</p>
<p>描述一系列信息</p>
<p>任务运行资源的集合（cpu、内存、io等）</p>
<p>任务运行环境</p>
<h2 id="yarn的资源管理">YARN的资源管理</h2>
<ol>
<li>
<p>资源调度和隔离是yarn作为一个资源管理系统，最重要且最基础的两个功能。资源调度由resourcemanager完成，而资源隔离由各个nodemanager实现。</p>
</li>
<li>
<p>Resourcemanager将某个nodemanager上资源分配给任务（这就是所谓的“资源调度”）后，nodemanager需按照要求为任务提供相应的资源，甚至保证这些资源应具有独占性，为任务运行提供基础和保证，这就是所谓的资源隔离。</p>
</li>
<li>
<p>当谈及到资源时，我们通常指内存、cpu、io三种资源。Hadoop yarn目前为止仅支持cpu和内存两种资源管理和调度。</p>
</li>
<li>
<p>内存资源多少决定任务的生死，如果内存不够，任务可能运行失败；相比之下，cpu资源则不同，它只会决定任务的快慢，不会对任务的生死产生影响。</p>
</li>
</ol>
<h3 id="yarn的内存管理">Yarn的内存管理</h3>
<p>yarn允许用户配置每个节点上可用的物理内存资源，注意，这里是“可用的”，因为一个节点上内存会被若干个服务贡享，比如一部分给了yarn，一部分给了hdfs，一部分给了hbase等，yarn配置的只是自己可用的，配置参数如下：</p>
<pre><code>yarn.nodemanager.resource.memory-mb
</code></pre>
<p>表示该节点上yarn可以使用的物理内存总量，默认是8192m，注意，如果你的节点内存资源不够8g，则需要调减这个值，yarn不会智能的探测节点物理内存总量。</p>
<pre><code>yarn.nodemanager.vmem-pmem-ratio
</code></pre>
<p>任务使用1m物理内存最多可以使用虚拟内存量，默认是2.1</p>
<pre><code>yarn.nodemanager.pmem-check-enabled
</code></pre>
<p>是否启用一个线程检查每个任务证使用的物理内存量，如果任务超出了分配值，则直接将其kill，默认是true。</p>
<pre><code>yarn.nodemanager.vmem-check-enabled
</code></pre>
<p>是否启用一个线程检查每个任务证使用的虚拟内存量，如果任务超出了分配值，则直接将其kill，默认是true。</p>
<pre><code>yarn.scheduler.minimum-allocation-mb
</code></pre>
<p>单个任务可以使用最小物理内存量，默认1024m，如果一个任务申请物理内存量少于该值，则该对应值改为这个数。</p>
<pre><code>yarn.scheduler.maximum-allocation-mb
</code></pre>
<p>单个任务可以申请的最多的内存量，默认8192m</p>
<h3 id="yarn-cpu管理">Yarn cpu管理</h3>
<p>目前cpu被划分为虚拟cpu，这里的虚拟cpu是yarn自己引入的概念，初衷是考虑到不同节点cpu性能可能不同，每个cpu具有计算能力也是不一样的，比如，某个物理cpu计算能力可能是另外一个物理cpu的2倍，这时候，你可以通过为第一个物理cpu多配置几个虚拟cpu弥补这种差异。用户提交作业时，可以指定每个任务需要的虚拟cpu个数。在yarn中，cpu相关配置参数如下：</p>
<pre><code>yarn.nodemanager.resource.cpu-vcores
</code></pre>
<p>表示该节点上yarn可使用的虚拟cpu个数，默认是8个，注意，目前推荐将该值为与物理cpu核数相同。如果你的节点cpu合数不够8个，则需要调减小这个值，而yarn不会智能的探测节点物理cpu总数。</p>
<pre><code>yarn.scheduler.minimum-allocation-vcores
</code></pre>
<p>单个任务可申请最小cpu个数，默认1，如果一个任务申请的cpu个数少于该数，则该对应值被修改为这个数</p>
<pre><code>yarn.scheduler.maximum-allocation-vcores
</code></pre>
<p>单个任务可以申请最多虚拟cpu个数，默认是32.</p>
<h2 id="搭建集群-ha">搭建集群 (HA)</h2>
<h3 id="节点规划">节点规划</h3>

<table>
<thead>
<tr>
<th></th>
<th>ResourceManager</th>
<th>NodeManager</th>
</tr>
</thead>
<tbody>
<tr>
<td>itaojin101</td>
<td>√</td>
<td></td>
</tr>
<tr>
<td>itaojin102</td>
<td>√</td>
<td></td>
</tr>
<tr>
<td>itaojin103</td>
<td></td>
<td>√</td>
</tr>
<tr>
<td>itaojin106</td>
<td></td>
<td>√</td>
</tr>
<tr>
<td>itaojin107</td>
<td></td>
<td>√</td>
</tr>
<tr>
<td>itaojin105</td>
<td></td>
<td></td>
</tr>
</tbody>
</table><h3 id="编辑yarn-env.sh"><a href="http://xn--yarn-env-8c2vp91j.sh">编辑yarn-env.sh</a></h3>
<pre><code>export JAVA_HOME=/usr/local/java/jdk1.8.0_152
</code></pre>
<h3 id="编辑yarn-site.xml">编辑yarn-site.xml</h3>
<pre><code>&lt;configuration&gt;
	&lt;!--是否启用yarn的HA--&gt;
	&lt;property&gt;
	  &lt;name&gt;yarn.resourcemanager.ha.enabled&lt;/name&gt;
	  &lt;value&gt;true&lt;/value&gt;
	&lt;/property&gt;
	&lt;!--yarn的HA虚拟服务名--&gt;
	&lt;property&gt;
	  &lt;name&gt;yarn.resourcemanager.cluster-id&lt;/name&gt;
	  &lt;value&gt;cluster1&lt;/value&gt;
	&lt;/property&gt;
	&lt;!--yarn的HA虚拟服务名下的具体服务名--&gt;
	&lt;property&gt;
	  &lt;name&gt;yarn.resourcemanager.ha.rm-ids&lt;/name&gt;
	  &lt;value&gt;rm1,rm2&lt;/value&gt;
	&lt;/property&gt;
	&lt;!--指定ResourceManager的主机名--&gt;
	&lt;property&gt;
	  &lt;name&gt;yarn.resourcemanager.hostname.rm1&lt;/name&gt;
	  &lt;value&gt;itaojin101&lt;/value&gt;
	&lt;/property&gt;
	&lt;property&gt;
	  &lt;name&gt;yarn.resourcemanager.hostname.rm2&lt;/name&gt;
	  &lt;value&gt;itaojin102&lt;/value&gt;
	&lt;/property&gt;
	&lt;!--RM的applications manager(ASM)端口--&gt;
	&lt;property&gt;
	  &lt;name&gt;yarn.resourcemanager.address.rm1&lt;/name&gt;
	  &lt;value&gt;itaojin101:8032&lt;/value&gt;
	&lt;/property&gt;
	&lt;property&gt;
	  &lt;name&gt;yarn.resourcemanager.address.rm2&lt;/name&gt;
	  &lt;value&gt;itaojin102:8032&lt;/value&gt;
	&lt;/property&gt;
	&lt;!--指定shuffle--&gt;
	&lt;property&gt;
		&lt;name&gt;yarn.nodemanager.aux-services&lt;/name&gt;
		&lt;value&gt;mapreduce_shuffle&lt;/value&gt;
	&lt;/property&gt;
	&lt;!--指定ResourceManager的web UI通讯地址--&gt;
	&lt;property&gt;
	  &lt;name&gt;yarn.resourcemanager.webapp.address.rm1&lt;/name&gt;
	  &lt;value&gt;itaojin101:8088&lt;/value&gt;
	&lt;/property&gt;
	&lt;property&gt;
	  &lt;name&gt;yarn.resourcemanager.webapp.address.rm2&lt;/name&gt;
	  &lt;value&gt;itaojin102:8088&lt;/value&gt;
	&lt;/property&gt;
	&lt;!--指定Zookeeper集群环境--&gt;
	&lt;property&gt;
	  &lt;name&gt;yarn.resourcemanager.zk-address&lt;/name&gt;
	  &lt;value&gt;itaojin105:2181,itaojin106:2181,itaojin107:2181&lt;/value&gt;
	&lt;/property&gt;
&lt;/configuration&gt;
</code></pre>
<p><strong>至此。Yarn相关的配置已经完成，等到MapReduce配置完毕后一并验证</strong></p>
<h1 id="mapreduce">MapReduce</h1>
<h2 id="mapreduce是什么">MapReduce是什么</h2>
<p>Hadoop MapReduce是一个软件框架，基于该框架能够容易地编写应用程序，这些应用程序能够运行在由上千个商用机器组成的大集群上，并以一种可靠的，具有容错能力的方式并行地处理上TB级别的海量数据集。这个定义里面有着这些关键词，</p>
<ul>
<li>一是软件框架</li>
<li>二是并行处理</li>
<li>三是可靠且容错</li>
<li>四是大规模集群</li>
<li>五是海量数据集</li>
</ul>
<h2 id="mapreduce做什么">MapReduce做什么</h2>
<p>MapReduce擅长处理大数据，它为什么具有这种能力呢？这可由MapReduce的设计思想发觉。MapReduce的思想就是“<strong>分而治之</strong>”。</p>
<p>（1）<strong>Mapper负责“分”</strong>，即把复杂的任务分解为若干个“简单的任务”来处理。“简单的任务”包含三层含义：</p>
<p>一是数据或计算的规模相对原任务要大大<strong>缩小</strong>；二是<strong>就近计算原则</strong>，即任务会分配到存放着所需数据的节点上进行计算；三是这些小任务<strong>可以并行计算，彼此间几乎没有依赖</strong>关系。</p>
<p>（2）<strong>Reducer</strong>负责对map阶段的<strong>结果进行汇总</strong>。至于需要多少个Reducer，用户可以根据具体问题，通过在mapred-site.xml配置文件里设置参数mapred.reduce.tasks的值，缺省值为1。</p>
<blockquote>
<p>一个比较形象的语言解释MapReduce：</p>
<p>我们要数图书馆中的所有书。你数1号书架，我数2号书架。这就是“<strong>Map</strong>”。我们人越多，数书就更快。</p>
<p>现在我们到一起，把所有人的统计数加在一起。这就是“<strong>Reduce</strong>”。</p>
</blockquote>
<h2 id="mapreduce工作机制">MapReduce工作机制</h2>
<p><img src="https://i.imgur.com/vNU5kPc.jpg" alt="enter image description here"></p>
<p>MapReduce的整个工作过程如上图所示，它包含如下4个独立的实体：</p>
<p>* 实体一：<strong>客户端</strong>，用来提交MapReduce作业。</p>
<p>* 实体二：<strong>JobTracker</strong>，用来协调作业的运行。</p>
<p>* 实体三：<strong>TaskTracker</strong>，用来处理作业划分后的任务。</p>
<p>* 实体四：<strong>HDFS</strong>，用来在其它实体间共享作业文件。</p>
<p><img src="https://i.imgur.com/fzAy2qH.png" alt="enter image description here"></p>
<h2 id="hadoop中的mapreduce框架">Hadoop中的MapReduce框架</h2>
<p>一个MapReduce作业通常会把输入的数据集切分为若干独立的数据块，由Map任务以完全并行的方式去处理它们。</p>
<p>框架会对Map的输出先进行排序，然后把结果输入给Reduce任务。通常作业的输入和输出都会被存储在文件系统中，整个框架负责任务的调度和监控，以及重新执行已经关闭的任务。</p>
<p>通常，MapReduce框架和分布式文件系统是运行在一组相同的节点上，也就是说，计算节点和存储节点通常都是在一起的。这种配置允许框架在那些已经存好数据的节点上高效地调度任务，这可以使得整个集群的网络带宽被非常高效地利用。</p>
<h3 id="mapreduce框架的组成">MapReduce框架的组成</h3>
<p><img src="https://i.imgur.com/qBh5jPC.png" alt="enter image description here"></p>
<ul>
<li>
<p>（1）JobTracker<br>
　　JobTracker负责调度构成一个作业的所有任务，这些任务分布在不同的TaskTracker上（由上图的JobTracker可以看到2 assign map 和 3 assign reduce）。你可以将其理解为公司的项目经理，项目经理接受项目需求，并划分具体的任务给下面的开发工程师。</p>
</li>
<li>
<p>（2）TaskTracker<br>
　　TaskTracker负责执行由JobTracker指派的任务，这里我们就可以将其理解为开发工程师，完成项目经理安排的开发任务即可。</p>
</li>
</ul>
<h3 id="mapreduce的输入输出">MapReduce的输入输出</h3>
<p>MapReduce框架运转在**&lt;key,value&gt;**键值对上，也就是说，框架把作业的输入看成是一组&lt;key,value&gt;键值对，同样也产生一组&lt;key,value&gt;键值对作为作业的输出，这两组键值对有可能是不同的。</p>
<p>一个MapReduce作业的输入和输出类型如下图所示：可以看出在整个流程中，会有三组&lt;key,value&gt;键值对类型的存在。<br>
<img src="https://i.imgur.com/sy5C7X9.png" alt="enter image description here"></p>
<h3 id="mapreduce的处理流程">MapReduce的处理流程</h3>
<p>这里以WordCount单词计数为例，介绍map和reduce两个阶段需要进行哪些处理。单词计数主要完成的功能是：统计一系列文本文件中每个单词出现的次数，如图所示：</p>
<p><img src="https://i.imgur.com/MACMucJ.jpg" alt="enter image description here"></p>
<p>（1）map任务处理<br>
<img src="https://i.imgur.com/jM1NdVg.png" alt="enter image description here"><br>
（2）reduce任务处理<br>
<img src="https://i.imgur.com/n0dZokQ.png" alt="enter image description here"></p>
<h2 id="第一个mapreduce程序：wordcount">第一个MapReduce程序：WordCount</h2>
<p>WordCount单词计数是最简单也是最能体现MapReduce思想的程序之一，该程序完整的代码可以在Hadoop安装包的src/examples目录下找到。</p>
<p>WordCount单词计数主要完成的功能是：<strong>统计一系列文本文件中每个单词出现的次数</strong>；</p>
<h3 id="初始化一个words.txt文件并上传hdfs">初始化一个words.txt文件并上传HDFS</h3>
<p>首先在Linux中通过Vim编辑一个简单的words.txt，其内容很简单如下所示：</p>
<pre><code>Hello Edison Chou
Hello Hadoop RPC
Hello Wncud Chou
Hello Hadoop MapReduce
Hello Dick Gu
</code></pre>
<p>通过Shell命令将其上传到一个指定目录中，这里指定为：/testdir/input</p>
<h3 id="自定义map函数">自定义Map函数</h3>
<p>在Hadoop 中， map 函数位于内置类org.apache.hadoop.mapreduce.<strong>Mapper</strong>&lt;KEYIN,VALUEIN, KEYOUT, VALUEOUT&gt;中，reduce 函数位于内置类org.apache.hadoop. mapreduce.<strong>Reducer</strong>&lt;KEYIN, VALUEIN, KEYOUT, VALUEOUT&gt;中。</p>
<p>我们要做的就是<strong>覆盖map 函数和reduce 函数</strong>，首先我们来覆盖map函数：继承Mapper类并重写map方法</p>
<pre><code>/** * @author Edison Chou
     * @version 1.0
     * @param KEYIN
     *            →k1 表示每一行的起始位置（偏移量offset）
     * @param VALUEIN
     *            →v1 表示每一行的文本内容
     * @param KEYOUT
     *            →k2 表示每一行中的每个单词
     * @param VALUEOUT
     *            →v2 表示每一行中的每个单词的出现次数，固定值为1 */
    public static class MyMapper extends Mapper&lt;LongWritable, Text, Text, LongWritable&gt; { protected void map(LongWritable key, Text value,
                Mapper&lt;LongWritable, Text, Text, LongWritable&gt;.Context context) throws java.io.IOException, InterruptedException {
            String[] spilted = value.toString().split(" "); for (String word : spilted) {
                context.write(new Text(word), new LongWritable(1L));
            }
        };
    }
</code></pre>
<p>Mapper 类，有四个泛型，分别是KEYIN、VALUEIN、KEYOUT、VALUEOUT，前面两个KEYIN、VALUEIN 指的是map 函数输入的参数key、value 的类型；后面两个KEYOUT、VALUEOUT 指的是map 函数输出的key、value 的类型；</p>
<p>从代码中可以看出，在Mapper类和Reducer类中都使用了Hadoop自带的基本数据类型，例如String对应Text，long对应LongWritable，int对应IntWritable。这是因为HDFS涉及到序列化的问题，Hadoop的基本数据类型都实现了一个Writable接口，而实现了这个接口的类型都支持序列化。</p>
<p>这里的map函数中通过空格符号来分割文本内容，并对其进行记录；</p>
<h3 id="自定义reduce函数">自定义Reduce函数</h3>
<p>现在我们来覆盖reduce函数：继承Reducer类并重写reduce方法</p>
<pre><code>/** * @author Edison Chou
     * @version 1.0
     * @param KEYIN
     *            →k2 表示每一行中的每个单词
     * @param VALUEIN
     *            →v2 表示每一行中的每个单词的出现次数，固定值为1
     * @param KEYOUT
     *            →k3 表示每一行中的每个单词
     * @param VALUEOUT
     *            →v3 表示每一行中的每个单词的出现次数之和 */
    public static class MyReducer extends Reducer&lt;Text, LongWritable, Text, LongWritable&gt; { protected void reduce(Text key,
                java.lang.Iterable&lt;LongWritable&gt; values,
                Reducer&lt;Text, LongWritable, Text, LongWritable&gt;.Context context) throws java.io.IOException, InterruptedException { long count = 0L; for (LongWritable value : values) {
                count += value.get();
            }
            context.write(key, new LongWritable(count));
        };
    }
</code></pre>
<p>Reducer 类，也有四个泛型，同理，分别指的是reduce 函数输入的key、value类型（这里输入的key、value类型通常和map的输出key、value类型保持一致）和输出的key、value 类型。</p>
<p>这里的reduce函数主要是将传入的&lt;k2,v2&gt;进行最后的合并统计，形成最后的统计结果。</p>
<h3 id="设置main函数">设置Main函数</h3>
<p>（1）设定输入目录，当然也可以作为参数传入</p>
<p>public static final String INPUT_PATH = “hdfs://hadoop-master:9000/testdir/input/words.txt”;</p>
<p>（2）设定输出目录（<strong>输出目录需要是空目录</strong>），当然也可以作为参数传入</p>
<p>public static final String OUTPUT_PATH = “hdfs://hadoop-master:9000/testdir/output/wordcount”;</p>
<p>（3）Main函数的主要代码</p>
<pre><code>public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration(); // 0.0:首先删除输出路径的已有生成文件
        FileSystem fs = FileSystem.get(new URI(INPUT_PATH), conf);
        Path outPath = new Path(OUTPUT_PATH); if (fs.exists(outPath)) {
            fs.delete(outPath, true);
        }

        Job job = new Job(conf, "WordCount");
        job.setJarByClass(MyWordCountJob.class); // 1.0:指定输入目录
        FileInputFormat.setInputPaths(job, new Path(INPUT_PATH)); // 1.1:指定对输入数据进行格式化处理的类（可以省略）
        job.setInputFormatClass(TextInputFormat.class); // 1.2:指定自定义的Mapper类
        job.setMapperClass(MyMapper.class); // 1.3:指定map输出的&lt;K,V&gt;类型（如果&lt;k3,v3&gt;的类型与&lt;k2,v2&gt;的类型一致则可以省略）
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(LongWritable.class); // 1.4:分区（可以省略）
        job.setPartitionerClass(HashPartitioner.class); // 1.5:设置要运行的Reducer的数量（可以省略）
        job.setNumReduceTasks(1); // 1.6:指定自定义的Reducer类
        job.setReducerClass(MyReducer.class); // 1.7:指定reduce输出的&lt;K,V&gt;类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(LongWritable.class); // 1.8:指定输出目录
        FileOutputFormat.setOutputPath(job, new Path(OUTPUT_PATH)); // 1.9:指定对输出数据进行格式化处理的类（可以省略）
        job.setOutputFormatClass(TextOutputFormat.class); // 2.0:提交作业
        boolean success = job.waitForCompletion(true); if (success) {
            System.out.println("Success");
            System.exit(0);
        } else {
            System.out.println("Failed");
            System.exit(1);
        }
    }
</code></pre>
<p>在Main函数中，主要做了三件事：一是指定输入、输出目录；二是指定自定义的Mapper类和Reducer类；三是提交作业；匆匆看下来，代码有点多，但有些其实是可以省略的。</p>
<p>（4）完整代码如下所示</p>
<h3 id="运行吧小demo">运行吧小DEMO</h3>
<p>（1）调试查看控制台状态信息<br>
<img src="https://i.imgur.com/2V3gdKm.jpg" alt="enter image description here"></p>
<p>（2）通过Shell命令查看统计结果<br>
<img src="https://i.imgur.com/DTuyw68.jpg" alt="enter image description here"></p>
<h2 id="使用toolrunner类改写wordcount">使用ToolRunner类改写WordCount</h2>
<p>Hadoop有个ToolRunner类，它是个好东西，简单好用。无论在《Hadoop权威指南》还是Hadoop项目源码自带的example，都推荐使用ToolRunner。</p>
<h3 id="最初的写法">最初的写法</h3>
<p>下面我们看下src/example目录下WordCount.java文件，它的代码结构是这样的：</p>
<pre><code>public class WordCount { // 略...
    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        String[] otherArgs = new GenericOptionsParser(conf, 
                                            args).getRemainingArgs(); // 略...
        Job job = new Job(conf, "word count"); // 略...
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}
</code></pre>
<p>WordCount.java中使用到了GenericOptionsParser这个类，它的作用是<strong>将命令行中参数自动设置到变量conf中</strong>。举个例子，比如我希望通过命令行设置reduce task数量，就这么写：</p>
<p>bin/hadoop jar MyJob.jar com.xxx.MyJobDriver -Dmapred.reduce.tasks=5</p>
<p>上面这样就可以了，不需要将其硬编码到java代码中，很轻松就可以将参数与代码分离开。</p>
<h3 id="加入toolrunner的写法">加入ToolRunner的写法</h3>
<p>至此，我们还没有说到ToolRunner，上面的代码我们使用了GenericOptionsParser帮我们解析命令行参数，编写ToolRunner的程序员更懒，它将 GenericOptionsParser调用隐藏到自身run方法，被自动执行了，修改后的代码变成了这样：</p>
<pre><code>public class WordCount extends Configured implements Tool {
    @Override public int run(String[] arg0) throws Exception {
        Job job = new Job(getConf(), "word count"); // 略...
        System.exit(job.waitForCompletion(true) ? 0 : 1); return 0;
    } public static void main(String[] args) throws Exception { int res = ToolRunner.run(new Configuration(), new WordCount(), args);
        System.exit(res);
    }
}
</code></pre>
<p>看看这段代码上有什么不同：</p>
<p>（1）让WordCount<strong>继承Configured并实现Tool接口</strong>。</p>
<p>（2）<strong>重写Tool接口的run方法</strong>，run方法不是static类型，这很好。</p>
<p>（3）在WordCount中我们将<strong>通过getConf()获取Configuration对象</strong>。</p>
<p>可以看出，通过简单的几步，就可以实现代码与配置隔离、上传文件到DistributeCache等功能。修改MapReduce参数不需要修改java代码、打包、部署，提高工作效率。</p>
<h3 id="重写wordcount程序">重写WordCount程序</h3>
<pre><code>public class MyJob extends Configured implements Tool { public static class MyMapper extends Mapper&lt;LongWritable, Text, Text, LongWritable&gt; { protected void map(LongWritable key, Text value,
                Mapper&lt;LongWritable, Text, Text, LongWritable&gt;.Context context) throws java.io.IOException, InterruptedException {
                       ......
            }
        };
    } public static class MyReducer extends Reducer&lt;Text, LongWritable, Text, LongWritable&gt; { protected void reduce(Text key,
                java.lang.Iterable&lt;LongWritable&gt; values,
                Reducer&lt;Text, LongWritable, Text, LongWritable&gt;.Context context) throws java.io.IOException, InterruptedException {
                       ......
        };
    } // 输入文件路径
    public static final String INPUT_PATH = "hdfs://hadoop-master:9000/testdir/input/words.txt"; // 输出文件路径
    public static final String OUTPUT_PATH = "hdfs://hadoop-master:9000/testdir/output/wordcount";

    @Override public int run(String[] args) throws Exception { // 首先删除输出路径的已有生成文件
        FileSystem fs = FileSystem.get(new URI(INPUT_PATH), getConf());
        Path outPath = new Path(OUTPUT_PATH); if (fs.exists(outPath)) {
            fs.delete(outPath, true);
        }

        Job job = new Job(getConf(), "WordCount"); // 设置输入目录
        FileInputFormat.setInputPaths(job, new Path(INPUT_PATH)); // 设置自定义Mapper
        job.setMapperClass(MyMapper.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(LongWritable.class); // 设置自定义Reducer
        job.setReducerClass(MyReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(LongWritable.class); // 设置输出目录
        FileOutputFormat.setOutputPath(job, new Path(OUTPUT_PATH));

        System.exit(job.waitForCompletion(true) ? 0 : 1); return 0;
    } public static void main(String[] args) {
        Configuration conf = new Configuration(); try { int res = ToolRunner.run(conf, new MyJob(), args);
            System.exit(res);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
</code></pre>
<h2 id="配置-1">配置</h2>
<h3 id="mapred-site.cml">mapred-site.cml</h3>
<pre><code>&lt;configuration&gt;
	&lt;!--指定mapreduce的运行框架--&gt;
	&lt;property&gt;
		&lt;name&gt;mapreduce.framework.name&lt;/name&gt;
		&lt;value&gt;yarn&lt;/value&gt;
		&lt;final&gt;true&lt;/final&gt;
	&lt;/property&gt;
&lt;/configuration&gt;
</code></pre>
<h2 id="启动-2">启动</h2>
<ol>
<li>
<p>在itaojin101上执行：</p>
<p><a href="http://start-yarn.sh">start-yarn.sh</a></p>
</li>
<li>
<p>此时三个NodeManager节点已经启动，且itaojin101上的ResourceManager节点也已经启动，需要手动到itaojin102终端启动ResourceManager：</p>
<p><a href="http://yarn-daemon.sh">yarn-daemon.sh</a> start resourcemanager</p>
</li>
</ol>
<p>至此，高可用MapReduce集群已经搭建完毕！</p>
<h2 id="验证-2">验证</h2>
<ol>
<li>运行 <code>jps</code>查看各个服务器上的相关节点，如下图所示就对了：</li>
</ol>

<table>
<thead>
<tr>
<th></th>
<th>NameNode</th>
<th>JournalNode</th>
<th>DataNode</th>
<th>Zookeeper</th>
<th>ZKFC</th>
<th>ResourceManager</th>
<th>NodeManager</th>
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
<td>√</td>
<td></td>
</tr>
<tr>
<td>itaojin102</td>
<td>√</td>
<td>√</td>
<td></td>
<td></td>
<td>√</td>
<td>√</td>
<td></td>
</tr>
<tr>
<td>itaojin103</td>
<td></td>
<td>√</td>
<td>√</td>
<td></td>
<td></td>
<td></td>
<td>√</td>
</tr>
<tr>
<td>itaojin106</td>
<td></td>
<td></td>
<td>√</td>
<td>√</td>
<td></td>
<td></td>
<td>√</td>
</tr>
<tr>
<td>itaojin107</td>
<td></td>
<td></td>
<td>√</td>
<td>√</td>
<td></td>
<td></td>
<td>√</td>
</tr>
<tr>
<td>itaojin105</td>
<td></td>
<td></td>
<td></td>
<td>√</td>
<td></td>
<td></td>
<td></td>
</tr>
</tbody>
</table><ol start="2">
<li>通过运行一个WordCount小程序来验证：</li>
</ol>
<ul>
<li>
<p>在本地/root/下新建一个WordCount文件，并编辑为：</p>
<p>hadoop is nice<br>
hadoop good<br>
hadoop is better</p>
</li>
<li>
<p>将WordCount文件上传至HDFS根目录</p>
<p>hdfs dfs -put /root/WordCount /</p>
</li>
<li>
<p>执行命令：</p>
<p>yarn jar &lt;hadoop_home&gt;/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.9.0.jar wordcount /WordCount /out<br>
<strong>运用Hadoop自带的wordcount统计器统计hdfs文件根目录下的WordCount文件，并且将结果输出到/out目录下</strong></p>
</li>
<li>
<p>查看结果<br>
<img src="https://i.imgur.com/U8jTQD5.png" alt="enter image description here"></p>
</li>
</ul>
<ol start="3">
<li>
<p>验证高可用<br>
<strong>将itaojin101上的ResourceManager进程杀死，itaojin102上的ResourceManager从Standby状态变成Active</strong><br>
通过以下命令查看节点身份：</p>
<p>yarn rmadmin -getServiceState rm1</p>
</li>
</ol>


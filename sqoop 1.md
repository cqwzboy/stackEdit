---


---

<h1 id="前言">前言</h1>
<p>笔者书写本文章的时间是2018-06-12，当前的Sqoop分为两个主流版本，1.x和2.x，1.x最新版本是<a href="http://mirror.bit.edu.cn/apache/sqoop/1.4.7/">1.4.7</a>，2.x最新版本是<a href="http://mirror.bit.edu.cn/apache/sqoop/1.99.7/">1.99.7</a>。本文章主要讲解1.4.7版本的Sqoop相关概念以及安装使用。</p>
<h1 id="概念">概念</h1>
<p>Sqoop是Hadoop生态体系中的一员，主要功能是实现<strong>关系型数据库</strong>和<strong>HDFS、Hive、HBase</strong>等组件间的数据互导，该组件在Hadoop生态中所处的位置是<strong>数据采集环节</strong>。</p>
<h2 id="sqoop1和sqoop2的对比">Sqoop1和Sqoop2的对比</h2>
<ul>
<li>Sqoop1解压即用，非常轻量级，且相比于Sqoop2的服务化架构，运行更加稳定；缺点就是复用性差，每一个job都需要从头到尾从新编写，即便是编写了服用的Job文件也显得不够灵活。</li>
<li>Sqoop2采用服务化架构，server直接跟Hadoop的Map Task交互，Sqoop客户端直接跟server交互，且将一个完整的Job拆分成多个连接和job模块，每个模块间可以相互复用；缺点是不够稳定，就我个人使用感受而言至少是的，且服务化架构不够轻量，Cloudera公司在将来的CDH 6版本中将不再支持Sqoop2.</li>
</ul>
<h2 id="sqoop1架构">Sqoop1架构</h2>
<p><img src="https://i.imgur.com/gRws8bt.jpg" alt="enter image description here"></p>
<p>这是一张网上流传较广的Sqoop1架构图，架构很简单，一个Sqoop客户端直接跟Hadoop的Map Task交互，从而为<strong>关系型数据库</strong>和<strong>HDFS/Hive/HBase</strong>之间搭起了一座交互的桥梁。</p>
<p>这里不再介绍Sqoop2的架构和原理，可以通过以下链接参阅一片写的不错的博客：<a href="https://blog.csdn.net/gamer_gyt/article/details/55225700">https://blog.csdn.net/gamer_gyt/article/details/55225700</a></p>
<h2 id="官方学习文档">官方学习文档</h2>
<p><a href="http://sqoop.apache.org/docs/1.4.7/SqoopUserGuide.html">http://sqoop.apache.org/docs/1.4.7/SqoopUserGuide.html</a></p>
<h1 id="安装">安装</h1>
<h2 id="环境说明">环境说明</h2>
<p>默认已经安装了：</p>
<ul>
<li>Hadoop集群2.x，包括HDFS和Yarn</li>
<li>Hive，版本1.x和2.x皆可</li>
<li>HBase</li>
<li>Zookeeper</li>
<li>测试用的MySQL数据库</li>
</ul>
<p>我将在<strong>itaojin106</strong>机器上安装Sqoop，当前该机器上的环境如下：</p>

<table>
<thead>
<tr>
<th></th>
<th></th>
<th></th>
<th></th>
<th></th>
<th></th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>节点</strong></td>
<td>DataNode</td>
<td>NodeManager</td>
<td>QuorumPeerMain</td>
<td>RunJar</td>
<td>HRegionServer</td>
</tr>
</tbody>
</table><h2 id="下载">下载</h2>
<p><a href="http://mirror.bit.edu.cn/apache/sqoop/1.4.7/">1.4.7下载链接</a><br>
<img src="https://i.imgur.com/G7G6HgC.png" alt="enter image description here"></p>
<p>发现有两个版本，其中一个后缀Hadoop-2.6.0，如果搭建的Hadoop是2.6.0版本，下载这个将拥有最好的兼容性，如果不是，可以下载第二个不带Hadoop后缀的版本。</p>
<blockquote>
<p>cd /usr/local/share/applications</p>
</blockquote>
<blockquote>
<p>wget <a href="http://mirror.bit.edu.cn/apache/sqoop/1.4.7/sqoop-1.4.7.bin__hadoop-2.6.0.tar.gz">http://mirror.bit.edu.cn/apache/sqoop/1.4.7/sqoop-1.4.7.bin__hadoop-2.6.0.tar.gz</a></p>
</blockquote>
<h2 id="解压">解压</h2>
<blockquote>
<p>tar -zxvf sqoop-1.4.7.bin__hadoop-2.6.0.tar.gz</p>
</blockquote>
<h2 id="设置环境变量">设置环境变量</h2>
<blockquote>
<p>vim /etc/profile</p>
</blockquote>
<blockquote>
<p><img src="https://i.imgur.com/KM1g5OS.png" alt="enter image description here"></p>
</blockquote>
<blockquote>
<p>source /etc/profile</p>
</blockquote>
<p>验证：</p>
<blockquote>
<p>echo $SQOOP_HOME</p>
</blockquote>
<p>能打印出刚才设置的路径则说明配置成功</p>
<h2 id="配置">配置</h2>
<p>复制$SQOOP_HOME/conf/sqoop-env-template.sh为sqoop-env.sh</p>
<blockquote>
<p>cd $SQOOP_HOME/conf</p>
</blockquote>
<blockquote>
<p>cp <a href="http://sqoop-env-template.sh">sqoop-env-template.sh</a> <a href="http://sqoop-env.sh">sqoop-env.sh</a></p>
</blockquote>
<h2 id="导入数据库驱动jar">导入数据库驱动jar</h2>
<p>由于我们的业务中关系型数据库以MySQL为主，所以这里以MySQL举例：</p>
<p>下载MySQL驱动jar，并放入$SQOOP_HOME/lib下</p>
<blockquote>
<p>cd $SQOOP_HOME/lib</p>
</blockquote>
<blockquote>
<p>wget <a href="http://192.168.1.105:8081/nexus/content/groups/public/mysql/mysql-connector-java/5.1.38/mysql-connector-java-5.1.38.jar">http://192.168.1.105:8081/nexus/content/groups/public/mysql/mysql-connector-java/5.1.38/mysql-connector-java-5.1.38.jar</a></p>
</blockquote>
<p>保持默认配置不用修改，最好要保证每一项配置都能找到相应的目录，即保证安装的机器上都要安装各个组件，由于itaojin106满足条件，故而选之。</p>
<h2 id="除去警告">除去警告</h2>
<blockquote>
<p>cd $SQOOP_HOME/bin</p>
</blockquote>
<blockquote>
<p>vim configure-sqoop</p>
</blockquote>
<p>注释掉如下代码，不然每当运行<strong>sqoop</strong>命令式都会有警告：</p>
<pre><code>## Moved to be a runtime check in sqoop.
#if [ ! -d "${HCAT_HOME}" ]; then
#  echo "Warning: $HCAT_HOME does not exist! HCatalog jobs will fail."
#  echo 'Please set $HCAT_HOME to the root of your HCatalog installation.'
#fi

#if [ ! -d "${ACCUMULO_HOME}" ]; then
#  echo "Warning: $ACCUMULO_HOME does not exist! Accumulo imports will fail."
#  echo 'Please set $ACCUMULO_HOME to the root of your Accumulo installation.'
#fi
</code></pre>
<h2 id="同步hive的jackson-.jar">同步Hive的jackson-*.jar</h2>
<p>在执行命令的过程中，遇到了坑，原因就是Hive中lib下的jackson-<em>.jar和Sqoop的lib下的版本不一致，需要将Hive中的jackson-</em>.jar复制到Sqoop中</p>
<p>如果Hive和Sqoop中的jackson版本不一致，会报如下的错误</p>
<pre><code>    18/06/12 11:09:33 INFO ql.Driver: Starting task [Stage-0:DDL] in serial mode
18/06/12 11:09:34 ERROR exec.DDLTask: java.lang.NoSuchMethodError: com.fasterxml.jackson.databind.ObjectMapper.readerFor(Ljava/lang/Class;)Lcom/fasterxml/jackson/databind/ObjectReader;
	at org.apache.hadoop.hive.common.StatsSetupConst$ColumnStatsAccurate.&lt;clinit&gt;(StatsSetupConst.java:165)
	at org.apache.hadoop.hive.common.StatsSetupConst.parseStatsAcc(StatsSetupConst.java:297)
	at org.apache.hadoop.hive.common.StatsSetupConst.setBasicStatsState(StatsSetupConst.java:230)
	at org.apache.hadoop.hive.common.StatsSetupConst.setBasicStatsStateForCreateTable(StatsSetupConst.java:292)
	at org.apache.hadoop.hive.ql.plan.CreateTableDesc.toTable(CreateTableDesc.java:839)
	at org.apache.hadoop.hive.ql.exec.DDLTask.createTable(DDLTask.java:4321)
	at org.apache.hadoop.hive.ql.exec.DDLTask.execute(DDLTask.java:354)
	at org.apache.hadoop.hive.ql.exec.Task.executeTask(Task.java:199)
	at org.apache.hadoop.hive.ql.exec.TaskRunner.runSequential(TaskRunner.java:100)
	at org.apache.hadoop.hive.ql.Driver.launchTask(Driver.java:2183)
	at org.apache.hadoop.hive.ql.Driver.execute(Driver.java:1839)
	at org.apache.hadoop.hive.ql.Driver.runInternal(Driver.java:1526)
	at org.apache.hadoop.hive.ql.Driver.run(Driver.java:1237)
	at org.apache.hadoop.hive.ql.Driver.run(Driver.java:1227)
	at org.apache.hadoop.hive.cli.CliDriver.processLocalCmd(CliDriver.java:233)
	at org.apache.hadoop.hive.cli.CliDriver.processCmd(CliDriver.java:184)
	at org.apache.hadoop.hive.cli.CliDriver.processLine(CliDriver.java:403)
	at org.apache.hadoop.hive.cli.CliDriver.processLine(CliDriver.java:336)
	at org.apache.hadoop.hive.cli.CliDriver.processReader(CliDriver.java:474)
	at org.apache.hadoop.hive.cli.CliDriver.processFile(CliDriver.java:490)
	at org.apache.hadoop.hive.cli.CliDriver.executeDriver(CliDriver.java:793)
	at org.apache.hadoop.hive.cli.CliDriver.run(CliDriver.java:759)
	at org.apache.hadoop.hive.cli.CliDriver.main(CliDriver.java:686)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.apache.sqoop.hive.HiveImport.executeScript(HiveImport.java:331)
	at org.apache.sqoop.hive.HiveImport.importTable(HiveImport.java:241)
	at org.apache.sqoop.tool.ImportTool.importTable(ImportTool.java:537)
	at org.apache.sqoop.tool.ImportTool.run(ImportTool.java:628)
	at org.apache.sqoop.Sqoop.run(Sqoop.java:147)
	at org.apache.hadoop.util.ToolRunner.run(ToolRunner.java:70)
	at org.apache.sqoop.Sqoop.runSqoop(Sqoop.java:183)
	at org.apache.sqoop.Sqoop.runTool(Sqoop.java:234)
	at org.apache.sqoop.Sqoop.runTool(Sqoop.java:243)
	at org.apache.sqoop.Sqoop.main(Sqoop.java:252)

FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. com.fasterxml.jackson.databind.ObjectMapper.readerFor(Ljava/lang/Class;)Lcom/fasterxml/jackson/databind/ObjectReader;
18/06/12 11:09:34 ERROR ql.Driver: FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. com.fasterxml.jackson.databind.ObjectMapper.readerFor(Ljava/lang/Class;)Lcom/fasterxml/jackson/databind/ObjectReader;
18/06/12 11:09:34 INFO ql.Driver: Completed executing command(queryId=root_20180612110925_f1db6aca-208a-4cb9-b7c6-e584079f1a5c); Time taken: 1.238 seconds
18/06/12 11:09:34 INFO conf.HiveConf: Using the default value passed in for log id: 85c91a62-b6d3-408f-a4af-b173c719000e
18/06/12 11:09:34 INFO session.SessionState: Resetting thread name to  main
18/06/12 11:09:34 INFO conf.HiveConf: Using the default value passed in for log id: 85c91a62-b6d3-408f-a4af-b173c719000e
18/06/12 11:09:34 INFO session.SessionState: Deleted directory: /tmp/hive/root/85c91a62-b6d3-408f-a4af-b173c719000e on fs with scheme hdfs
18/06/12 11:09:34 INFO session.SessionState: Deleted directory: /tmp/root/85c91a62-b6d3-408f-a4af-b173c719000e on fs with scheme file
18/06/12 11:09:34 INFO hive.metastore: Closed a connection to metastore, current connections: 0
18/06/12 11:09:34 ERROR tool.ImportTool: Import failed: java.io.IOException: Hive CliDriver exited with status=1
	at org.apache.sqoop.hive.HiveImport.executeScript(HiveImport.java:355)
	at org.apache.sqoop.hive.HiveImport.importTable(HiveImport.java:241)
	at org.apache.sqoop.tool.ImportTool.importTable(ImportTool.java:537)
	at org.apache.sqoop.tool.ImportTool.run(ImportTool.java:628)
	at org.apache.sqoop.Sqoop.run(Sqoop.java:147)
	at org.apache.hadoop.util.ToolRunner.run(ToolRunner.java:70)
	at org.apache.sqoop.Sqoop.runSqoop(Sqoop.java:183)
	at org.apache.sqoop.Sqoop.runTool(Sqoop.java:234)
	at org.apache.sqoop.Sqoop.runTool(Sqoop.java:243)
	at org.apache.sqoop.Sqoop.main(Sqoop.java:252)
</code></pre>
<p>先备份$SQOOP_HOME/lib下的jackson-*.jar</p>
<blockquote>
<p>cd $SQOOP_HOME</p>
</blockquote>
<blockquote>
<p>mkdir jackson-bak</p>
</blockquote>
<blockquote>
<p>mv jackson-*.jar jackson-bak/</p>
</blockquote>
<p>拷贝<span class="katex--inline">KaTeX parse error: Expected 'EOF', got '下' at position 14: HIVE_HOME/lib下̲的jackson-*.jar到</span>SQOOP_HOME/lib下</p>
<blockquote>
<p>cp $HIVE_HOME/lib/jackson-*.jar .</p>
</blockquote>
<h2 id="测试">测试</h2>
<blockquote>
<p>sqoop list-databases --connect jdbc:mysql://localhost:3306 --username root --password root</p>
</blockquote>
<p>运行结果如下：</p>
<p><img src="https://i.imgur.com/mzozn6M.png" alt="enter image description here"></p>
<p>至此，安装完毕！！！</p>
<h1 id="语法">语法</h1>
<p>Sqoop有许多组工具，罗列如下：</p>

<table>
<thead>
<tr>
<th>Sqoop工具</th>
</tr>
</thead>
<tbody>
<tr>
<td><a href="http://sqoop.apache.org/docs/1.4.7/SqoopUserGuide.html#_literal_sqoop_import_literal"><code>sqoop-import</code></a></td>
</tr>
<tr>
<td><a href="http://sqoop.apache.org/docs/1.4.7/SqoopUserGuide.html#_literal_sqoop_import_all_tables_literal"><code>sqoop-import-all-tables</code></a></td>
</tr>
<tr>
<td><a href="http://sqoop.apache.org/docs/1.4.7/SqoopUserGuide.html#_literal_sqoop_import_mainframe_literal"><code>sqoop-import-mainframe</code></a></td>
</tr>
<tr>
<td><a href="http://sqoop.apache.org/docs/1.4.7/SqoopUserGuide.html#_literal_sqoop_export_literal"><code>sqoop-export</code></a></td>
</tr>
<tr>
<td><a href="http://sqoop.apache.org/docs/1.4.7/SqoopUserGuide.html#validation"><code>validation</code></a></td>
</tr>
<tr>
<td><a href="http://sqoop.apache.org/docs/1.4.7/SqoopUserGuide.html#_literal_sqoop_job_literal"><code>sqoop-job</code></a></td>
</tr>
<tr>
<td><a href="http://sqoop.apache.org/docs/1.4.7/SqoopUserGuide.html#_literal_sqoop_metastore_literal"><code>sqoop-metastore</code></a></td>
</tr>
<tr>
<td><a href="http://sqoop.apache.org/docs/1.4.7/SqoopUserGuide.html#_literal_sqoop_merge_literal"><code>sqoop-merge</code></a></td>
</tr>
<tr>
<td><a href="http://sqoop.apache.org/docs/1.4.7/SqoopUserGuide.html#_literal_sqoop_codegen_literal"><code>sqoop-codegen</code></a></td>
</tr>
<tr>
<td><a href="http://sqoop.apache.org/docs/1.4.7/SqoopUserGuide.html#_literal_sqoop_create_hive_table_literal"><code>sqoop-create-hive-table</code></a></td>
</tr>
<tr>
<td><a href="http://sqoop.apache.org/docs/1.4.7/SqoopUserGuide.html#_literal_sqoop_eval_literal"><code>sqoop-eval</code></a></td>
</tr>
<tr>
<td><a href="http://sqoop.apache.org/docs/1.4.7/SqoopUserGuide.html#_literal_sqoop_list_databases_literal"><code>sqoop-list-databases</code></a></td>
</tr>
<tr>
<td><a href="http://sqoop.apache.org/docs/1.4.7/SqoopUserGuide.html#_literal_sqoop_list_tables_literal"><code>sqoop-list-tables</code></a></td>
</tr>
<tr>
<td><a href="http://sqoop.apache.org/docs/1.4.7/SqoopUserGuide.html#_literal_sqoop_help_literal"><code>sqoop-help</code></a></td>
</tr>
<tr>
<td><a href="http://sqoop.apache.org/docs/1.4.7/SqoopUserGuide.html#_literal_sqoop_version_literal"><code>sqoop-version</code></a></td>
</tr>
</tbody>
</table><h2 id="sqoop-import">sqoop-import</h2>
<p>Sqoop最重要的功能，也是业务中用得最多的情形是 <strong>将关系型数据库中的数据导入HDFS/Hive/HBase</strong>。</p>
<h3 id="连接数据库服务器">连接数据库服务器</h3>

<table>
<thead>
<tr>
<th>参数</th>
<th>描述</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>--connect &lt;jdbc-uri&gt;</code></td>
<td>指定JDBC连接地址(<code>jdbc:mysql://&lt;ip&gt;:&lt;port&gt;[/&lt;schema&gt;]</code>)</td>
</tr>
<tr>
<td><code>--connection-manager &lt;class-name&gt;</code></td>
<td>指定连接管理器主类</td>
</tr>
<tr>
<td><code>--driver &lt;class-name&gt;</code></td>
<td>指定JDBC驱动程序类</td>
</tr>
<tr>
<td><code>--hadoop-mapred-home &lt;dir&gt;</code></td>
<td>覆盖配置中的 $HADOOP_MAPRED_HOME</td>
</tr>
<tr>
<td><code>--username &lt;username&gt;</code></td>
<td>设置数据库用户名</td>
</tr>
<tr>
<td><code>--password &lt;password&gt;</code></td>
<td>设置数据库密码（明文）</td>
</tr>
<tr>
<td><code>-P</code></td>
<td>从控制台输入流读取数据库密码</td>
</tr>
<tr>
<td><code>--password-file</code></td>
<td>从一个文件中读取密码</td>
</tr>
<tr>
<td><code>--verbose</code></td>
<td>打印更加详细的信息</td>
</tr>
<tr>
<td><code>--connection-param-file &lt;filename&gt;</code></td>
<td>从一个文件中读取数据库连接信息</td>
</tr>
<tr>
<td><code>--relaxed-isolation</code></td>
<td>设置事务隔离级别</td>
</tr>
</tbody>
</table><ul>
<li>测试与数据库的连通性</li>
</ul>
<blockquote>
<p>sqoop list-databases --connect jdbc:mysql://localhost:3306 --username root --password root</p>
</blockquote>
<ul>
<li>以下命令等同于上面的命令：</li>
</ul>
<blockquote>
<p>sqoop list-databases --connect jdbc:mysql://localhost:3306 --username root -P</p>
</blockquote>
<p>只不过需要再控制台手动输入密码，这样就提高了安全性。</p>
<h3 id="有选择性的导入数据">有选择性的导入数据</h3>

<table>
<thead>
<tr>
<th>参数</th>
<th>描述</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>--append</code></td>
<td>HDFS中，往已存在的文件中追加内容</td>
</tr>
<tr>
<td><code>--as-avrodatafile</code></td>
<td>以Avro的格式导入</td>
</tr>
<tr>
<td><code>--as-sequencefile</code></td>
<td>以序列化文件的格式导入</td>
</tr>
<tr>
<td><code>--as-textfile</code></td>
<td>以文本的格式导入</td>
</tr>
<tr>
<td><code>--as-parquetfile</code></td>
<td>Imports data to Parquet Files</td>
</tr>
<tr>
<td><code>--boundary-query &lt;statement&gt;</code></td>
<td>自定义边界查询SQL，默认是以主键的最大最小值作为边界</td>
</tr>
<tr>
<td><code>--columns &lt;col,col,col…&gt;</code></td>
<td>设定需要导入的列名</td>
</tr>
<tr>
<td><code>--delete-target-dir</code></td>
<td>HDFS中，如果导入目录存在则删除</td>
</tr>
<tr>
<td><code>--direct</code></td>
<td>采用mysqldump工具替代JDBC直连</td>
</tr>
<tr>
<td><code>--fetch-size &lt;n&gt;</code></td>
<td>从数据库中读取的条目的数量</td>
</tr>
<tr>
<td><code>--inline-lob-limit &lt;n&gt;</code></td>
<td>Set the maximum size for an inline LOB</td>
</tr>
<tr>
<td><code>-m,--num-mappers &lt;n&gt;</code></td>
<td>并行运行的Map Task的数量</td>
</tr>
<tr>
<td><code>-e,--query &lt;statement&gt;</code></td>
<td>自定义查询语句</td>
</tr>
<tr>
<td><code>--split-by &lt;column-name&gt;</code></td>
<td>用于分割的列名，默认是主键，该配置不能与<code>--autoreset-to-one-mapper</code>共同使用</td>
</tr>
<tr>
<td><code>--split-limit &lt;n&gt;</code></td>
<td>每个分割大小的上限，这只适用于Integer和Date类型（date, timestamp）字段，对于date或timestamp字段，它是以秒为单位计算的</td>
</tr>
<tr>
<td><code>--autoreset-to-one-mapper</code></td>
<td>Import should use one mapper if a table has no primary key and no split-by column is provided. Cannot be used with  <code>--split-by &lt;col&gt;</code>  option.</td>
</tr>
<tr>
<td><code>--table &lt;table-name&gt;</code></td>
<td>需要被导出的MySQL中的表名</td>
</tr>
<tr>
<td><code>--target-dir &lt;dir&gt;</code></td>
<td>HDFS中的输出文件目录</td>
</tr>
<tr>
<td><code>--temporary-rootdir &lt;dir&gt;</code></td>
<td>设置导入HDFS时存放文件的临时目录，会覆盖默认的“_sqoop”目录</td>
</tr>
<tr>
<td><code>--warehouse-dir &lt;dir&gt;</code></td>
<td>HDFS parent for table destination</td>
</tr>
<tr>
<td><code>--where &lt;where clause&gt;</code></td>
<td>设置查询条件，作为数据导入的查询条件</td>
</tr>
<tr>
<td><code>-z,--compress</code></td>
<td>开启压缩功能</td>
</tr>
<tr>
<td><code>--compression-codec &lt;c&gt;</code></td>
<td>再开启压缩功能前提下，设置压缩文件的格式，默认“gzip”</td>
</tr>
<tr>
<td><code>--null-string &lt;null-string&gt;</code></td>
<td>The string to be written for a null value for string columns</td>
</tr>
<tr>
<td><code>--null-non-string &lt;null-string&gt;</code></td>
<td>The string to be written for a null value for non-string columns</td>
</tr>
</tbody>
</table><ul>
<li>一个导入的例子：</li>
</ul>
<blockquote>
<p>sqoop import --connect jdbc:mysql://localhost:3306/itaojin --username root \<br>
–password root --table t --delete-target-dir --target-dir /user/sqoop1 -m 2 \<br>
–direct  --columns “name,age,address” --where “age &gt; 24”</p>
</blockquote>
<h3 id="灵活的查询导入">灵活的查询导入</h3>

<table>
<thead>
<tr>
<th>参数</th>
<th>描述</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>--query</code></td>
<td>自定义查询语句，灵活易用</td>
</tr>
</tbody>
</table><ul>
<li>
<p>举例</p>
<blockquote>
<p>sqoop import --connect jdbc:mysql://localhost:3306/itaojin \<br>
–username root --password root  \<br>
–query ‘select * from t where $CONDITIONS’  \ 	<br>
–split-by id --target-dir /user/sqoop1/temp_dir</p>
</blockquote>
</li>
<li>
<p>使用“–query”导入数据时必须遵循两个条件：<br>
1. 必须指定 <code>--target-dir</code><br>
2. 必须指定 <code>--split-by</code> 或 <code>-m 1</code></p>
</li>
<li>
<p>注意：<code>--query</code>后如果是双引号(" ")，则where条件后跟随 <code>\$CONDITIONS</code>；如果是单引号(’ ')，则where后面跟随 <code>$CONDITIONS</code></p>
</li>
</ul>


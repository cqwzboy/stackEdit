---


---

<h1 id="hive是什么">Hive是什么</h1>
<p>Hive是一个数据仓库基础工具在Hadoop中用来处理结构化数据。它架构在Hadoop之上，总归为大数据，并使得查询和分析方便。</p>
<p>最初，Hive是由Facebook开发，后来由Apache软件基金会开发，并作为进一步将它作为名义下Apache Hive为一个开源项目。它用在好多不同的公司。例如，亚马逊使用它在 Amazon Elastic MapReduce。</p>
<h1 id="hive不是">Hive不是</h1>
<ul>
<li>一个关系数据库</li>
<li>一个设计用于联机事务处理（OLTP）</li>
<li>实时查询和行级更新的语言</li>
</ul>
<h1 id="hiver特点">Hiver特点</h1>
<ul>
<li>它存储架构在一个数据库中并处理数据到HDFS。</li>
<li>它是专为OLAP设计。</li>
<li>它提供SQL类型语言查询叫HiveQL或HQL。</li>
<li>它是熟知，快速，可扩展和可扩展的。</li>
</ul>
<h1 id="hive架构">Hive架构</h1>
<p>下面的组件图描绘了Hive的结构：<br>
<img src="https://www.yiibai.com/uploads/allimg/141228/1-14122R10152108.jpg" alt="enter image description here"><br>
该组件图包含不同的单元。下表描述每个单元：</p>

<table>
<thead>
<tr>
<th>单元名称</th>
<th>操作</th>
</tr>
</thead>
<tbody>
<tr>
<td>用户接口/界面</td>
<td>Hive是一个数据仓库基础工具软件，可以创建用户和HDFS之间互动。用户界面，Hive支持是Hive的Web UI，Hive命令行，HiveHD洞察（在Windows服务器）。</td>
</tr>
<tr>
<td>元存储</td>
<td>Hive选择各自的数据库服务器，用以储存表，数据库，列模式或元数据表，它们的数据类型和HDFS映射。</td>
</tr>
<tr>
<td>HiveQL处理引擎</td>
<td>HiveQL类似于SQL的查询上Metastore模式信息。这是传统的方式进行MapReduce程序的替代品之一。相反，使用Java编写的MapReduce程序，可以编写为MapReduce工作，并处理它的查询。</td>
</tr>
<tr>
<td>执行引擎</td>
<td>HiveQL处理引擎和MapReduce的结合部分是由Hive执行引擎。执行引擎处理查询并产生结果和MapReduce的结果一样。它采用MapReduce方法。</td>
</tr>
<tr>
<td>HDFS 或 HBASE</td>
<td>Hadoop的分布式文件系统或者HBASE数据存储技术是用于将数据存储到文件系统。</td>
</tr>
</tbody>
</table><h1 id="hive工作原理">Hive工作原理</h1>
<p>下图描述了Hive 和Hadoop之间的工作流程。</p>
<p><img src="https://www.yiibai.com/uploads/allimg/141228/1-14122R10220b9.jpg" alt="enter image description here"></p>
<p>下表定义Hive和Hadoop框架的交互方式：</p>

<table>
<thead>
<tr>
<th>Step No</th>
<th>操作</th>
</tr>
</thead>
<tbody>
<tr>
<td>1</td>
<td><strong>Execute QueryHive</strong>接口，如命令行或Web UI发送查询驱动程序（任何数据库驱动程序，如JDBC，ODBC等）来执行。</td>
</tr>
<tr>
<td>2</td>
<td><strong>Get Plan</strong>在驱动程序帮助下查询编译器，分析查询检查语法和查询计划或查询的要求。</td>
</tr>
<tr>
<td>3</td>
<td><strong>Get Metadata</strong>编译器发送元数据请求到Metastore（任何数据库）。</td>
</tr>
<tr>
<td>4</td>
<td><strong>Send Metadata</strong>Metastore发送元数据，以编译器的响应。</td>
</tr>
<tr>
<td>5</td>
<td><strong>Send Plan</strong>编译器检查要求，并重新发送计划给驱动程序。到此为止，查询解析和编译完成。</td>
</tr>
<tr>
<td>6</td>
<td><strong>Execute Plan</strong>驱动程序发送的执行计划到执行引擎。</td>
</tr>
<tr>
<td>7</td>
<td><strong>Execute Job</strong>在内部，执行作业的过程是一个MapReduce工作。执行引擎发送作业给JobTracker，在名称节点并把它分配作业到TaskTracker，这是在数据节点。在这里，查询执行MapReduce工作。</td>
</tr>
<tr>
<td>8</td>
<td><strong>Metadata Ops</strong>与此同时，在执行时，执行引擎可以通过Metastore执行元数据操作。</td>
</tr>
<tr>
<td>9</td>
<td><strong>Fetch Result</strong>执行引擎接收来自数据节点的结果。</td>
</tr>
<tr>
<td>10</td>
<td><strong>Send Results</strong>执行引擎发送这些结果值给驱动程序。</td>
</tr>
<tr>
<td>11</td>
<td><strong>Send Results</strong>驱动程序将结果发送给Hive接口。</td>
</tr>
</tbody>
</table><h1 id="安装">安装</h1>
<h2 id="下载并解压">下载并解压</h2>
<p><strong>在这一步，我踩了不少坑，现总结一下：</strong></p>
<ul>
<li>登录官网，进入下载页面：<br>
<img src="https://i.imgur.com/6tJcQtH.png" alt="enter image description here"></li>
<li>点击官网推荐下载链接</li>
<li>选择稳定版本进行下载，根据需要选择1.x和2.x，这里我选择了2.3.3版本<br>
<img src="https://i.imgur.com/7rMsNmD.png" alt="enter image description here"></li>
<li><code>cd /usr/local/share/applications/</code></li>
<li><code>wget http://apache.cs.utah.edu/hive/stable-2/apache-hive-2.3.3-bin.tar.gz</code></li>
<li>解压：`tar -zxvf apache-hive-2.3.3-bin.tar.gz</li>
</ul>
<h2 id="配置环境变量">配置环境变量</h2>
<p>将Hive根目录下的/bin目录加入Path</p>
<pre><code>vim /etc/profile
</code></pre>
<p><img src="https://i.imgur.com/O36nZ8q.png" alt="enter image description here"></p>
<pre><code>source /etc/profile
</code></pre>
<h2 id="将mysql-connector-java.jar复制到hive的lib目录下">将mysql-connector-java.jar复制到Hive的lib目录下</h2>
<p>由于我们要采用mysql存储schema数据的方式，所以需要添加mysql链接驱动jar</p>
<p>下载jar有三种方法：</p>
<ul>
<li>到官方直接下载，但是貌似不能选择想要的版本</li>
<li>创建一个maven项目，在pom文件中导入想要版本的jar，再到repository中将jar上传至linux服务器</li>
<li>由于我在itaojin105上搭建了一套nuxus环境，所以完成第二步后，直接到nuxus拷贝链接，再到linux服务器上wget即可。<br>
<a href="http://itaojin105:8081/nexus/content/groups/public/mysql/mysql-connector-java/">Mysql各版本驱动</a></li>
</ul>
<h2 id="配置hive-site.xml">配置hive-site.xml</h2>
<ul>
<li>
<p>进入&lt;hive_home&gt;/conf/目录下，拷贝一份模板</p>
<p>cp hive-default.xml.template hiv-site.xml</p>
</li>
<li>
<p>编辑hive-site.xml<br>
一共自改三处：</p>
<ol>
<li>搜索并修改Schema数据存放的数据库信息<br>
<img src="https://i.imgur.com/pA83yeH.png" alt="enter image description here"></li>
<li>搜索并修改掉<strong>所有</strong><code>${system</code>开头的路径，改成具体路径<br>
<img src="https://i.imgur.com/yULeisl.png" alt="enter image description here"></li>
<li>搜索并修改 <strong>hive.server2.enable.doAs</strong><br>
<img src="https://i.imgur.com/RQ0ZDPS.png" alt="enter image description here"></li>
</ol>
</li>
</ul>
<h2 id="配置mysql">配置Mysql</h2>
<ul>
<li>
<p>如果没有安装Mysql，则先安装</p>
</li>
<li>
<p>新增hive用户和hive数据库</p>
<p>create user ‘hive’ identified by ‘hive’;<br>
create database hive;</p>
</li>
<li>
<p>授权</p>
<p>grant all privileges on hive.* to ‘hive’@’%’ identified by ‘hive’;</p>
</li>
</ul>
<h2 id="初始化schema数据">初始化Schema数据</h2>
<p>我们知道，我们将hive的schema数据存放在mysql，将真实数据存放在hdfs。<br>
那什么是schema数据呢？简言之，就是<strong>表格信息</strong>。</p>
<pre><code>schematool -dbType mysql -initSchema
</code></pre>
<p>看到成功提示后，访问mysql的hive数据库，能看到新建了跟多表格，这些都是存放schema数据的表格。</p>
<h2 id="hive">hive</h2>
<p>在命令行直接输入<strong>hive</strong>即可进入操作界面，我们可以像操作Mysql一样去操作Hiive，比如我们执行：</p>
<pre><code>show databases;
create database test;
use test;
create table t (id int, name string, age int);
insert into t values (1,'zhangsan', 12);
select * from t;
</code></pre>
<ul>
<li>
<p>我们再进入存放schema的Mysql数据库，查看我们新建的数据库和表格：</p>
<p><code>select * from DBS;</code><br>
<img src="https://i.imgur.com/tzWHjkx.png" alt="enter image description here"></p>
<p>select * from TBLS;<br>
<img src="https://i.imgur.com/cAmwhYP.png" alt="enter image description here"></p>
</li>
<li>
<p>查看数据存放在HDFS中的位置</p>
<p>hdfs dfs -lsr /<br>
<img src="https://i.imgur.com/BTfMDmy.png" alt="enter image description here"></p>
</li>
</ul>
<p><strong>hive</strong>命令行已经够方便了，但是它有一个致命缺点：<strong>只能在Hive的安装机器上执行，在其他客户机上不能</strong>。</p>
<p>为了解决以上问题，Hive引入了<strong>hiveserver2</strong>的服务概念，即在服务器上启动一个server进程，端口号默认是<strong>10000</strong>，客户端可以通过连接<strong>hiveserver2</strong>来操作Hive。</p>
<h2 id="hiveserver2">hiveserver2</h2>
<pre><code>hive --service hiveserver2 &amp;
或者
hiveserver2 $
</code></pre>
<p>后台启动：</p>
<blockquote>
<p>nohup hive --service hiveserver2 &amp;</p>
</blockquote>
<p>即可后台启动<strong>hiveserver2</strong>进程<br>
jps可以看到有一个<strong>RunJar</strong>进程，这就是hiveserver2</p>
<h2 id="beeline">beeline</h2>
<p>这是一个连接<strong>hiveserver2</strong>服务的客户端</p>
<pre><code>beeline
!connect jdbc:hive2://itaojin101:10000/test;
</code></pre>
<p>即可进入Hive的命令行界面，功能跟<strong>hive命令行</strong>一模一样</p>
<h1 id="java客户端访问">Java客户端访问</h1>
<ul>
<li>
<p>导入Jar依赖<br>
<code>&lt;dependency&gt; &lt;groupId&gt;org.apache.hive&lt;/groupId&gt; &lt;artifactId&gt;hive-jdbc&lt;/artifactId&gt; &lt;version&gt;2.3.3&lt;/version&gt; &lt;/dependency&gt;</code></p>
<p><strong>注意：这里有个坑，hive-jdbc的版本必须和服务器的Hive版本对上，不然会报错，切记！</strong></p>
</li>
<li>
<p>java代码</p>
<p>public static void main(String[] args){<br>
Connection connection = null;<br>
PreparedStatement pstat = null;<br>
ResultSet res = null;</p>
<p>try{<br>
Class.forName(“org.apache.hive.jdbc.HiveDriver”);<br>
connection = DriverManager.getConnection(“jdbc:hive2://itaojin101:10000/test”);<br>
pstat = connection.prepareStatement(“select * from t”);<br>
res = pstat.executeQuery();<br>
if(res != null){<br>
while (res.next()){<br>
System.out.println(res.getInt(1)+" - “+res.getString(2)+” - "+res.getInt(3));<br>
}<br>
}<br>
}catch (Exception e){<br>
e.printStackTrace();<br>
}finally {<br>
if(connection != null){<br>
try {<br>
connection.close();<br>
} catch (SQLException e) {<br>
e.printStackTrace();<br>
}<br>
}<br>
if(pstat != null){<br>
try {<br>
pstat.close();<br>
} catch (SQLException e) {<br>
e.printStackTrace();<br>
}<br>
}<br>
if(res != null){<br>
try {<br>
res.close();<br>
} catch (SQLException e) {<br>
e.printStackTrace();<br>
}<br>
}<br>
}</p>
<p>}</p>
</li>
</ul>
<h2 id="坑（windows客户端连接hive）">坑（windows客户端连接Hive）</h2>
<ul>
<li>针对问题：</li>
</ul>
<blockquote>
<p>hadoop2.9.0，运用客户端连接hiveserver2报错 HADOOP_HOME and hadoop.home.dir are unset</p>
</blockquote>
<ul>
<li>
<p>文件下载地址<br>
<a href="https://pan.baidu.com/s/1H5nba7mHTuv90rGUrJjotA">https://pan.baidu.com/s/1H5nba7mHTuv90rGUrJjotA</a></p>
</li>
<li>
<p>解决办法：</p>
</li>
</ul>
<ol>
<li>将文件解压到hadoop的bin目录下</li>
<li>将hadoop.dll复制到C:\Window\System32下</li>
<li>添加环境变量HADOOP_HOME，指向hadoop目录</li>
<li>将%HADOOP_HOME%\bin加入到path里面</li>
<li>重启IDE，或者在代码首行添加：</li>
</ol>
<blockquote>
<p>System.setProperty(“hadoop.home.dir”, “E:\hadoop-2.9.0”);</p>
</blockquote>
<h1 id="hive中的表">Hive中的表</h1>
<h2 id="managed-table">managed table</h2>
<p><strong>托管表</strong> - 删除表时（Mysql上），数据也一并删除（HDFS上）</p>
<h2 id="external-table">external table</h2>
<p><strong>外部表</strong> - 删除表时（Mysql上），数据不被删除（HDFS上）</p>
<h2 id="分区表">分区表</h2>
<p>即表支持分区存储</p>
<h2 id="非分区表">非分区表</h2>
<h2 id="thinking-time">Thinking Time</h2>
<p>Tip:</p>
<blockquote>
<p>本段内容属于超前内容，可以先学习到<strong>分区表</strong>章节后再回来阅读</p>
</blockquote>
<p>从上面的分类可以看出：Hive中的表大体分为两大类：<strong>托管表和外部表</strong>，<strong>分区表和非分区表</strong>。那么，如果一张表既既是外部表又是分区表，那会不会有更多的特性呢？</p>
<p>先将两组特性的组合情况编号：</p>

<table>
<thead>
<tr>
<th></th>
<th>删分区删数据</th>
<th>删分区不删数据</th>
<th>不分区</th>
</tr>
</thead>
<tbody>
<tr>
<td>删表删数据</td>
<td>1</td>
<td>2</td>
<td>5</td>
</tr>
<tr>
<td>删表不删数据</td>
<td>3</td>
<td>4</td>
<td>6</td>
</tr>
</tbody>
</table><p>结论：</p>

<table>
<thead>
<tr>
<th></th>
<th>分区表</th>
<th>非分区表</th>
</tr>
</thead>
<tbody>
<tr>
<td>托管表</td>
<td>1</td>
<td>5</td>
</tr>
<tr>
<td>外部表</td>
<td>4</td>
<td>6</td>
</tr>
</tbody>
</table><p><strong>为了加强知识吸收，强烈建议通过命令行亲自验证以上结论</strong></p>
<h1 id="hive命令">Hive命令</h1>
<h2 id="创建表">创建表</h2>
<blockquote>
<p>CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name<br>
[(col_name data_type [COMMENT col_comment], …)]<br>
[COMMENT table_comment]<br>
[PARTITIONED BY (col_name data_type [COMMENT col_comment], …)]<br>
[CLUSTERED BY (col_name, col_name, …)<br>
[SORTED BY (col_name [ASC|DESC], …)] INTO num_buckets BUCKETS]<br>
[ROW FORMAT row_format]<br>
[STORED AS file_format]<br>
[LOCATION hdfs_path]</p>
</blockquote>
<ul>
<li>
<p><strong>PARTITIONED BY</strong>给表格指定分区，分区的要素可以有多个</p>
</li>
<li>
<p><strong>CREATE TABLE</strong> 创建一个指定名字的表。如果相同名字的表已经存在，则抛出异常；用户可以用 IF NOT EXIST 选项来忽略这个异常</p>
</li>
<li>
<p><strong>EXTERNAL</strong> 关键字可以让用户创建一个外部表，在建表的同时指定一个指向实际数据的路径（LOCATION）</p>
</li>
<li>
<p><strong>LIKE</strong> 允许用户复制现有的表结构，但是不复制数据</p>
</li>
<li>
<p><strong>COMMENT</strong>可以为表与字段增加描述</p>
</li>
<li>
<p><strong>ROW FORMAT</strong></p>
</li>
</ul>
<blockquote>
<p>DELIMITED<br>
[FIELDS TERMINATED BY char]<br>
[COLLECTION ITEMS TERMINATED BY char]<br>
[MAP KEYS TERMINATED BY char]<br>
[LINES TERMINATED BY char]<br>
| SERDE serde_name<br>
[WITH SERDEPROPERTIES (property_name=property_value, property_name=property_value, …)]</p>
</blockquote>
<p>用户在建表的时候可以自定义 SerDe 或者使用自带的 SerDe。如果没有指定 ROW FORMAT 或者 ROW FORMAT DELIMITED，将会使用自带的 SerDe。在建表的时候，用户还需要为表指定列，用户在指定表的列的同时也会指定自定义的 SerDe，Hive 通过 SerDe 确定表的具体的列的数据。</p>
<ul>
<li><strong>STORED AS</strong></li>
</ul>
<blockquote>
<p>SEQUENCEFILE<br>
| TEXTFILE<br>
| RCFILE<br>
| INPUTFORMAT input_format_classname OUTPUTFORMAT output_format_classname</p>
</blockquote>
<p>如果文件数据是纯文本，可以使用 STORED AS TEXTFILE。如果数据需要压缩，使用 STORED AS SEQUENCE 。</p>
<p>举例：</p>
<pre><code>CREATE external TABLE IF NOT EXISTS t2(
id int,
name string,
age int
) COMMENT 'xx' 
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' 
STORED AS TEXTFILE;
</code></pre>
<h3 id="坑">坑</h3>
<p>Hive的Mysql驱动使用的是mysql-connector-java-5.1.17.jar，但是服务器的Mysql版本是5.6.40，当我在hive命令窗口执行"hive&gt;create database mydb2;"时报错：</p>
<blockquote>
<p>FAILED: Execution Error, return code 1 from<br>
org.apache.hadoop.hive.ql.exec.DDLTask. MetaException(message:You have<br>
an error in your SQL syntax; check the manual that corresponds to your<br>
MySQL server version for the right syntax to use near ‘OPTION<br>
SQL_SELECT_LIMIT=DEFAULT’ at line 1)</p>
</blockquote>
<p>于是到mysql客户端执行</p>
<pre><code>nysql&gt;SET OPTION SQL_SELECT_LIMIT=DEFAULT
</code></pre>
<p>报错</p>
<blockquote>
<p>ERROR 1064 (42000): You have an error in your SQL syntax; check the<br>
manual that corresponds to your MySQL server version for the right<br>
syntax to use near ‘OPTION SQL_SELECT_LIMIT=DEFAULT’ at line 1</p>
</blockquote>
<p>终于找到问题所在了，原来是因为低版本的mysql-connector-java-5.1.17.jar在创建数据库的时候发送测试语句<code>SET OPTION SQL_SELECT_LIMIT=DEFAULT</code>，上面可以看到mysql5.6的版本已经不在支持该语句，所以会报错，<br>
<strong>解决办法就是将connector版本升级到与MysqlServer一致的版本</strong>。</p>
<h2 id="load-data">load data</h2>
<p>虽然Hive已经提供了类sql的 insert语句支持插入数据，但是这种只支持单条插入的语句不能满足大多数生产情况，<strong>load data</strong>命令可以实现导入一个事先格式化好的文件，从而实现批量导入，这种做法有两个好处：</p>
<ul>
<li>批导入提升了效率</li>
<li>节省了HDFS中NameNode的管理开销</li>
</ul>
<h3 id="导入本地文件">导入本地文件</h3>
<p>加载服务器本地的文件</p>
<blockquote>
<p>load data local inpath ‘&lt;local_path&gt;’ [overwrite] into table &lt;table_name&gt;;</p>
</blockquote>
<p><strong>[overwrite]</strong>：覆盖原有数据，可选<br>
举例：</p>
<blockquote>
<p>load data local inpath ‘/root/test3.txt’ into table t2;</p>
</blockquote>
<h3 id="导入hdfs中的文件">导入HDFS中的文件</h3>
<blockquote>
<p>load data inpath ‘&lt;local_path&gt;’ [overwrite] into table &lt;table_name&gt;;</p>
</blockquote>
<h3 id="java代码演示">Java代码演示</h3>
<pre><code>/** 
 * LoadData测试 
 * * /
 public static void main(String[] args){  
    Connection connection = null;  
    PreparedStatement pstat = null;  
  
	 try{  
	        Class.forName("org.apache.hive.jdbc.HiveDriver");  
			connection = DriverManager.getConnection("jdbc:hive2://itaojin101:10000/test");  
			  pstat = connection.prepareStatement("load data local inpath '/root/test5.txt' into table t2");  
			  pstat.execute();  
			  System.out.println("--success--");  
	  }catch(Exception ex){  
	        ex.printStackTrace();  
	  }finally {  
	        if(connection != null){  
	            try {  
	                connection.close();  
				  } catch (SQLException e) {  
				                e.printStackTrace();  
				  }  
			  }  
			  
		if(pstat != null){  
			try {  
			    pstat.close();  
			} catch (SQLException e) {  
			    e.printStackTrace();  
			}  
	    }  
	  }  
}
</code></pre>
<h2 id="复制表">复制表</h2>
<h3 id="全表复制">全表复制</h3>
<p>在mysql中复制一张表的表结构以及数据：</p>
<blockquote>
<p>mysql&gt;create table &lt;table_name1&gt; as select * from &lt;table_name2&gt;;</p>
</blockquote>
<p>在Hive中也一样</p>
<blockquote>
<p>hive&gt;create table &lt;table_name1&gt; as select * from &lt;table_name2&gt;;</p>
</blockquote>
<h3 id="只复制表结构">只复制表结构</h3>
<p>mysql：</p>
<blockquote>
<p>mysql&gt;create table &lt;table_name1&gt; like &lt;table_name2&gt;;</p>
</blockquote>
<p>Hive:</p>
<blockquote>
<p>hive&gt;create table &lt;table_name1&gt; like &lt;table_name2&gt;;</p>
</blockquote>
<h2 id="一些会触发mr的sql">一些会触发MR的sql</h2>
<p>前面说过，Hive是将HQL转化为Hadoop的MR (MapReduce)来操作HDFS的。更严谨的说，只是一部分HQL会触发MR。</p>
<ul>
<li>普通查询</li>
</ul>
<blockquote>
<p>select * from table_name; 	<br>
select col_1, col_2 from table_name;<br>
select * from table_name where col_1=value;</p>
</blockquote>
<p>以上三种查询均是普通查询，即不带排序和聚合的查询，这种查询语句不会触发MR任务，Hive会立即返回结果，速度比较快。</p>
<p><img src="https://i.imgur.com/DQ7FA5d.png" alt="enter image description here"></p>
<ul>
<li>排序</li>
</ul>
<blockquote>
<p>select * from table_name order by col_1 DESC;</p>
</blockquote>
<p>由于排序的缘故，根据MR架构思想，在HDFS中，先在各个DataNode上查询到满足条件的数据集，然后NameNode统一收集各个NameNode反馈的数据集进行排序，所以会触发MR。<br>
<img src="https://i.imgur.com/jwYgzsx.png" alt="enter image description here"></p>
<p>也可以到Hadoop的MR管理页面查看刚刚执行作业<br>
<img src="https://i.imgur.com/sXNpFYW.png" alt="enter image description here"></p>
<ul>
<li>统计</li>
</ul>
<blockquote>
<p>select count(*) from table_name;</p>
</blockquote>
<p><img src="https://i.imgur.com/25k1gO3.png" alt="enter image description here"></p>
<h2 id="分区表-1">分区表</h2>
<p>在Mysql中，表分区的概念已然存在，在Hive中得到了完美的支持。</p>
<p>表分区的好处在于：<strong>将同一表中的不同业务场景，或者根据某种算法分组后的数据分开存储，当需要查询时，根据分区规则查询指定分区数据，避免全局扫描搜索，从而提高搜索效率，这也是Hive优化手段之一，对于超大数据这种效率提升越发明显。</strong></p>
<ul>
<li><strong>创建分区表</strong><br>
在创建表格时，就应该显示指定该表为分区表，且还要知名分区字段有哪些。</li>
</ul>
<blockquote>
<pre><code>CREATE TABLE t3(
id int,
name string,
age int
) 
PARTITIONED BY (Year INT, Month INT) 
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' ;
</code></pre>
</blockquote>
<p>存在两个分区字段 <strong>Year</strong> 和 <strong>Month</strong><br>
创建完毕后，通过命令</p>
<blockquote>
<p>desc t3;</p>
</blockquote>
<p>查看表结构信息，可以看到分区元信息：<br>
<img src="https://i.imgur.com/TzVitgT.png" alt="enter image description here"></p>
<ul>
<li><strong>添加分区数据</strong></li>
</ul>
<blockquote>
<p>alter table t3 add partition (year=2018, month=5);</p>
</blockquote>
<p>通过以下命令查看表格的分区信息：</p>
<blockquote>
<p>show partitions t3;</p>
</blockquote>
<p><img src="https://i.imgur.com/qy9S2jT.png" alt="enter image description here"></p>
<p>添加的分区数据，反映到HDFS，是一个个文件夹：</p>
<blockquote>
<p>hive&gt;dfs -lsr / ;</p>
</blockquote>
<p><img src="https://i.imgur.com/ramLihG.png" alt="enter image description here"></p>
<ul>
<li><strong>导入数据</strong><br>
还是以 <strong>load data</strong> 命令为例：</li>
</ul>
<blockquote>
<p>load data local inpath ‘/root/test.txt’ into table t3 partition(year=2018,month=5);</p>
</blockquote>
<p><img src="https://i.imgur.com/tXnZbRk.png" alt="enter image description here"></p>
<p>从上图可以得到以下信息：<strong>建表时并没有创建year和month字段，Hive会自动将分区字段作为伪字段存储，在查询时可以用伪字段作为查询条件</strong>。<br>
<img src="https://i.imgur.com/8egUZnP.png" alt="enter image description here"></p>
<p>再看看HDFS中的存储情况：</p>
<pre><code>hive&gt;dfs -lsr /;
</code></pre>
<p><img src="https://i.imgur.com/xyPGq5c.png" alt="enter image description here"></p>
<ul>
<li>
<p><strong>思考？</strong><br>
在导入数据之前，如果不添加分区信息，即不执行</p>
<blockquote>
<p>alter table t3 add partition (year=2018, month=5);</p>
</blockquote>
<p>命令，导入数据会报错吗？</p>
</li>
</ul>
<p>我们先来做个实验，直接执行：</p>
<pre><code>jdbc:hive2://localhost:10000/test&gt; load data local inpath '/root/test.txt' into table t3 partition(year=2018,month=4);
</code></pre>
<p>再查看HDFS目录结构，数据导入了！</p>
<p>结论：<strong>分区信息会被自动创建</strong>。</p>
<ul>
<li><strong>删除分区</strong></li>
</ul>
<blockquote>
<p>alter table t3 drop if exists partition (year=2018, month=4);</p>
</blockquote>
<p><strong>执行完毕后，不仅分区信息被删除，分区下的数据也全部被删除。原因就是该表是managed table类型的表。</strong></p>
<h2 id="桶表">桶表</h2>
<ul>
<li>
<p>定义：简单的理解，根据某个字段的值Hash后，将数据分成多个bucket存储。</p>
</li>
<li>
<p>与分区表的区别：<strong>在HDFS层面，分区表是按照文件夹区分存储位置，桶表则是在同一文件夹下，按照不同编号的文件区分存储位置</strong>。</p>
</li>
<li>
<p>创建：</p>
</li>
</ul>
<blockquote>
<p>CREATE TABLE t4(<br>
id int, 	<br>
name string, 	<br>
age int 	<br>
)<br>
CLUSTERED BY (id) 	INTO 3 BUCKETS<br>
ROW FORMAT DELIMITED FIELDS TERMINATED BY ‘,’ ;</p>
</blockquote>
<p>指定分桶的字段是 <strong>id</strong>，桶数为 <strong>3</strong></p>
<ul>
<li>载入数据
<ol>
<li><strong>load data</strong> 命令将对桶表失效</li>
<li>采用 <strong>insert</strong> 命令载入其他表的数据
<blockquote>
<p>insert into t4 select id,name,age from t3;</p>
</blockquote>
</li>
<li>查看HDFS文件目录
<blockquote>
<p>$hive&gt;dfs -lsr /;</p>
</blockquote>
</li>
</ol>
</li>
</ul>
<p><img src="https://i.imgur.com/RbFU4bC.png" alt="enter image description here"><br>
一共有3个桶，编号分别为：<strong>000000_0</strong>，<strong>000001_0</strong>，<strong>000002_0</strong>。<br>
再次执行插入语句，且查看HDFS目录：<br>
<img src="https://i.imgur.com/06tW6Ap.png" alt="enter image description here"></p>
<ul>
<li>
<p>桶数的选择<br>
<img src="https://i.imgur.com/lOgO7uH.png" alt="enter image description here"><br>
<strong>总结： 桶数太多太少都不好，有一个粗算规则：先评估表的数量级，再除以hdfs的block的大小的2倍，商值就是桶数。</strong></p>
</li>
<li>
<p>桶表的优点：根据分桶字段查询极快。</p>
</li>
</ul>
<h2 id="导入导出-数据">导入/导出 数据</h2>
<h3 id="导出数据">导出数据</h3>
<blockquote>
<p>export table &lt;table_name&gt; to ‘&lt;path_&gt;’;</p>
<ul>
<li>&lt;table_name&gt;: 表名</li>
<li>&lt;path_&gt;: 导出的HDFS路径</li>
</ul>
</blockquote>
<p>举例：</p>
<blockquote>
<p>export table t3 to ‘/root/t3.txt’;</p>
</blockquote>
<p>使用以下命令查看hdfs目录：</p>
<blockquote>
<p>$hive&gt;dfs -lsr /;</p>
</blockquote>
<p><img src="https://i.imgur.com/lJSdzBb.png" alt="enter image description here"></p>
<p>一共导出了两部分数据：</p>
<blockquote>
<ul>
<li>_metadata：表结构数据</li>
<li>test.txt：文件</li>
</ul>
</blockquote>
<h3 id="导入数据">导入数据</h3>
<h2 id="排序">排序</h2>
<h3 id="order-by">order by</h3>
<p>Hive中的order by跟传统的sql语言中的order by作用是一样的，会对查询的结果做一次全局排序，所以说，只有hive的sql中制定了order by所有的数据都会到同一个reducer进行处理（不管有多少map，也不管文件有多少的block只会启动一个reducer）。但是对于大量数据这将会消耗很长的时间去执行。</p>
<p>这里跟传统的sql还有一点区别：如果指定了hive.mapred.mode=strict（默认值是nonstrict）,这时就必须指定limit来限制输出条数，原因是：所有的数据都会在同一个reducer端进行，数据量大的情况下可能不能出结果，那么在这样的严格模式下，必须指定输出的条数。</p>
<h3 id="sort-by">sort by</h3>
<p>Hive中指定了sort by，那么在每个reducer端都会做排序，也就是说保证了局部有序（每个reducer出来的数据是有序的，但是不能保证所有的数据是有序的，除非只有一个reducer），好处是：执行了局部排序之后可以为接下去的全局排序提高不少的效率（其实就是做一次归并排序就可以做到全局排序了）。</p>
<h3 id="distribute-by和sort-by一起使用">distribute by和sort by一起使用</h3>
<p>ditribute by是控制map的输出在reducer是如何划分的，举个例子，我们有一张表，mid是指这个store所属的商户，money是这个商户的盈利，name是这个store的名字</p>

<table>
<thead>
<tr>
<th>mid</th>
<th>money</th>
<th>name</th>
</tr>
</thead>
<tbody>
<tr>
<td>AA</td>
<td>15.0</td>
<td>商店1</td>
</tr>
<tr>
<td>AA</td>
<td>20.0</td>
<td>商店2</td>
</tr>
<tr>
<td>BB</td>
<td>22.0</td>
<td>商店3</td>
</tr>
<tr>
<td>CC</td>
<td>44.0</td>
<td>商店4</td>
</tr>
</tbody>
</table><blockquote>
<p>select mid, money, name from store distribute by mid sort by mid asc, money asc;</p>
</blockquote>
<p>我们所有的mid相同的数据会被送到同一个reducer去处理，这就是因为指定了distribute by mid，这样的话就可以统计出每个商户中各个商店盈利的排序了（这个肯定是全局有序的，因为相同的商户会放到同一个reducer去处理）。这里需要注意的是distribute by必须要写在sort by之前。</p>
<h3 id="cluster-by">cluster by</h3>
<p>cluster by的功能就是distribute by和sort by相结合，如下2个语句是等价的：</p>
<blockquote>
<p>select mid, money, name from store cluster by mid;<br>
等价于<br>
select mid, money, name from store distribute by mid sort by mid;</p>
</blockquote>
<p>如果需要获得与上面的语句一样的效果：</p>
<blockquote>
<p>select mid, money, name from store cluster by mid sort by money;</p>
</blockquote>
<p><strong>注意被cluster by指定的列只能是降序，不能指定asc和desc。</strong></p>
<p><img src="https://i.imgur.com/AWCIeVm.png" alt="enter image description here"></p>
<h2 id="作业参数">作业参数</h2>
<ul>
<li>设置reducetask的字节数</li>
</ul>
<blockquote>
<p>$hive&gt;set hive.exec.reducers.bytes.per.reducer=xxx</p>
</blockquote>
<ul>
<li>设置reduce task的最大任务数</li>
</ul>
<blockquote>
<p>$hive&gt;set hive.exec.reducers.max=0</p>
</blockquote>
<ul>
<li>设置reducetask个数</li>
</ul>
<blockquote>
<p>$hive&gt;set mapreduce.job.reduces=0</p>
</blockquote>


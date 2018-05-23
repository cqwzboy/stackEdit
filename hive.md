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

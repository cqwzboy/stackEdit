---


---

<h1 id="前言">前言</h1>
<p>Hive是数据仓库，其特点就是 <strong>一次写入，多次读取</strong>，所以不支持数据的<strong>删除</strong> 和 <strong>修改</strong>。</p>
<p>HBase是一种<strong>实时随机</strong>查询的数据仓库，特点就是<strong>查询快</strong>，但是不支持像Hive那样较为复杂的<strong>SQL计算</strong>。</p>
<p>故而，很自然地会想到将Hive和HBase整合使用，各取所长，从而达到 <strong>查询快</strong>、<strong>支持删除和修改</strong> 且 <strong>能进行复杂SQL运算</strong>的目的。</p>
<p>目前较为流行的整合方式：</p>
<ul>
<li>Hive读取操作HBase的数据。即数据存放在HBase中，Hive可以对其数据进行去读和计算。</li>
</ul>
<p>Tips：</p>
<blockquote>
<p>整合方式应该不止这一种，后期发现了会更新</p>
</blockquote>
<h1 id="前提条件">前提条件</h1>
<p>默认已经具备以下环境：</p>
<ul>
<li>HDFS 和 YARN 环境</li>
<li>Hive环境，且已经和Hadoop完成整合</li>
<li>HBase环境，且已经和Hadoop完成整合</li>
</ul>
<h2 id="注意">注意</h2>
<ul>
<li>在部署HiveServer节点时，一定要将该节点部署在具备 HBase环境的机器上（HMaster 或者 HRegionServer均可），因为HIverServer在读取HBase中数据时，会访问本地的<strong>16020</strong>端口，此端口是HBase中对外提供RPC服务的端口。</li>
</ul>
<h1 id="hive读取hbase中的数据">Hive读取HBase中的数据</h1>
<h2 id="在hbase中建表">在HBase中建表</h2>
<blockquote>
<p>create ‘ns1:test’, ‘f1’;</p>
</blockquote>
<p>在HBase中创建表比较简单。上诉建表语句解读为：创建一个<strong>命名空间</strong>（namespace）为 ns1 的，表名为 test 的表，该表有一个<strong>列族</strong>（family） f1 。</p>
<h2 id="在hive中建表">在Hive中建表</h2>
<blockquote>
<p>CREATE EXTERNAL TABLE test<br>
(id int, name string, age int, address string)<br>
ROW FORMAT<br>
SERDE ‘org.apache.hadoop.hive.hbase.HBaseSerDe’<br>
STORED BY ‘org.apache.hadoop.hive.hbase.HBaseStorageHandler’<br>
WITH SERDEPROPERTIES (<br>
‘serialization.format’=’\t’,<br>
‘hbase.columns.mapping’=’:key,f1:name,f1:age,f1:address’,<br>
‘field.delim’=’\t’<br>
)<br>
TBLPROPERTIES (‘<a href="http://hbase.table.name">hbase.table.name</a>’=‘ns1:test’);</p>
</blockquote>
<p>建表语句解读：</p>
<ul>
<li>创建一张<strong>外部表</strong>，字段分别是 id，name，age，address。</li>
<li>SERDE （序列化器/反序列化器）采用Hive自带的<code>org.apache.hadoop.hive.hbase.HBaseSerDe</code>解析器。</li>
<li>STORED BY，通过<code>org.apache.hadoop.hive.hbase.HBaseStorageHandler</code>转换和存储到HBase</li>
<li>WITH SERDEPROPERTIES，定义解析器的初始化参数，以供<code>org.apache.hadoop.hive.hbase.HBaseSerDe</code>解析器初始化。初始化参数中，定义了Hive和Hbase的表中字段的映射关系，如id字段作为HBase的行id，name映射HBase中f1列族下的name列，依次类推。</li>
<li>TBLPROPERTIES，Hive表<code>test</code>与HBase中的表<code>ns1:test</code>建立映射关系。</li>
</ul>
<p>这里涉及到一些Hive和HBase整合后的一些知识点。</p>
<h3 id="hive表分类">Hive表分类</h3>
<p>Hive的表格分类，按照Hive对数据的控制权的维度来分，分为 <strong>内部表</strong> 和 <strong>外部表</strong>：</p>
<ul>
<li>
<p>内部表：是Hive中最常用的一种表类型，内部表中的数据完全受Hive控制，即其他组件，如Pig，HBase，不能访问内部表的数据，HIve对内部表中的数据具有完全控制权，所以，<strong>当删除内部表时，源数据和表内数据一起删除</strong>。</p>
</li>
<li>
<p>外部表：当Hive需要与其他组件共享数据时，推荐使用外部表。顾名思义，外部表就是存在于HIve之外的表，Hive对其没有控制权，因为数据是共享的。所以，<strong>当删除外部表时，Hive只能讲元数据删除，表内数据不删除</strong>。</p>
<ol>
<li>外部表具备以下优势：比如有这么一个业务场景，有一份数据需要被共享给多个组件（Hive，Pig等）查询计算，name就可以约定一个hdfs的文件目录，这个目录下存放共享数据，并约定好文件的格式是“一行六个字段，字段之间以逗号隔开”，那么Hive和Pig各自建立表格，且设置类似如下功能<code>fields terminated by ','</code>，那么Hive和Pig就能共享一个目录下的数据，且再次将一份相同文档规范的文件放进该目录下时，Hive和Pig会自动识别读取，是不是感觉很赞。</li>
</ol>
</li>
</ul>
<h3 id="hive整合hbase后的表分类">Hive整合HBase后的表分类</h3>
<p>但是，当Hive和HBase整合后，表格的类型大致可以分为四种：</p>
<ol>
<li><strong>managed native</strong>: what you get by default with CREATE TABLE</li>
<li><strong>external native</strong>: what you get with CREATE EXTERNAL TABLE when no STORED BY clause is specified</li>
<li><strong>managed non-native</strong>: what you get with CREATE TABLE when a STORED BY clause is specified; Hive stores the definition in its metastore, but does not create any files itself; instead, it calls the storage handler with a request to create a corresponding object structure</li>
<li><strong>external non-native</strong>: what you get with CREATE EXTERNAL TABLE when a STORED BY clause is specified; Hive registers the definition in its metastore and calls the storage handler to check that it matches the primary definition in the other system</li>
</ol>
<p>而整合Hive和HBase时创建的HIve表是 <strong>external non-native</strong> 类型，所以当我们在给Hive的表load数据时会报错：</p>
<blockquote>
<p>A non-native table cannot be used as target for LOAD</p>
</blockquote>
<p>为了解决以上问题，我们采用在HIve中创建一张表结构与关联表一致的<strong>内部表</strong>，我们把数据直接load到这张临时表，然后再通过<code>insert overwrite table</code>命令来将临时表中的数据导入关联表。</p>
<ul>
<li>创建临时表</li>
</ul>
<blockquote>
<p>CREATE TABLE IF NOT EXISTS test_temp<br>
( id int, name string,  age int,  address string )<br>
COMMENT 'test_temp’<br>
ROW FORMAT<br>
DELIMITED FIELDS TERMINATED BY ','<br>
STORED AS TEXTFILE;</p>
</blockquote>
<ul>
<li>导入数据：</li>
</ul>
<blockquote>
<p>LOAD DATA INPATH ‘…/path/…’ into table test_temp;</p>
</blockquote>
<ul>
<li>临时表数据写入关联表</li>
</ul>
<blockquote>
<p>INSERT OVERWRITE TABLE test SELECT * FROM test_temp;</p>
</blockquote>


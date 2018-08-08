---


---

<h1 id="本地项目发布到nexus私服">本地项目发布到Nexus私服</h1>
<ul>
<li>
<p>前提<br>
已具备 jdk, Maven和Nexus环境</p>
</li>
<li>
<p>在Maven的配置文件settings.xml中<br>
1. 在 <code>&lt;mirrors&gt;</code> 下新增镜像</p>
<pre><code>  &lt;mirror&gt;
     &lt;id&gt;pri-aliyun&lt;/id&gt;
     &lt;mirrorOf&gt;*&lt;/mirrorOf&gt;     
     &lt;url&gt;http://47.106.201.17:8081/nexus/content/repositories/releases/&lt;/url&gt;
   &lt;/mirror&gt;
</code></pre>
<p>其中，<code>&lt;id&gt;</code>是镜像id，随便取名，保证唯一性；<code>&lt;url&gt;</code>是Nexus服务的地址</p>
<ol start="2">
<li>
<p>在<code>&lt;servers&gt;</code>下新增验证信息</p>
<pre><code> &lt;server&gt;
   &lt;id&gt;pri-aliyun-snapshots&lt;/id&gt;
   &lt;username&gt;admin&lt;/username&gt;
   &lt;password&gt;fqqamm31415927&lt;/password&gt;
 &lt;/server&gt;
 &lt;server&gt;
   &lt;id&gt;pri-aliyun-releases&lt;/id&gt;
   &lt;username&gt;admin&lt;/username&gt;
   &lt;password&gt;fqqamm31415927&lt;/password&gt;
 &lt;/server&gt;
</code></pre>
</li>
</ol>
<p>这里有两个配置，一个是针对快照的验证，另一个是针对正式版本的验证。<code>&lt;id&gt;</code>是镜像名称拼接上 <strong>快照/正式</strong> 类型。</p>
<ol start="3">
<li>
<p>在<code>&lt;profiles&gt;</code>下新增</p>
<pre><code> &lt;profile&gt;
    &lt;id&gt;pri-aliyun&lt;/id&gt;
    &lt;repositories&gt;
        &lt;repository&gt;
            &lt;id&gt;pri-aliyun&lt;/id&gt;
            &lt;name&gt;Aliyun&lt;/name&gt;
            &lt;url&gt;http://47.106.201.17:8081/nexus/content/groups/public/&lt;/url&gt;
            &lt;releases&gt;&lt;enabled&gt;true&lt;/enabled&gt;&lt;/releases&gt;
            &lt;snapshots&gt;&lt;enabled&gt;true&lt;/enabled&gt;&lt;/snapshots&gt;
        &lt;/repository&gt;
    &lt;/repositories&gt;
    &lt;pluginRepositories&gt;
        &lt;pluginRepository&gt;
            &lt;id&gt;pri-aliyun&lt;/id&gt;
            &lt;name&gt;Aliyun&lt;/name&gt;
            &lt;url&gt;http://47.106.201.17:8081/nexus/content/groups/public/&lt;/url&gt;
            &lt;releases&gt;&lt;enabled&gt;true&lt;/enabled&gt;&lt;/releases&gt;
            &lt;snapshots&gt;&lt;enabled&gt;true&lt;/enabled&gt;&lt;/snapshots&gt;
        &lt;/pluginRepository&gt;
    &lt;/pluginRepositories&gt;
  &lt;/profile&gt;
</code></pre>
</li>
</ol>
<p>其中<code>&lt;id&gt;</code>和镜像id相同</p>
</li>
<li>
<p>在项目的pom.xml的<code>&lt;project&gt;</code>标签下</p>
<pre><code>  &lt;distributionManagement&gt;  
  	 &lt;repository&gt; 
  		 &lt;id&gt;nexus-releases&lt;/id&gt;  
  		 &lt;name&gt;Nexus Release Repository&lt;/name&gt;  
  		 &lt;url&gt;http://192.168.1.105:8081/nexus/content/repositories/releases/&lt;/url&gt;  
  	 &lt;/repository&gt; 
  	 &lt;snapshotRepository&gt;
  		  &lt;id&gt;nexus-snapshots&lt;/id&gt;  
  		  &lt;name&gt;Nexus Snapshot Repository&lt;/name&gt;  
  		  &lt;url&gt;http://192.168.1.105:8081/nexus/content/repositories/snapshots/&lt;/url&gt;  
  	 &lt;/snapshotRepository&gt;  
    &lt;/distributionManagement&gt;
</code></pre>
<p>注意：这里的<code>&lt;id&gt;</code>必须和 settings.xml中<code>&lt;server&gt;</code>标签下的id一致。</p>
</li>
</ul>


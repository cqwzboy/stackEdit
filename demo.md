---


---

<h1 id="接口文档">接口文档</h1>
<ol>
<li>获取错误日志列表<br>
描述：获取错误日志的列表，支持分页功能，且分页参数为必传参数
<blockquote>
<p>request</p>
</blockquote>
</li>
</ol>

<table>
<thead>
<tr>
<th>编号</th>
<th>参数</th>
<th>类型</th>
<th>说明</th>
<th>是否必填</th>
</tr>
</thead>
<tbody>
<tr>
<td>1</td>
<td>pageSize</td>
<td>int</td>
<td>每页中的数据量</td>
<td>是</td>
</tr>
<tr>
<td>2</td>
<td>pageNumber</td>
<td>int</td>
<td>页码，从1开始</td>
<td>是</td>
</tr>
<tr>
<td>3</td>
<td>shopName</td>
<td>string</td>
<td>店铺名称</td>
<td>否</td>
</tr>
<tr>
<td>4</td>
<td>date</td>
<td>string</td>
<td>日期</td>
<td>否</td>
</tr>
</tbody>
</table><p>举例：<code>http://192.168.1.107/tj_getdata_web/LogManage/getShopLogs?pageSize=20&amp;pageNumber=1</code></p>
<blockquote>
<p>response</p>
</blockquote>
<pre><code>{
"data": {
	"endRow": 5,
	"firstPage": 1,
	"hasNextPage": true,
	"hasPreviousPage": false,
	"isFirstPage": true,
	"isLastPage": false,
	"lastPage": 8,
	"list": [{
		"date": "2018-05-07",
		"errorNum": 4,
		"id": 1,
		"sessionKey": "62028226751f7beaf543cd6c3a4a1bdff101043d13b30691996241556",
		"shopId": 3847,
		"shopName": "evedacc爱芙德旗舰店"
	}, {
		"date": "2018-05-07",
		"errorNum": 2,
		"id": 2,
		"sessionKey": "6101601f8108bb2a8cd36782295607800d99c056cefcd9f2716292474",
		"shopId": 4107,
		"shopName": "迈路士跑步"
	}, {
		"date": "2018-05-07",
		"errorNum": 1,
		"id": 3,
		"sessionKey": "6202004e1d7ceg5c03db1cd810a27d7dc134f7dea1356b21848432360",
		"shopId": 4185,
		"shopName": "德亚旗舰店"
	}, {
		"date": "2018-05-07",
		"errorNum": 1,
		"id": 4,
		"sessionKey": "6200514c2df22f00471063ZZa43f800abbf8bac05fa13fa1969813859",
		"shopId": 4187,
		"shopName": "瓦伦丁酒类旗舰店"
	}, {
		"date": "2018-05-07",
		"errorNum": 1,
		"id": 5,
		"sessionKey": "6200a237e753e017021b654ea03f83ceg141317ae0c7c9e1968929055",
		"shopId": 4189,
		"shopName": "品利食品旗舰店"
	}],
	"navigateFirstPage": 1,
	"navigateLastPage": 8,
	"navigatePages": 8,
	"navigatepageNums": [1, 2, 3, 4, 5, 6, 7, 8],
	"nextPage": 2,
	"pageNum": 1,
	"pageSize": 5,
	"pages": 9,
	"prePage": 0,
	"size": 5,
	"startRow": 1,
	"total": 41
},
"message": "成功",
"status": true
</code></pre>
<p>}</p>


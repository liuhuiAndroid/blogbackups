###一、了解HTML
```
标题标签  <h1></h1>
水平线标签  <hr />
段落标签  <p></p>
字体标签  <font color="#0000FF" size="100" face="楷体">楷体</font>
粗体标签  <b></b>
斜体标签  <i></i>
换行标签  <br />
图片标签  <img src="../../img/logo.png" width="150px" height="30px" alt="用于在图像无法显示时,代替图像显示在浏览器中的内容"/>
超链接标签  <a href="https://www.baidu.com" target="_blank">百度</a>  _blank 在新窗口中打开被链接文档。 _self 默认
无序列表标签
<ul type="square">
  <li>CSDN</li>
  <li>百度</li>
  <li>京东</li>
</ul>
有序列表标签
<ol start="4" reversed="reversed" type="a">
  <li>CSDN</li>
  <li>百度</li>
  <li>京东</li>
</ol>
框架集标签
<frameset rows="20%,*">
	<frame src="top.html" />
	<frameset cols="20%,*">
		<frame src="left.html" />
		<frame name="right" />
	</frameset>
</frameset>
```
```
表格标签
<!--cellspacing:设置边框与边框的间距，cellpadding:设置的是边框与内容的间距-->
<table border="1px" width="450px" height="150px" align="center" 
			bgcolor="pink" cellspacing="0px"  cellpadding="0px">
			<tr height="100px" bgcolor="gold">
				<td>11</td>
				<td>12</td>
				<td>13</td>
			</tr>
			<tr>
				<td>21</td>
				<td>22</td>
				<td>23</td>
			</tr>
			<tr>
				<td>31</td>
				<td>32</td>
				<td>33</td>
			</tr>
</table>
```
```
表格的跨行跨列操作
<table border="1px" width="500px" height="200px" align="center" cellspacing="0px" cellpadding="0px">
	<tr>
		<td colspan="2" align="center" width="250px" height="50px">
			<img src="../../img/1.jpg" height="100%" width="100%"/>
		</td>
		<td>13</td>
		<td>14</td>
	</tr>
	<tr>
		<td>21</td>
		<td colspan="2" rowspan="2" align="center">
			<table border="1px" align="center" width="100%" height="100%">
				<tr>
					<td>11</td>
					<td>12</td>
					<td>13</td>
				</tr>
				<tr>
					<td>21</td>
					<td>22</td>
					<td>23</td>
				</tr>
				<tr>
					<td>31</td>
					<td>32</td>
					<td>33</td>
				</tr>
			</table>
		</td>
		
		<td>24</td>
	</tr>
	<tr>
		<td>31</td>
		<td rowspan="2" align="center">34</td>
	</tr>
	<tr>
		<td>41</td>
		<td>42</td>
		<td>43</td>
	</tr>
</table>
```
```
表单标签
<form action="#" method="get">
	隐藏字段:<input type="hidden" name="id" value="1" /><br />
	用户名：<input type="text" name="username" readonly="readonly" value="zhangsan" size="40px" maxlength="5" placeholder="请输入用户名"/><br />
	密码：<input type="password" name="password" required="required"/><br />
	确认密码：<input type="password" name="repassword"/><br />
	性别：<input type="radio" name="sex" value="1"/>男
	<input type="radio" name="sex" value="0" checked="checked"/>女<br />
	爱好：<input type="checkbox" name="hobby" value="钓鱼"/>钓鱼
	<input type="checkbox" name="hobby" value="打电动"/>打电动
	<input type="checkbox" name="hobby" value="写代码" checked="checked"/>写代码<br />
	头像：<input type="file" name="file"/><br />
	籍贯：<select name="province">
		<option>--请选择--</option>
		<option value="北京">北京</option>
		<option value="上海" selected="selected">上海</option>
		<option value="广州">广州</option>
	</select><br />
	自我介绍：
		<textarea name="interduce">
		</textarea><br />
	提交按钮：<input type="submit" value="注册"/><br />
	普通按钮：<input type="button" value="注册"/><br />
	重置按钮：<input type="reset" />
</form>
```
###二、了解CSS
```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>div效果演示</title>
		<style>
			div{
				border: 1px solid red;
				width: 200px;
				height: 100px;
			}
			span{
				font-size: 30px;
				color: red;
			}
		</style>
	</head>
	<body>
		<div id="">
			生命不息，奋斗不止
		</div>
		生命不息，奋斗不止
		<br>
		<span>
			生命不息，奋斗不止
		</span>
		<br>
		生命不息，奋斗不止
	</body>
</html>
```
1. CSS选择器
```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>id选择器</title>
		<style>
			/*元素选择器*/
			div{
				font-size: 30px;
				color: pink;
			}
			/*类选择器*/
			.div2{
				font-size: 30px;
				color: green;
			}
			/*id选择器*/
			#div5{
				font-size: 30px;
				color: yellow;
			}
			/*层级选择器*/
			div p{
				font-size: 30px;
				color: blue;
			}
			/*属性选择器*/
			input[type='text']{
				background-color: pink;
			}
			/*属性选择器*/
			input[type='password']{
				background-color: pink;
			}
		</style>
	</head>
	<body>
		<div>
            生命不息，奋斗不止
		</div>
		<div class="div2">
            生命不息，奋斗不止 类选择器
		</div>
		<div>
            生命不息，奋斗不止
		</div>
		<div id="div5">
            生命不息，奋斗不止 id选择器
		</div>
		<div>
            生命不息，奋斗不止
		</div>
		<div>
			<p>
				生命不息，奋斗不止 层级选择器
			</p>
		</div>
		<p>
			生命不息，奋斗不止
		</p>
		用户名：<input type="text" name="username"/><br />
		密码：<input type="password" name="password"/>
	</body>
</html>
```
2. CSS引入方式
上面所用到的这种引入方式是内部引入。
```
行内引入
<div style="font-size: 20px;color: blue;">
	生命不息，奋斗不止
</div>
外部引入
<head>
	<meta charset="UTF-8">
	<title>选择器</title>
	<link rel="stylesheet" href="style.css" type="text/css"/>
</head>
```
3. CSS浮动
```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>CSS浮动</title>
		<style>
			#one{
				border: 1px solid red;
				width: 600px;
				height: 150px;
				float: left;
			}
			#two{
				border: 1px solid black;
				width: 600px;
				height: 190px;
				float:left;
			}
			#three{
				border: 1px solid blue;
				width: 600px;
				height: 160px;
				float: left;
			}
			/*清除浮动*/
			#clear{
				clear: both;
			}
		</style>
	</head>
	<body>
		<div id="one">
		</div>
		<div id="clear">
		</div>
		<div id="two">
		</div>
		<div id="three">
		</div>
	</body>
</html>
```
###三、了解JS
1. JS获取元素
```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>获取元素</title>
		<script>
			window.onload = function(){
				//获取页面指定位置的元素
				var uEle = document.getElementById("username");
				//alert(uEle);
				//获取页面指定位置的内容(值)
				var uValue = uEle.value;
				alert(uValue);
			}
		</script>
	</head>
	<body>
		用户名：<input type="text" name="username" id="username" value="123456"/><br />
		密码: <input type="password" name="password" />
	</body>
</html>
``` 
JS正则表达式校验
```
			function checkForm(){
				/*校验邮箱*/
				var eValue = document.getElementById("eamil").value;
				if(!/^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(.[a-zA-Z0-9_-])+/.test(eValue)){
					alert("邮箱格式不正确!");
					return false;
				}
			}
```
2. JS实现轮播图效果
```
	<head>
		<script>
			function init(){
				//书写轮图片显示的定时操作
				setInterval("changeImg()",3000);
			}
			//书写函数
			var i=0
			function changeImg(){
				i++;
				//获取图片位置并设置src属性值
				document.getElementById("img1").src="../img/"+i+".jpg";
				if(i==3){
					i=0;
				}
			}
		</script>
	</head>
	<body onload="init()">
	</body>
```
3. BOM对象
```
<!DOCTYPE html>
<html>
<head>
	<meta charset="UTF-8">
	<title>BOM对象</title>
	<script>
        function back(){
            //history对象可以实现返回上一页
            history.back();
        }
	</script>
</head>
<body>
	<input type="button" value="返回上一页" onclick="back()"/><br />
	<!--location对象-->
	<input type="button" value="跳转页面" onclick="javascript:location.href='02_History对象.html'"/>
</body>
</html>
```
4. JS实现弹出广告效果
```
JS可以用外部引入的方式
<script type="text/javascript" src="1.js" ></script>
```
```
function init(){
	//1.设置显示广告图片的定时操作
	time = setInterval("showAd()",3000);
}

//2.书写显示广告图片的函数
function showAd(){
	//3.获取广告图片的位置
	var adEle = document.getElementById("img2");
	//4.修改广告图片元素里面的属性让其显示
	adEle.style.display = "block";
	//5.清除显示图片的定时操作
	clearInterval(time);
	//6.设置隐藏图片的定时操作
	time = setInterval("hiddenAd()",3000);
}

//7.书写隐藏广告图片的函数
function hiddenAd(){
	//8.获取广告图片并设置其style属性的display值为none
	document.getElementById("img2").style.display= "none";
	//9.清除隐藏广告图片的定时操作
	clearInterval(time);
}
```
```
<img src="../img/f001a62f-a49d-4a4d-b56f-2b6908a0002c_g.jpg" width="100%" style="display: none" id="img2"/>
```
5. JS实现表单校验
```
<script>
	function showTips(id,info){
		document.getElementById(id+"span").innerHTML="<font color='gray'>"+info+"</font>";
	}
	function check(id,info){
		//1.获取用户输入的用户名数据
		var uValue = document.getElementById(id).value;
		//2.进行校验
		if(uValue==""){
			document.getElementById(id+"span").innerHTML="<font color='red'>"+info+"</font>";
		}else{
			document.getElementById(id+"span").innerHTML="";
		}
	}
</script>
```
```
<input type="text" name="user" size="34px" id="user" onfocus="showTips('user','用户名必填！')" onblur="check('user','用户名不能为空！')"/><span id="userspan"></span>
<input type="password" name="password" size="34px" id="password" onfocus="showTips('password','密码必填')" onblur="check('password','密码不能为空!')"/><span id="passwordspan"></span>
```
6. JS完成表格隔行换色
测试onload事件
```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<script>
			window.onload = function(){
				document.getElementById("btn").onclick = function(){
					location.href="惊喜.html"
				}
			}
		</script>
	</head>
	<body>
		<input type="button" value="点我有惊喜！" id="btn" />
	</body>
</html>
```
表格隔行换色
```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>表格隔行换色</title>
		<script>
			window.onload = function(){
				//1.获取表格
				var tblEle = document.getElementById("tbl");
				//2.获取表格中tbody里面的行数(长度)
				var len = tblEle.tBodies[0].rows.length;
				//3.对tbody里面的行进行遍历
				for(var i=0;i<len;i++){
					if(i%2==0){
						//4.对偶数行设置背景颜色
						tblEle.tBodies[0].rows[i].style.backgroundColor="pink";
					}else{
						//5.对奇数行设置背景颜色
						tblEle.tBodies[0].rows[i].style.backgroundColor="gold";
					}
				}
			}
		</script>
	</head>
	<body>
		<table border="1" width="500" height="50" align="center" id="tbl">
			<thead>
				<tr>
					<th>编号</th>
					<th>姓名</th>
					<th>年龄</th>
				</tr>
			</thead>
			<tbody>
				<tr >
					<td>1</td>
					<td>张三</td>
					<td>22</td>
				</tr>
				<tr >
					<td>2</td>
					<td>李四</td>
					<td>25</td>
				</tr>
				<tr >
					<td>3</td>
					<td>王五</td>
					<td>27</td>
				</tr>
				<tr >
					<td>4</td>
					<td>赵六</td>
					<td>29</td>
				</tr>
				<tr >
					<td>5</td>
					<td>田七</td>
					<td>30</td>
				</tr>
				<tr >
					<td>6</td>
					<td>汾九</td>
					<td>20</td>
				</tr>
			</tbody>
		</table>
	</body>
</html>
```
表格高亮显示
```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>表格高亮显示</title>
		<script>
			function changeColor(id,flag){
				if(flag=="over"){
					document.getElementById(id).style.backgroundColor="red";
				}else if(flag=="out"){
					document.getElementById(id).style.backgroundColor="white";
				}
			}
		</script>
	</head>
	<body>
		<table border="1" width="500" height="50" align="center">
			<thead>
				<tr>
					<th>编号</th>
					<th>姓名</th>
					<th>年龄</th>
				</tr>
			</thead>
			<tbody>
				<tr onmouseover="changeColor('tr1','over')" id="tr1" onmouseout="changeColor('tr1','out')">
					<td>1</td>
					<td>张三</td>
					<td>22</td>
				</tr>
				<tr onmouseover="changeColor('tr2','over')" id="tr2" onmouseout="changeColor('tr2','out')">
					<td>2</td>
					<td>李四</td>
					<td>25</td>
				</tr>
				<tr onmouseover="changeColor('tr3','over')" id="tr3" onmouseout="changeColor('tr3','out')">
					<td>3</td>
					<td>王五</td>
					<td>27</td>
				</tr>
				<tr onmouseover="changeColor('tr4','over')" id="tr4" onmouseout="changeColor('tr4','out')">
					<td>4</td>
					<td>赵六</td>
					<td>29</td>
				</tr>
				<tr onmouseover="changeColor('tr5','over')" id="tr5" onmouseout="changeColor('tr5','out')">
					<td>5</td>
					<td>田七</td>
					<td>30</td>
				</tr>
				<tr onmouseover="changeColor('tr6','over')" id="tr6" onmouseout="changeColor('tr6','out')">
					<td>6</td>
					<td>汾九</td>
					<td>20</td>
				</tr>
			</tbody>
		</table>
	</body>
</html>
```
7. JS完成全选和全不选
复选框全选和全不选
```
		<script>
			function checkAll(){
				//1.获取编号前面的复选框
				var checkAllEle = document.getElementById("checkAll");
				//2.对编号前面复选框的状态进行判断
				if(checkAllEle.checked==true){
					//3.获取下面所有的复选框
					var checkOnes = document.getElementsByName("checkOne");
					//4.对获取的所有复选框进行遍历
					for(var i=0;i<checkOnes.length;i++){
						//5.拿到每一个复选框，并将其状态置为选中
						checkOnes[i].checked=true;
					}
				}else{
					//6.获取下面所有的复选框
					var checkOnes = document.getElementsByName("checkOne");
					//7.对获取的所有复选框进行遍历
					for(var i=0;i<checkOnes.length;i++){
						//8.拿到每一个复选框，并将其状态置为未选中
						checkOnes[i].checked=false;
					}
				}
			}
		</script>
```
动态添加城市
```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>动态添加城市</title>
		<script>
			window.onload = function(){
				document.getElementById("btn").onclick = function(){
					//1.获取ul元素节点
					var ulEle = document.getElementById("ul1");
					//2.创建城市文本节点
					var textNode = document.createTextNode("深圳");//深圳
					//3.创建li元素节点
					var liEle = document.createElement("li");//<li></li>
					//4.将城市文本节点添加到li元素节点中去
					liEle.appendChild(textNode);//<li>深圳</li>
					//5.将li添加到ul中去
					ulEle.appendChild(liEle);
				}
			}
		</script>		
	</head>
	<body>
		<input type="button" value="添加新城市" id="btn"/>
		<ul id="ul1">
			<li>北京</li>
			<li>上海</li>
			<li>广州</li>
		</ul>
	</body>
</html>
```
8. JS完成省市二级联动
省市二级联动
```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>省市二级联动</title>
		<script>
			//1.创建一个二维数组用于存储省份和城市
			var cities = new Array(3);
			cities[0] = new Array("武汉市","黄冈市","襄阳市","荆州市");
			cities[1] = new Array("长沙市","郴州市","株洲市","岳阳市");
			cities[2] = new Array("石家庄市","邯郸市","廊坊市","保定市");
			cities[3] = new Array("郑州市","洛阳市","开封市","安阳市");
			function changeCity(val){
				//7.获取第二个下拉列表
				var cityEle = document.getElementById("city");
				//9.清空第二个下拉列表的option内容
				cityEle.options.length=0;
				//2.遍历二维数组中的省份
				for(var i=0;i<cities.length;i++){
					//注意，比较的是角标
					if(val==i){
						//3.遍历用户选择的省份下的城市
						for(var j=0;j<cities[i].length;j++){
							//4.创建城市的文本节点
							var textNode = document.createTextNode(cities[i][j]);
							//5.创建option元素节点
							var opEle = document.createElement("option");
							//6.将城市的文本节点添加到option元素节点
							opEle.appendChild(textNode);
							//8.将option元素节点添加到第二个下拉列表中去
							cityEle.appendChild(opEle);
						}
					}
				}
			}
		</script>
	</head>
	<body>
		<select onchange="changeCity(this.value)">
			<option>--请选择--</option>
			<option value="0">湖北</option>
			<option value="1">湖南</option>
			<option value="2">河北</option>
			<option value="3">河南</option>
		</select>
		<select id="city">
		</select>
	</body>
</html>
```
```
			var str = "-a-b-c-d-e-f-";
                        //抽取从 start 下标开始的指定数目的字符
			var str1 = str.substr(2,4);//-b-c
                        //和java一样，包括起点不包括终点
			var str2 = str.substring(2,4);//-b
			var str3 = "alert('abc')";
			//eval() 函数可计算某个字符串，并执行其中的的 JavaScript 代码。
			eval(str3);
```

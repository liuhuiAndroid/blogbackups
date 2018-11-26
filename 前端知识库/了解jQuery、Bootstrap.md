###了解jQuery
1. JQ入门
JQ页面加载
```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>JS与JQ页面加载区别</title>
		<script type="text/javascript" src="../../js/jquery-1.8.3.js" ></script>
		<script>
			window.onload = function(){
				alert("张三");
			}

			jQuery(document).ready(function(){
				alert("李四");
			});
			
            $(document).ready(function(){
                alert("王五");
            });

            //简写
            $(function(){
                alert("汾九");
            });

		</script>
	</head>
	<body>
	</body>
</html>
```
JQ获取元素
```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>JQ的获取</title>
		<script type="text/javascript" src="../../js/jquery-1.8.3.js" ></script>
		<script>
			$(function(){
				//1.JS方式获取
				document.getElementById("btn").onclick= function(){
					location.href="惊喜.html";
				}
				//2.JQ方式获取=====>>>$("#btn")
				$("#btn").click(function(){
					location.href="惊喜.html"
				});
			});
		</script>
	</head>
	<body>
		<input type="button" value="点我有惊喜" id="btn"/>
	</body>
</html>
```
2. DOM对象和JQ对象的转换
```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>Dom与JQ对象之间的转换</title>
		<script type="text/javascript" src="../../js/jquery-1.8.3.js" ></script>
		<script>
			function write1(){
                //1.JS的DOM操作
                document.getElementById("span1").innerHTML="萌萌哒！";
                //DOM对象无法操作JQ对象里面属性和方法
                // document.getElementById("span1").html("萌萌哒！");这样写是错误的
				// var spanEle = document.getElementById("span1");
				//将DOM对象转换成JQ对象
				// $(spanEle).html("思密达");
			}
            $(function(){
                $("#btn").click(function(){
                    // JQ对象无法操作JS里面的属性和方法！！！这样写是错误的
                    // $("#span1").innerHTML="呵呵哒！"  这样写是错误的
                    $("#span1").html("呵呵哒！");  //应该这样写
                    //JQ对象向DOM对象转换方式一
                    // $("#span1").get(0).innerHTML="美美哒！";
                    //JQ对象向DOM对象转换方式二
                    // $("#span1")[0].innerHTML="棒棒哒！";
                });
            });
		</script>
	</head>
	<body>
		<input type="button" value="JS写入" onclick="write1()"/>
		<input type="button" value="JQ写入" id="btn"/><br />
		班长：<span id="span1">你好帅！</span>
	</body>
</html>
```
3. JQ选择器
基本选择器
```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>基本选择器</title>
		<link rel="stylesheet" href="../../css/style.css" />
		<script type="text/javascript" src="../../js/jquery-1.8.3.js" ></script>
		<script>
			$(function(){
				$("#btn1").click(function(){
					$("#one").css("background-color","pink");
				});
				$("#btn2").click(function(){
					$(".mini").css("background-color","pink");
				});
				$("#btn3").click(function(){
					$("div").css("background-color","pink");
				});
				$("#btn4").click(function(){
					$("*").css("background-color","pink");
				});
				$("#btn5").click(function(){
					$("#two,.mini").css("background-color","pink");
				});
			});
		</script>
	</head>
	<body>
		<input type="button" id="btn1" value="选择为one的元素"/>
		<input type="button" id="btn2" value="选择样式为mini的元素"/>
		<input type="button" id="btn3" value="选择所有的div元素"/>
		<input type="button" id="btn4" value="选择所有元素"/>
		<input type="button" id="btn5" value="选择id为two和样式为mini的元素"/>
		<hr/>
		<div id="one">
			<div class="mini">
				111
			</div>
		</div>
		<div id="two">
			<div class="mini">
				222
			</div>
			<div class="mini">
				333
			</div>
		</div>
		<div id="three">
			<div class="mini">
				444
			</div>
			<div class="mini">
				555
			</div>
			<div class="mini">
				666
			</div>
		</div>
		<span id="four">
		</span>
	</body>
</html>
```
层级选择器
```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>层级选择器</title>
		<link rel="stylesheet" type="text/css" href="../../css/style.css"/>
		<script type="text/javascript" src="../../js/jquery-1.8.3.js" ></script>
		<script type="text/javascript">
			$(function(){
				$("#btn1").click(function(){
					$("body div").css("background-color","gold");
				});
				
				$("#btn2").click(function(){
					$("body>div").css("background-color","gold");
				});
				
				$("#btn3").click(function(){
					$("#two+div").css("background-color","gold");
				});
				
				$("#btn4").click(function(){
					$("#one~div").css("background-color","gold");
				});
			});
		</script>
	</head>
	<body>
		<input type="button" id="btn1" value="选择body中的所有的div元素"/>
		<input type="button" id="btn2" value="选择body中的第一级的孩子"/>
		<input type="button" id="btn3" value="选择id为two的元素的下一个元素"/>
		<input type="button" id="btn4" value="选择id为one的所有的兄弟元素"/>
		<hr/>
		<div id="one">
			<div class="mini">
				111
			</div>
		</div>
		
		<div id="two">
			<div class="mini">
				222
			</div>
			<div class="mini">
				333
			</div>
		</div>
		
		<div id="three">
			<div class="mini">
				444
			</div>
			<div class="mini">
				555
			</div>
			<div class="mini">
				666
			</div>
		</div>
		<span id="four">
		</span>
	</body>
</html>
```
基本过滤选择器
```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>基本过滤选择器</title>
		<link rel="stylesheet" href="../../css/style.css" type="text/css"/>
		<script type="text/javascript" src="../../js/jquery-1.8.3.js" ></script>
		<script>
			$(function(){
				$("#btn1").click(function(){
					$("body div:first").css("background-color","red");
				});
				$("#btn2").click(function(){
					$("body div:last").css("background-color","red");
				});
				$("#btn3").click(function(){
					$("body div:odd").css("background-color","red");
				});
				$("#btn4").click(function(){
					$("body div:even").css("background-color","red");
				});
			});
		</script>
	</head>
	<body>
		<input type="button" id="btn1" value="body中的第一个div元素"/>
		<input type="button" id="btn2" value="body中的最后一个div元素"/>
		<input type="button" id="btn3" value="选择body中的奇数的div"/>
		<input type="button" id="btn4" value="选择body中的偶数的div"/>
		<hr/>
		<div id="one">
			<div class="mini">
				111
			</div>
		</div>
		<div id="two">
			<div class="mini">
				222
			</div>
			<div class="mini">
				333
			</div>
		</div>
		<div id="three">
			<div class="mini">
				444
			</div>
			<div class="mini">
				555
			</div>
			<div class="mini">
				666
			</div>
		</div>
		<span id="four">
		</span>
	</body>
</html>
```
属性选择器
```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>层级选择器</title>
		<link rel="stylesheet" href="../../css/style.css" />
		<script type="text/javascript" src="../../js/jquery-1.8.3.js" ></script>
		<script>
			$(function(){
				$("#btn1").click(function(){
					$("div[id]").css("background-color","red");
				});
				$("#btn2").click(function(){
					$("div[id='two']").css("background-color","red");
				});
			});
		</script>
	</head>
	<body>
		<input type="button" id="btn1" value="选择有id属性的div"/>
		<input type="button" id="btn2" value="选择有id属性的值为two的div"/>
		<hr/>
		<div id="one">
			<div class="mini">
				111
			</div>
		</div>
		<div id="two">
			<div class="mini">
				222
			</div>
			<div class="mini">
				333
			</div>
		</div>
		<div id="three">
			<div class="mini">
				444
			</div>
			<div class="mini">
				555
			</div>
			<div class="mini">
				666
			</div>
		</div>
		<span id="four">
		</span>
	</body>
</html>
```
表单选择器
```
```
4. JQ完成定时弹出广告
使用toggle完成显示隐藏
```
$("#img1").toggle();
```
```
<script>
	$(function(){
		//1.书写显示广告图片的定时操作
		time = setInterval("showAd()",3000);
	});
	//2.书写显示广告图片的函数
	function showAd(){
		//3.获取广告图片，并让其显示
		//$("#img2").show(1000);
		//$("#img2").slideDown(5000);
		$("#img2").fadeIn(4000);
		//4.清除显示图片定时操作
		clearInterval(time);
		//5.设置隐藏图片的定时操作
		time = setInterval("hiddenAd()",3000);
	}
	function hiddenAd(){
		//6.获取广告图片，并让其隐藏
		//$("#img2").hide();
		//$("#img2").slideUp(5000);
		$("#img2").fadeOut(6000);
		//7.清除隐藏图片的定时操作
		clearInterval(time);
	}
</script>
```
5. JQ完成表格隔行换色
```
.even		{ background:#FFF38F;}  /* 偶数行样式*/
.odd		{ background:#FFFFEE;}  /* 奇数行样式*/

<script>
	//1.页面加载
	$(function(){
		/*//2.获取tbody下面的偶数行并设置背景颜色
		$("tbody tr:even").css("background-color","yellow");
		//3.获取tbody下面的奇数行并设置背景颜色
		$("tbody tr:odd").css("background-color","green");*/
		//2.获取tbody下面的偶数行并设置背景颜色
		$("tbody tr:even").addClass("even");
		$("tbody tr:even").removeClass("even");
		//3.获取tbody下面的奇数行并设置背景颜色
		$("tbody tr:odd").addClass("odd");
	});
</script>
```
6. JQ完成全选和全不选
```
<script>
	$(function(){
		$("#select").click(function(){
			//获取下面所有的 复选框并将其选中状态设置跟编码的前端 复选框保持一致。
			//attr方法与JQ的版本有关，在1.8.3及以下有效。
			//$("tbody input").attr("checked",this.checked);
			$("tbody input").prop("checked",this.checked);
		});
	});
</script>
```
7. JQ完成省市二级联动
```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>使用jQuery完成省市二级联动</title>
		<script type="text/javascript" src="../js/jquery-1.8.3.js" ></script>
		<script>
			$(function(){
				//2.创建二维数组用于存储省份和城市
				var cities = new Array(3);
				cities[0] = new Array("武汉市","黄冈市","襄阳市","荆州市");
				cities[1] = new Array("长沙市","郴州市","株洲市","岳阳市");
				cities[2] = new Array("石家庄市","邯郸市","廊坊市","保定市");
				cities[3] = new Array("郑州市","洛阳市","开封市","安阳市");
				$("#province").change(function(){
					//10.清除第二个下拉列表的内容
					$("#city").empty();
					//1.获取用户选择的省份
					var val = this.value;
					//alert(val);
					//3.遍历二维数组中的省份
					$.each(cities,function(i,n){
						//alert(i+":"+n);
						//4.判断用户选择的省份和遍历的省份
						if(val==i){
							//5.遍历该省份下的所有城市
							$.each(cities[i], function(j,m) {
								//alert(m);
								//6.创建城市文本节点
								var textNode = document.createTextNode(m);
								//7.创建option元素节点
								var opEle = document.createElement("option");
								//8.将城市文本节点添加到option元素节点中去
								$(opEle).append(textNode);
								//9.将option元素节点追加到第二个下拉列表中去
								$(opEle).appendTo($("#city"));
							});
						}
					});
				});
			});
		</script>
	</head>
	<body>
		<!--2.确定事件，通过函数传参的方式拿到改变后的城市-->
		<select id="province">
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
8. JQ完成下拉列表左右选择
```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>下拉列表左右选择</title>
		<script type="text/javascript" src="../js/jquery-1.8.3.js" ></script>
		<script>
			$(function(){
				/*1.选中单击去右边*/
				$("#selectOneToRight").click(function(){
					$("#left option:selected").appendTo($("#right"));
				});
				/*2.单击全部去右边*/
				$("#selectAllToRight").click(function(){
					$("#left option").appendTo($("#right"));
				});
				/*3.选中双击去右边*/
				$("#left option").dblclick(function(){
					$("#left option:selected").appendTo($("#right"));
				});
			});
		</script>
	</head>
	<body>
		<table border="1" width="600" align="center">
			<tr>
				<td>
					<span style="float: left;">
						<font color="green" face="宋体">已有商品</font><br/>
						<select multiple="multiple" style="width: 100px;height: 200px;" id="left">
							<option>IPhone6s</option>
							<option>小米4</option>
							<option>锤子T2</option>
						</select>
						<p><a href="#" style="padding-left: 20px;" id="selectOneToRight">&gt;&gt;</a></p>
						<p><a href="#" style="padding-left: 20px;" id="selectAllToRight">&gt;&gt;&gt;</a></p>
					</span>
					<span style="float: right;">
						<font color="red" face="宋体">未有商品</font><br/>
						<select multiple="multiple" style="width: 100px;height: 200px;" id="right">
							<option>三星Note3</option>
							<option>华为6s</option>
						</select>
						<p><a href="#" >&lt;&lt;</a></p>
						<p><a href="#" >&lt;&lt;&lt;</a></p>
					</span>
				</td>
			</tr>
		</table>
	</body>
</html>
```
9. validate入门案例
```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>validate入门案例</title>
		<script type="text/javascript" src="../../js/jquery-1.8.3.js" ></script>
		<!--validate.js是建立在jquery之上的，所以得先导入jquery的类库-->
		<script type="text/javascript" src="../../js/jquery.validate.min.js" ></script>
		<!--引入国际化js文件-->
		<script type="text/javascript" src="../../js/messages_zh.js" ></script>
		<style>
            label.error{
                background:url(../../img/unchecked.gif) no-repeat 10px 3px;
                padding-left: 30px;
                font-family:georgia;
                font-size: 15px;
                font-style: normal;
                color: red;
            }
            label.success{
                background:url(../../img/checked.gif) no-repeat 10px 3px;
                padding-left: 30px;
            }
		</style>
		<script>
			$(function(){
				$("#checkForm").validate({
					rules:{
						username:{
							required:true,
							minlength:6
						},
						password:{
							required:true,
							digits:true,
							minlength:6
						}
					},
					messages:{
						username:{
							required:"用户名不能为空!",
							minlength:"用户名不得少于6位！"
						},
						password:{
							required:"密码不能为空!",
							digits:"密码必须是整数!",
							minlength:"密码不得少于6位!"
						}
					},
                    errorElement: "label", //用来创建错误提示信息标签,validate插件默认的就是label
                    success: function(label) { //验证成功后的执行的回调函数
                        label.text(" ") //清空错误提示消息
                            .addClass("success"); //加上自定义的success类
                    }
				});
			});
		</script>
	</head>
	<body>
		<form action="#" id="checkForm">
			用户名：<input type="text" name="username" /><br />
			密码：<input type="password" name="password"/><br />
			<input type="submit"/>
		</form>
	</body>
</html>
```
###了解Bootstrap
1. Bootstrap是什么
>Bootstrap是Twitter推出的一个用于前端开发的开源工具包

[**Bootstrap Github地址**](https://github.com/twbs/bootstrap)
[**Bootstrap 中文网**](http://www.bootcss.com/)
[**Bootstrap4 教程**](http://www.runoob.com/bootstrap4/bootstrap4-tutorial.html)
[**Bootstrap 手册**](http://www.jqhtml.com/bootstraps-syntaxhigh/index.html)
2. Bootstrap例子
# **[Bootstrap-Learning-Demo](https://github.com/liuhuiAndroid/Bootstrap-Learning-Demo)**

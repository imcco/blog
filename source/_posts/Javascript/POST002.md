---
title: JS防止表单重复提交
tags: JavaScript
category: JavaScript
abbrlink: 53250
date: 2018-03-28 20:30:36
---
![image](http://ovi3ob9p4.bkt.clouddn.com/TIETU/CT0165.jpg)
JS防止表单重复提交
<!--more-->
第一种：

用flag标识，下面的代码设置checkSubmitFlg标志： 

```html
<script language="”javascript”"> 
var checkSubmitFlg = false; 
function checkSubmit(){ 
if(checkSubmitFlg ==true){ return false; //当表单被提交过一次后checkSubmitFlg将变为true,根据判断将无法进行提交。 
} 
checkSubmitFlg ==true; 
return true; 
} 
< /script > 
< form name=”form1” method=”post” onsubmit=”return checkSubmit();”> 
………..< /form> 
```

第二种：

在onsubmit事件中设置，在第一次提交后使提交按钮失效，代码如下： 

```html
<form action=”about:blank” method=”post” onsubmit="getElementById(‘submitInput').disabled=true;return true;" target=”_blank”> 
<input type=”submit” id=”submitInput”/> 
</form> 
	</body> 
</html> 
</script> 
```

因为程序源码跟WIN2000的注册表有冲突，帖子发出后会出现无效页面，以致于论坛里有很多无恶意的重复帖子，后来想出了一个办法，用JS避免重复提交，下面是部分源码： 

```html
<script Language='JavaScript'>
	function formsubmit() {
		Today = new Date();
		var NowHour = Today.getHours();
		var NowMinute = Today.getMinutes();
		var NowSecond = Today.getSeconds();
		var mysec = (NowHour * 3600) + (NowMinute * 60) + NowSecond;
		if ((mysec - document.formsubmitf.mypretime.value) > 600)
		//600只是一个时间值，就是5分钟内禁止重复提交，值随你高兴设 
		{
			document.formsubmitf.mypretime.value = mysec;
		} else {
			alert(' 按一次就够了，请勿重复提交！请耐心等待！谢谢合作！');
			return false;
		}
		document.forms.formsubmitf.submit();
	}
</script>

</HEAD>

<BODY BGCOLOR="#FFFFFF">
	<form name=formsubmitf id="the" method="post" action="XXX.asp">
		<input type=hidden name='mypretime' value='0'>
			//这句不能少，用隐含变量传递一个时间初值 //这里是你要提交的内容 <input type="button" value="写好了"
			name="button1" class="4round" onclick='formsubmit()'> <font
				class="red">(请按一次，耐心等待！)</font> <input type="reset" value="重 写"
				name="button2" class="4round">
	</form>
```

用了这个代码，论坛的重复帖子明显减少，不过有个缺点，就是刷新一次，检测就不起作用，好处就是利用JS检测，不需要额外的权限支持，至于效果如何，用不用就随你们了，（最好前端跟后端都加上检测）.

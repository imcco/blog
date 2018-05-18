---
title: JS特效-tab选项卡列表切换
tags: js
category: js
abbrlink: 59893
date: 2018-04-08 20:30:36
---
![image](http://ovi3ob9p4.bkt.clouddn.com/TIETU/CT0170.jpg)
tab选项卡列表切换
<!--more-->
#### index.html
```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>列表切换效果代码</title>
<link rel="stylesheet" href="css/tab.css" />
</head>
<body>

    <div id="tab">
        <h3 class="up" id="two1" onmouseover="setContentTab('two',1,4)">热点新闻</h3>
        <div class="block" id="con_two_1">
            <ul>
                <li><a class="tab_menu" href="#">[新闻热点]</a><a class="tab_title" href="#">点点妙：欢庆“十一”送祝福</a><span>2014-09-05</span></li>
                <li><a class="tab_menu" href="#">[新闻热点]</a><a class="tab_title" href="#">点点妙携手移动端APP 开创棋牌游戏新模</a><span>2014-09-05</span></li>
                <li><a class="tab_menu" href="#">[新闻热点]</a><a class="tab_title" href="#">看棋牌大亨如何颠覆快播江湖</a><span>2014-09-05</span></li>
                <li><a class="tab_menu" href="#">[新闻热点]</a><a class="tab_title" href="#">棋牌行业：解密点点妙合作内幕 分成竟高达</a><span>2014-09-05</span></li>
            </ul>
        </div>
        <h3 id="two2" onmouseover="setContentTab('two',2,4)">合作播报</h3>
        <div id="con_two_2">
            <ul>
                <li><a class="tab_menu" href="#">[合作播报]</a><a class="tab_title" href="#">收银妹代理棋牌游戏 成功晋级游戏女老板2</a><span>2014-09-05</span></li>
                <li><a class="tab_menu" href="#">[合作播报]</a><a class="tab_title" href="#">收银妹代理棋牌游戏 成功晋级游戏女老板1</a><span>2014-09-05</span></li>
                <li><a class="tab_menu" href="#">[合作播报]</a><a class="tab_title" href="#">IT精英放弃高薪只因想“玩”游戏</a><span>2014-09-05</span></li>
                <li><a class="tab_menu" href="#">[合作播报]</a><a class="tab_title" href="#">“干爹”扶持，乐猴棋牌开疆拓土</a><span>2014-09-05</span></li>
            </ul>
        </div>
        <h3 id="two3" onmouseover="setContentTab('two',3,4)">行业咨询</h3>
        <div id="con_two_3">
            <ul>
                <li><a class="tab_menu" href="#">[行业咨询]</a><a class="tab_title" href="#">收银妹代理棋牌游戏 成功晋级游戏女老板3</a><span>2014-09-05</span></li>
                <li><a class="tab_menu" href="#">[行业咨询]</a><a class="tab_title" href="#">紫金阁：代理棋牌游戏如何能稳赚不亏？</a><span>2014-09-05</span></li>
                <li><a class="tab_menu" href="#">[行业咨询]</a><a class="tab_title" href="#">你知道吗？“紫金阁”竟然升级做了“小三”</a><span>2014-09-05</span></li>
                <li><a class="tab_menu" href="#">[行业咨询]</a><a class="tab_title" href="#">都市娱乐赚钱首选 棋牌游戏代理</a><span>2014-09-05</span></li>
            </ul>
        </div>
        <h3 id="two4" onmouseover="setContentTab('two',4,4)">运营攻略</h3>
        <div id="con_two_4">
            <ul>
                <li><a class="tab_menu" href="#">[运营攻略]</a><a class="tab_title" href="#">加盟网络棋牌游戏的注意事项</a><span>2014-09-05</span></li>
                <li><a class="tab_menu" href="#">[运营攻略]</a><a class="tab_title" href="#">奇葩：一款可以自动赚钱的棋牌游戏</a><span>2014-09-05</span></li>
                <li><a class="tab_menu" href="#">[运营攻略]</a><a class="tab_title" href="#">代理棋牌游戏 紫金阁支招稳赚不亏</a><span>2014-09-05</span></li>
                <li><a class="tab_menu" href="#">[运营攻略]</a><a class="tab_title" href="#">棋牌行业内幕：什么时候代理棋牌游戏最合适</a><span>2014-09-05</span></li>
            </ul>
        </div>
    </div>
	
<script type="text/javascript">
function setContentTab(name, curr, n) {
    for (i = 1; i <= n; i++) {
        var menu = document.getElementById(name + i);
        var cont = document.getElementById("con_" + name + "_" + i);
        menu.className = i == curr ? "up" : "";
        if (i == curr) {
            cont.style.display = "block";
        } else {
            cont.style.display = "none";
        }
    }
}
</script>
</body>
</html>
```

#### tab.css

```css
/* CSS Document */
#tab { width:412px; height:216px; position:relative;margin:40px auto 0 auto;}
/*设置容器高宽等*/
html > body #tab { width:412px;}
/*兼容IE6:IE6下宽度不够*/
#tab div { position:absolute; top:30px; left:0; width:412px; height:186px;}
/*设置容器高宽等*/
#tab div { display:none;}
/*设置容器默认隐藏:不用ID是因为下面将利用class来控制容器显示,而class优先级低于id选择器*/
#tab .block { display:block;}
/*选中的容器*/
#tab h3 { float:left; width:103px; height:30px; line-height:30px; margin:0 0 0 0; font-size:12px; cursor:pointer; background-color:#c5c5c5; text-align:center; color:#5a5a5a; font-family:Microsoft YaHei;font-weight:normal;}
/*默认标题样式*/
#tab .up { background:#0292cf;color:#fff;}
/*选中的标题样式*/
/*修饰列表内容*/
#tab ul { list-style:none; padding:0; height:186px; margin-top:0px;}
#tab li { margin-left:10px; margin-right:10px; border-bottom:1px dotted #c6c6c6; height:22px; padding-top:23px; overflow:hidden; font-size:12px;}
#tab li a { display:inline; font-size:12px; text-decoration:none; text-indent:10px; margin-right:10px;}
#tab li span{ display:block; float:right; margin-right:5px; color: #bdacb3;}
a.tab_title:link { color: #5a5a5a; text-decoration:none;}
a.tab_title:visited { color:#5a5a5a; text-decoration:none;}
a.tab_title:hover { color:#5a5a5a; text-decoration:none;}
a.tab_title:active { color:#5a5a5a; text-decoration:none;}
a.tab_menu:link { color:#6464d5; text-decoration:none;}
a.tab_menu:hover { color:#8888e0; text-decoration:underline;}
```

参考: http://www.lanrenzhijia.com/tab

​          https://www.cnblogs.com/cheryshi/p/7903918.html

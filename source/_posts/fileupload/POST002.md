---
title: WebUploader上传组件
tags: WebUploader
category: WebUploader
abbrlink: 41980
date: 2018-03-22 20:30:36
---
![image](http://ovi3ob9p4.bkt.clouddn.com/TIETU/CT0161.jpg)

大文件上传组件
<!--more-->
#### 基本用法

`$(function(){…});   jQuery(function($) {…});  $(document).ready(function(){…})`这三个的作用是一样的，本人比较需要用第一种、书写简单。文档载入完成后执行的函数。

```js
//一个div用来存放文件上传时的信息
//一个div用来存放上传相关的按钮
<link rel="stylesheet" type="text/css" href="./web-uploader/webuploader.css" />
<!--<script style="text/javascript" src="./jQuery/jquery-2.2.3.min.js"></script>-->
<script type="text/javascript" src="./web-uploader/webuploader.js"></script>
<div id="uploader" class="wu-example">
    <!--用来存放文件信息-->
    <div id="thelist" class="uploader-list"></div>
    <div class="btns">
        <div id="picker">选择文件</div>
        <button id="ctlBtn" class="btn btn-default">开始上传</button>
        <button id="goBack" class="btn btn-default">返回</button>
    </div>
</div>
/*
1、首先用WebUploader.create创建一个 WebUploader对象 ，并在create中添加自定义配置项
2、然后手动给WebUploader对象添加事件，用到的基本事件是 
	fileQueued 文件被添加进队列的时候，在thelist div 中显示文件信息
	uploadProgress 文件上传过程中创建进度条实时显示
	uploadSuccess
	uploadError 
	uploadComplete 在文件上传完后都会触发uploadComplete事件
3、最后 调用upload()方法实现上传，
*/
<script>
var uploader = WebUploader.create({

	// swf文件路径
	swf:  '/js/Uploader.swf',
	formData:{"dn":$("#requestDn").val()},//参数列表
	// 文件接收服务端。
	server: '/tp5/index/user/uploadFile',
	// 选择文件的按钮。可选。
	// 内部根据当前运行是创建，可能是input元素，也可能是flash.
	pick: '#picker',
	// 不压缩image, 默认如果是jpeg，文件上传前会压缩一把再上传！
	resize: false,
	// 只允许选择图片文件。
	accept: {
		title: 'file',
		extensions: 'cer'
//                mimeTypes: '.cer,'
	}
});
var $list = $("#thelist");
uploader.on( 'fileQueued', function( file ) {
	$list.append( '<div id="' + file.id + '" class="item">' +
			'<h4 class="info">' + file.name + '</h4>' +
			'<p class="state">等待上传...</p>' +
			'<p class="progress progress-bar">上传进度...</p>' +
			'</div>' );
});

uploader.on( 'uploadSuccess', function( file ) {
	$( '#'+file.id ).find('p.state').text('已上传');
});
// 文件上传过程中创建进度条实时显示。
uploader.on( 'uploadProgress', function( file, percentage ) {
	var $li = $( '#'+file.id ),
			$percent = $li.find('.progress .progress-bar');

	// 避免重复创建
	if ( !$percent.length ) {
		$percent = $('<div class="progress progress-striped active">' +
				'<div class="progress-bar" role="progressbar" style="width: 0%">' +
				'</div>' +
				'</div>').appendTo( $li ).find('.progress-bar');
	}

	$li.find('p.state').text('上传中');

	$percent.css( 'width', percentage * 100 + '%' );
});
uploader.on( 'uploadError', function( file ) {
	$( '#'+file.id ).find('p.state').text('上传出错');
});

uploader.on( 'uploadComplete', function( file ) {
	$( '#'+file.id ).find('.progress').fadeOut();
});

$("#ctlBtn").on('click', function() {
	uploader.upload();
});
$("#goBack").on('click', function() {
	$("#uploadFileDiv").empty();
	$("#uploadFile").removeClass("hidden");
});
</script>
```

#### 接口说明 

这里是简单介绍，具体接口参考 

webuploader接口文档地址

Web Uploader内部类的详细说明，以下提及的功能类，都可以在 WebUploader 这个变量中访问到。

也就是说下面提到的 Base类 、Mediator类 、file类 、Queue类 都可以直接用 WebUploader 创建的变量直接访问，

例如下面创建的 uploader 变量，就可以直接访问 Base类 的 uploader.browser.ie

//Demo中使用的是WebUploader.create方法来初始化的，实际上可直接访问WebUploader.Uploader

##### Uploader类 上传入口类

###### 参数说明

下面所有参数都是可选的，并且都有默认值

```js
	var uploader = WebUploader.Uploader({	
		//几个常用的参数：swf,pick,formData,runtimeOrder
		
		//所有参数列表
		swf: 'path_of_swf/Uploader.swf',
		dnd: '#dndArea', // [默认值：undefined] 指定Drag And Drop拖拽的容器，如果不指定，则不启动。
		disableGlobalDnd: true,, // [默认值：false] 是否禁掉整个页面的拖拽功能，如果不禁用，图片拖进来的时候会默认被浏览器打开
		paste: '#uploader', // [默认值：undefined] 指定监听paste事件的容器，如果不指定，不启用此功能。此功能为通过粘贴来添加截屏的图片。建议设置为document.body.
		pick:'#filePicker',//也可以用下面的方式详细配置
		// {Selector, Object}  [默认值：undefined] 指定选择文件的按钮容器，不指定则不创建按钮。
		pick: {
			id: '#filePicker',//Seletor|dom 指定选择文件的按钮容器，不指定则不创建按钮。注意 这里虽然写的是 id, 但是不是只支持 id, 还支持 class, 或者 dom 节点。
			label: '点击选择图片',//请采用 innerHTML 代替
			innerHTML: "点击选择图片",// 指定按钮文字。不指定时优先从指定的容器中看是否自带文字。
			multiple:true //是否开起同时选择多个文件能力。
		},
		//限制上传的文件类型
		accept: {
			title: 'Images',// {String} 文字描述
			extensions: 'gif,jpg,jpeg,bmp,png,rar',// {String} 允许的文件后缀，不带点，多个用逗号分割。
			mimeTypes: 'image/gif，image/jpg，image/jpeg，image/bmp，image/png，.rar'// 多个用逗号分割。
		},
		// 设置缩略图。
		thumb: {
			width: 110,
			height: 110,
			// 图片质量，只有type为`image/jpeg`的时候才有效。
			quality: 70,
			// 是否允许放大，如果想要生成小图的时候不失真，此选项应该设置为false.
			allowMagnify: true,
			// 是否允许裁剪。是否采用裁剪模式。如果采用这样可以避免空白内容。
			crop: true,
			// 为空的话则保留原有图片格式。
			// 否则强制转换成指定的类型。
			type: 'image/jpeg'
		},
		// 配置压缩的图片的选项。如果此选项为false, 则图片在上传前不进行压缩。
		compress: {
			width: 1600,
			height: 1600,
			// 图片质量，只有type为`image/jpeg`的时候才有效。
			quality: 90,
			// 是否允许放大，如果想要生成小图的时候不失真，此选项应该设置为false.
			allowMagnify: false,
			// 是否允许裁剪。
			crop: false,
			// 是否保留头部meta信息。
			preserveHeaders: true,
			// 如果发现压缩后文件大小比原来还大，则使用原来图片
			// 此属性可能会影响图片自动纠正功能
			noCompressIfLarger: false,
			// 单位字节，如果图片大小小于此值，不会采用压缩。
			compressSize: 0
		}, 


		auto: true, // [默认值：false] 设置为 true 后，不需要手动调用上传，有文件选择即开始上传。
		runtimeOrder: 'flash', // [默认值：html5,flash] 指定运行时启动顺序。默认会想尝试 html5 是否支持，如果支持则使用 html5, 否则则使用 flash.可以将此值设置成 flash，来强制使用 flash 运行时。
		prepareNextFile:false, // [默认值：false] 是否允许在文件传输时提前把下一个文件准备好。 对于一个文件的准备工作比较耗时，比如图片压缩，md5序列化。 如果能提前在当前文件传输期处理，可以节省总体耗时。
		chunked:false, // [默认值：false] 是否要分片处理大文件上传。
		chunkSize: 512 * 1024,// [默认值：5242880] 如果要分片，分多大一片？ 默认大小为5M.
		chunkRetry:2, // [默认值：2] 如果某个分片由于网络问题出错，允许自动重传多少次？
		threads:3, // [默认值：3] 上传并发数。允许同时最大上传进程数。
		formData: {"data":"value","data":"value"}, // [默认值：{}] 文件上传请求的参数表，每次发送都会发送此对象中的参数。
		fileVal:"file", // [默认值：'file'] 设置文件上传域的name。
		method :"POST", // [默认值：'POST'] 文件上传方式，POST或者GET。
		sendAsBinary :false, // [默认值：false] 是否已二进制的流的方式发送文件，这样整个上传内容php://input都为文件内容， 其他参数在$_GET数组中。
		fileNumLimit :10, // [默认值：undefined] 验证文件总数量, 超出则不允许加入队列。
		fileSizeLimit : 200 * 1024 * 1024,    // 200 M  [默认值：undefined] 验证文件总大小是否超出限制, 超出则不允许加入队列。
		fileSingleSizeLimit: 50 * 1024 * 1024,    // 50 M [默认值：undefined] 验证单个文件大小是否超出限制, 超出则不允许加入队列。
		duplicate :true, // [默认值：undefined] 去重， 根据文件名字、文件大小和最后修改时间来生成hash Key.
		disableWidgets: {String, Array}, // [默认值：undefined] 默认所有 Uploader.register 了的 widget 都会被加载，如果禁用某一部分，请通过此 option 指定黑名单。
	});
```

###### uploader对象的选项

```js
	1、option() //获取或者设置Uploader配置项。
	// 修改后图片上传前，尝试将图片压缩到1600 * 1600
	uploader.option( 'compress', {
		width: 1600,
		height: 1600
	});	
	2、getStats() //获取文件统计信息。返回一个包含一下信息的对象。
	//successNum 上传成功的文件数
	//progressNum 上传中的文件数
	//cancelNum 被删除的文件数
	//invalidNum 无效的文件数
	//uploadFailNum 上传失败的文件数
	//queueNum 还在队列中的文件数
	//interruptNum 被暂停的文件数
	stats = uploader.getStats();
	if ( stats.successNum && !stats.uploadFailNum ) {
		setState( 'finish' );
		return;
	}
	3、destroy() //销毁 webuploader 实例
	4、addButton() //添加文件选择按钮，如果一个按钮不够，需要调用此方法来添加。参数跟options.pick一致。
		uploader.addButton({
			id: '#filePicker2',
			label: '继续添加'
		});
	5、makeThumb() //生成缩略图，此过程为异步，所以需要传入callback。 通常情况在图片加入队里后调用此方法来生成预览图以增强交互效果。
				//当 width 或者 height 的值介于 0 - 1 时，被当成百分比使用。
				//callback中可以接收到两个参数。
				//第一个为error，如果生成缩略图有错误，此error将为真。
				//第二个为ret, 缩略图的Data URL值。
				//注意 Date URL在IE6/7中不支持，所以不用调用此方法了，直接显示一张暂不支持预览图片好了。 也可以借助服务端，将 base64 数据传给服务端，生成一个临时文件供预览。
		uploader.makeThumb( file, function( error, src ) {
		var img;

		if ( error ) {
			$wrap.text( '不能预览' );
			return;
		}

		if( isSupportBase64 ) {
			img = $('<img src="'+src+'">');
			$wrap.empty().append( img );
		} else {
			$.ajax('../../server/preview.php', {
				method: 'POST',
				data: src,
				dataType:'json'
			}).done(function( response ) {
				if (response.result) {
					img = $('<img src="'+response.result+'">');
					$wrap.empty().append( img );
				} else {
					$wrap.text("预览出错");
				}
			});
		}
	}, thumbnailWidth, thumbnailHeight );
	
	6、md5File()// 计算文件 md5 值，返回一个 promise 对象，可以监听 progress 进度。
	7、addFiles()  //添加文件到队列
	8、removeFile()  //移除某一文件, 默认只会标记文件状态为已取消，如果第二个参数为 true 则会从 queue 中移除
	9、getFiles()  //返回指定状态的文件集合，不传参数将返回所有状态的文件。
	10、retry() //重试上传，重试指定文件，或者从出错的文件开始重新上传。
	11、sort() //排序队列中的文件，在上传之前调整可以控制上传顺序。
	12、reset() //重置uploader。目前只重置了队列。
	13、predictRuntimeType() //预测Uploader将采用哪个Runtime
	14、upload() //开始上传。此方法可以从初始状态调用开始上传流程，也可以从暂停状态调用，继续上传流程。可以指定开始某一个文件
	15、stop() //暂停上传。第一个参数为是否中断上传当前正在上传的文件。如果第一个参数是文件，则只暂停指定文件。
	16、cancelFile() //标记文件状态为已取消, 同时将中断文件传输。
	17、isInProgress() //判断Uplaoder是否正在上传中。
	18、skipFile() //掉过一个文件上传，直接标记指定文件为已上传状态。
	19、request() //发送命令。当传入callback或者handler中返回promise时。返回一个当所有handler中的promise都完成后完成的新promise。
	
	20、Uploader.register() //添加组件
	21、Uploader.unRegister() //删除插件，只有在注册时指定了名字的才能被删除。
```

###### 事件说明

```js
dndAccept :// 阻止,此事件可以拒绝某些类型的文件拖入进来。目前只有 chrome 提供这样的 API，且只能通过 mime-type 验证。
beforeFileQueued :// 当文件被加入队列之前触发，此事件的handler返回值为false，则此文件不会被添加进入队列。
fileQueued :// 当文件被加入队列以后触发。
filesQueued :// 当一批文件添加进队列以后触发。
fileDequeued :// 当文件被移除队列后触发。
reset :// 当 uploader 被重置的时候触发。
startUpload :// 当开始上传流程时触发。
stopUpload :// 当开始上传流程暂停时触发。
uploadFinished :// 当所有文件上传结束时触发。
uploadStart :// 某个文件开始上传前触发，一个文件只会触发一次。
uploadBeforeSend :// 当某个文件的分块在发送前触发，主要用来询问是否要添加附带参数，大文件在开起分片上传的前提下此事件可能会触发多次。
uploadAccept :// 当某个文件上传到服务端响应后，会派送此事件来询问服务端响应是否有效。如果此事件handler返回值为false, 则此文件将派送server类型的uploadError事件。
uploadProgress :// 上传过程中触发，携带上传进度。
uploadError :// 当文件上传出错时触发。
uploadSuccess :// 当文件上传成功时触发。
uploadComplete :// 不管成功或者失败，文件上传完成时触发。
error :// 当validate不通过时，会以派送错误事件的形式通知调用者。通过upload.on('error', handler)可以捕获到此类错误，目前有以下错误会在特定的情况下派送错来。
		//Q_EXCEED_NUM_LIMIT 在设置了fileNumLimit且尝试给uploader添加的文件数量超出这个值时派送。
		//Q_EXCEED_SIZE_LIMIT 在设置了Q_EXCEED_SIZE_LIMIT且尝试给uploader添加的文件总大小超出这个值时派送。
		//Q_TYPE_DENIED 当文件类型不满足时触发。。
/*Web Uploader内部类的详细说明，以下提及的功能类，都可以在`WebUploader`这个变量中访问到。 即 Base类 Mediator类  File类都可以在`WebUploader`这个变量中访问到*/ 
```

##### Base类 

基础类方法 WebUploader 基础类，提供一些简单常用的方法  WebUploader.browser.ie

```js
	create() //创建Uploader实例，等同于new Uploader( opts );
	version //当前版本号
	$//引用依赖的jQuery或者Zepto对象
	browser  //简单的浏览器检查结果
	os  android、ios
	inherits //实现类与类之间的继承
	noop  //一个不做任何事情的方法。可以用来赋值给默认的callback
	bindFn //返回一个新的方法，此方法将已指定的context来执行
	log //引用Console.log如果存在的话，否则引用一个空函数noop。
	slice //被uncurrythis的数组slice方法。 将用来将非数组对象转化成数组对象
	guid //生成唯一的ID
	formatSize //格式化文件大小, 输出成带单位的字符串
	Deferred //创建一个Deferred对象。 详细的Deferred用法说明，请参照jQuery的API文档。Deferred对象在钩子回掉函数中经常要用到，用来处理需要等待的异步操作。
	isPromise //判断传入的参数是否为一个 promise 对象。
	when //返回一个promise，此promise在所有传入的promise都完成了后完成
```

##### Mediator类 

事件处理类，可以独立使用，也可以扩展给对象使用 中介，它本身是个单例，但可以通过installTo方法，使任何对象具备事件行为。 主要目的是负责模块与模块之间的合作，降低耦合度

```js
on once off trigger installTo 
```

##### File类 

文件类  本类的一般在 UploadProgress 这些事件中的回调函数中变量使用比较多

```js
	name//文件名，包括扩展名（后缀）
	size//文件体积（字节）
	type//文件MIMETYPE类型，与文件类型的对应关系请参考http://t.cn/z8ZnFny
	lastModifiedDate//文件最后修改日期
	id//文件ID，每个对象具有唯一ID，与文件名无关
	ext//文件扩展名，通过文件名获取，例如test.png的扩展名为png
	statusText//状态文字说明。在不同的status语境下有不同的用途。
	setStatus//设置状态，状态变化时会触发change事件。
	setStatus( status[, statusText] );//参数:status {File.Status, String}文件状态值
	statusText //{String} [可选] [默认值: ''] 状态说明，常在error时使用，用http, abort,server等来标记是由于什么原因导致文件错误。
	File.Status//文件状态值，具体包括以下几种类型：
	inited //初始状态
	queued //已经进入队列, 等待上传
	progress //上传中
	complete //上传完成。
	error //上传出错，可重试
	interrupt //上传中断，可续传。
	invalid //文件不合格，不能重试上传。会自动从队列中移除。
	cancelled //文件被移除。
```

##### Queue 类 

文件队列, 用来存储各个状态中的文件

```js
	stats//统计文件数。
	numOfQueue //队列中的文件数。
	numOfSuccess //上传成功的文件数
	numOfCancel //被取消的文件数
	numOfProgress //正在上传中的文件数
	numOfUploadFailed //上传错误的文件数。
	numOfInvalid //无效的文件数。
	numofDeleted //被移除的文件数。
	append//将新文件加入对队列尾部
    prepend//将新文件加入对队列头部
    getFile//获取文件对象
    fetch//从队列中取出一个指定状态的文件。
    sort//对队列进行排序，能够控制文件上传顺序。
    getFiles//获取指定类型的文件列表, 列表中每一个成员为File对象。
    removeFile//在队列中删除文件。
```

​	github中的代码给的例子基本上可以实现想要的功能，如果有别的需求可以结合代码中的例子根据接口手册进行相应的修改。


​	Web Uploader的所有代码都在一个内部闭包中，对外暴露了唯一的一个变量WebUploader，所以完全不用担心此框架会与其他框架冲突。
Uploader实例具有Backbone同样的事件API：`on，off，once，trigger。`
如同Document Element中的onEvent一样，他的执行比on添加的handler的要晚。如果那些handler里面，有一个return false了，此onEvent里面是不会执行到的

```js
uploader.on( 'fileQueued', function( file ) {
    // do some things.
});
//或
uploader.onFileQueued = function( file ) {
    // do some things.
};
```
#### 断点上传实例

##### 准备：

Uploader.swf、webuploader.css、webuploader.js，其中Uploader.swf只在初始化webUploader时用到，其余两个文件在页面引用即可。下载地址：https://github.com/fex-team/webuploader/releases

##### Jsp代码：

```jsp
<!-- 断点续传   start-->
<!-- 隐藏域 实时保存上传进度 -->
<input id="jindutiao" type="hidden"/>
<div id="uploader" class="wu-example">
    <label class="text-right" style="font-weight:100;float:left;margin-left:15px;width:144px;margin-right:15px;">大文件：</label>
    <div class="btns">
    <div id="picker" class="webuploader-container">
        <div class="webuploader-pick">选择文件</div>
        <div id="rt_rt_1bchdejhrarjdvd11h41eoh1nt1" style="position: absolute; top: 0px; left: 0px; width: 88px; height: 35px; overflow: hidden; bottom: auto; right: auto;">
            <input id="file_bp" name="file" class="webuploader-element-invisible" type="file" />
            <label style="opacity: 0; width: 100%; height: 100%; display: block; cursor: pointer; background: rgb(255, 255, 255) none repeat scroll 0% 0%;"></label>
        </div>
    </div>
    <!-- 文件列表：选择文件后在该div显示 -->
    <div id="thelist" class="uploader-list list-group-item clearfix ng-hide" style="margin-left:160px;"></div>
    <label class="text-right" style="font-weight:100;float:left;margin-left:15px;width:144px;margin-right:15px;"></label>
    <button class="btn m-b-xs btn-sm btn-info btn-addon" id="startOrStopBtn" style="padding:7px 50px;margin-top:20px;">开始上传</button>
    </div>
</div>
<!-- 断点续传   end-->
```

##### js代码

```js
<script type="text/javascript">  
jQuery(function() {
    /*******************初始化参数*********************************/
    var $list = $('#thelist'),//文件列表
        $btn = $('#startOrStopBtn'),//开始上传按钮
        state = 'pending',//初始按钮状态
        uploader; //uploader对象
    var fileMd5;  //文件唯一标识
    
    /******************下面的参数是自定义的*************************/
    var fileName;//文件名称
    var oldJindu;//如果该文件之前上传过 已经上传的进度是多少
    var count=0;//当前正在上传的文件在数组中的下标，一次上传多个文件时使用
    var filesArr=new Array();//文件数组：每当有文件被添加进队列的时候 就push到数组中
    var map={};//key存储文件id，value存储该文件上传过的进度
    
    /***************************************************** 监听分块上传过程中的三个时间点 start ***********************************************************/
    WebUploader.Uploader.register({  
        "before-send-file":"beforeSendFile",//整个文件上传前
        "before-send":"beforeSend",  //每个分片上传前
        "after-send-file":"afterSendFile",  //分片上传完毕
    },
    {  
        //时间点1：所有分块进行上传之前调用此函数  
        beforeSendFile:function(file){
            var deferred = WebUploader.Deferred();  
            //1、计算文件的唯一标记fileMd5，用于断点续传  如果.md5File(file)方法里只写一个file参数则计算MD5值会很慢 所以加了后面的参数：10*1024*1024
            (new WebUploader.Uploader()).md5File(file,0,10*1024*1024).progress(function(percentage){
                $('#'+file.id ).find('p.state').text('正在读取文件信息...');
            })  
            .then(function(val){  
                $('#'+file.id ).find("p.state").text("成功获取文件信息...");  
                fileMd5=val;  
                //获取文件信息后进入下一步  
                deferred.resolve();  
            });  
            
            fileName=file.name; //为自定义参数文件名赋值
            return deferred.promise();  
        },  
        //时间点2：如果有分块上传，则每个分块上传之前调用此函数  
        beforeSend:function(block){
            var deferred = WebUploader.Deferred();  
            $.ajax({  
                type:"POST",  
                url:"${ctx}/testController/mergeOrCheckChunks.do?param=checkChunk",  //ajax验证每一个分片
                data:{  
                    fileName : fileName,
                    jindutiao:$("#jindutiao").val(),
                    fileMd5:fileMd5,  //文件唯一标记  
                    chunk:block.chunk,  //当前分块下标  
                    chunkSize:block.end-block.start//当前分块大小  
                },  
                cache: false,
                async: false,  // 与js同步
                timeout: 1000, //todo 超时的话，只能认为该分片未上传过
                dataType:"json",  
                success:function(response){  
                    if(response.ifExist){
                        //分块存在，跳过  
                        deferred.reject();  
                    }else{  
                        //分块不存在或不完整，重新发送该分块内容  
                        deferred.resolve();  
                    }  
                }  
            });  
                             
            this.owner.options.formData.fileMd5 = fileMd5;  
            deferred.resolve();  
            return deferred.promise();  
        },  
        //时间点3：所有分块上传成功后调用此函数  
        afterSendFile:function(){
            //如果分块上传成功，则通知后台合并分块  
            $.ajax({  
                type:"POST",  
                url:"${ctx}/testController/mergeOrCheckChunks.do?param=mergeChunks",  //ajax将所有片段合并成整体
                data:{  
                    fileName : fileName,
                    fileMd5:fileMd5,
                },  
                success:function(data){
                    count++; //每上传完成一个文件 count+1
                    if(count<=filesArr.length-1){
                        uploader.upload(filesArr[count].id);//上传文件列表中的下一个文件
                    }
                     //合并成功之后的操作
                }  
            });  
        }  
    });
    /***************************************************** 监听分块上传过程中的三个时间点 end **************************************************************/
    
    /************************************************************ 初始化WebUploader start ******************************************************************/
    uploader = WebUploader.create({
        auto:false,//选择文件后是否自动上传
        chunked: true,//开启分片上传
        chunkSize:10*1024*1024,// 如果要分片，分多大一片？默认大小为5M
        chunkRetry: 3,//如果某个分片由于网络问题出错，允许自动重传多少次
        threads: 3,//上传并发数。允许同时最大上传进程数[默认值：3]
        duplicate : false,//是否重复上传（同时选择多个一样的文件），true可以重复上传
        prepareNextFile: true,//上传当前分片时预处理下一分片
        swf: '${ctx}/resource/webuploader/Uploader.swf',// swf文件路径  
        server: '${ctx}/testController/fileSave.do',// 文件接收服务端
        fileSizeLimit:6*1024*1024*1024,//6G 验证文件总大小是否超出限制, 超出则不允许加入队列
        fileSingleSizeLimit:3*1024*1024*1024,  //3G 验证单个文件大小是否超出限制, 超出则不允许加入队列
        pick: {
                id: '#picker', //这个id是你要点击上传文件按钮的外层div的id
                multiple : false //是否可以批量上传，true可以同时选择多个文件
        },  
        resize: false,  //不压缩image, 默认如果是jpeg，文件上传前会先压缩再上传！
        accept: {  
                //允许上传的文件后缀，不带点，多个用逗号分割
            extensions: "txt,jpg,jpeg,bmp,png,zip,rar,war,pdf,cebx,doc,docx,ppt,pptx,xls,xlsx",  
            mimeTypes: '.txt,.jpg,.jpeg,.bmp,.png,.zip,.rar,.war,.pdf,.cebx,.doc,.docx,.ppt,.pptx,.xls,.xlsx',  
        }  
    });  
    /************************************************************ 初始化WebUploader end ********************************************************************/
    
    //当有文件被添加进队列的时候（点击上传文件按钮，弹出文件选择框，选择完文件点击确定后触发的事件）  
    uploader.on('fileQueued', function(file) {
        //限制单个文件的大小 超出了提示
        if(file.size>3*1024*1024*1024){
            alert("单个文件大小不能超过3G");
            return false;
        }
        
        /*************如果一次只能选择一个文件，再次选择替换前一个，就增加如下代码*******************************/
        //清空文件队列
        $list.html("");
        //清空文件数组
        filesArr=[];
        /*************如果一次只能选择一个文件，再次选择替换前一个，就增加以上代码*******************************/
        
        //将选择的文件添加进文件数组
        filesArr.push(file);

        $.ajax({  
            type:"POST",  
            url:"${ctx}/testController/selectProgressByFileName.do",  //先检查该文件是否上传过，如果上传过，上传进度是多少
            data:{  
                fileName : file.name  //文件名
            },  
            cache: false,
            async: false,  // 同步
            dataType:"json",  
            success:function(data){  
                //上传过
                if(data>0){
                    //上传过的进度的百分比
                            oldJindu=data/100;
                    //如果上传过 上传了多少
                    var jindutiaoStyle="width:"+data+"%";
                    $list.append( '<div id="' + file.id + '" class="item">' +
                        '<h4 class="info">' + file.name + '</h4>' +
                        '<p class="state">已上传'+data+'%</p>' +
                        '<a href="javascript:void(0);" class="btn btn-primary file_btn btnRemoveFile">删除</a>' +
                            '<div class="progress progress-striped active">' +
                      '<div class="progress-bar" role="progressbar" style="'+jindutiaoStyle+'">' +
                      '</div>' +
                    '</div>'+
                    '</div>' );
                    //将上传过的进度存入map集合
                    map[file.id]=oldJindu;
                }else{//没有上传过
                    $list.append( '<div id="' + file.id + '" class="item">' +
                        '<h4 class="info">' + file.name + '</h4>' +
                        '<p class="state">等待上传...</p>' +
                        '<a href="javascript:void(0);" class="btn btn-primary file_btn btnRemoveFile">删除</a>' +
                    '</div>' );
                }  
            }  
        });
        uploader.stop(true);
        //删除队列中的文件
        $(".btnRemoveFile").bind("click", function() {
            var fileItem = $(this).parent();
            uploader.removeFile($(fileItem).attr("id"), true);
            $(fileItem).fadeOut(function() {
                $(fileItem).remove();
            });
        
            //数组中的文件也要删除
            for(var i=0;i<filesArr.length;i++){
                if(filesArr[i].id==$(fileItem).attr("id")){
                    filesArr.splice(i,1);//i是要删除的元素在数组中的下标，1代表从下标位置开始连续删除一个元素
                }
            }
        });
    });  
         
    //文件上传过程中创建进度条实时显示
    uploader.on('uploadProgress', function(file, percentage) {
        var $li = $( '#'+file.id ),
        $percent = $li.find('.progress .progress-bar');
        //避免重复创建
        if (!$percent.length){
            $percent = $('<div class="progress progress-striped active">' +
              '<div class="progress-bar" role="progressbar" style="width: 0%">' +
              '</div>' +
            '</div>').appendTo( $li ).find('.progress-bar');
        }
        
        //将实时进度存入隐藏域
        $("#jindutiao").val(Math.round(percentage * 100));
        
        //根据fielId获得当前要上传的文件的进度
        var oldJinduValue = map[file.id];
        
        if(percentage<oldJinduValue && oldJinduValue!=1){
            $li.find('p.state').text('上传中'+Math.round(oldJinduValue * 100) + '%');
            $percent.css('width', oldJinduValue * 100 + '%');
        }else{
            $li.find('p.state').text('上传中'+Math.round(percentage * 100) + '%');
            $percent.css('width', percentage * 100 + '%');
        }
    });  
    
    //上传成功后执行的方法
    uploader.on('uploadSuccess', function( file ) {  
        //上传成功去掉进度条
        $('#'+file.id).find('.progress').fadeOut();
        //隐藏删除按钮
        $(".btnRemoveFile").hide();
        //隐藏上传按钮
        $("#startOrStopBtn").hide();
        $('#'+file.id).find('p.state').text('文件已上传成功，系统后台正在处理，请稍后...');  
    });  
    
    //上传出错后执行的方法
    uploader.on('uploadError', function( file ) {
        errorUpload=true;
        $btn.text('开始上传');
        uploader.stop(true);
        $('#'+file.id).find('p.state').text('上传出错，请检查网络连接');
    });  
      
    //文件上传成功失败都会走这个方法
    uploader.on('uploadComplete', function( file ) {
        
    });  
    
    uploader.on('all', function(type){
        if (type === 'startUpload'){
            state = 'uploading';
        }else if(type === 'stopUpload'){
            state = 'paused';
        }else if(type === 'uploadFinished'){
            state = 'done';
        }

        if (state === 'uploading'){
            $btn.text('暂停上传');
        } else {
            $btn.text('开始上传');
        }
    });

    //上传按钮的onclick时间
    $btn.on('click', function(){
        if (state === 'uploading'){
            uploader.stop(true);
        } else {
            //当前上传文件的文件名
            var currentFileName;
            //当前上传文件的文件id
            var currentFileId;
            //count=0 说明没开始传 默认从文件列表的第一个开始传
            if(count==0){
                currentFileName=filesArr[0].name;
                currentFileId=filesArr[0].id;
            }else{
                if(count<=filesArr.length-1){
                    currentFileName=filesArr[count].name;
                    currentFileId=filesArr[count].id;
                }
            }
            
            //先查询该文件是否上传过 如果上传过已经上传的进度是多少
            $.ajax({  
                type:"POST",  
                url:"${ctx}/testController/selectProgressByFileName.do",  
                data:{  
                    fileName : currentFileName//文件名
                },  
                cache: false,
                async: false,  // 同步
                dataType:"json",  
                success:function(data){  
                    //如果上传过 将进度存入map
                    if(data>0){
                        map[currentFileId]=data/100;
                    }
                    //执行上传
                    uploader.upload(currentFileId);
                }  
            });
        }
    });
});
</script>
```

##### Java代码

```java
//合并、验证分片方法
public void mergeOrCheckChunks(HttpServletRequest request, HttpServletResponse response) {
    String param = request.getParameter("param");  
    String fileName = request.getParameter("fileName");
    
    //当前登录用户信息
    SysUser sysUser = (SysUser)request.getSession().getAttribute("user");
    String newFilePath = sysUser.getUserId()+"_"+fileName;
    String savePath = request.getRealPath("/");
    //文件上传的临时文件保存在项目的temp文件夹下 定时删除
    savePath = new File(savePath) + "/upload/";
    if(param.equals("mergeChunks")){
        //合并文件  
        Jedis jedis =null;
        try {
            jedis =jedisPool.getResource();
            //读取目录里的所有文件  
            File f = new File(savePath+"/"+jedis.get("fileName_"+fileName));  
            File[] fileArray = f.listFiles(new FileFilter(){  
                //排除目录只要文件  
                @Override  
                public boolean accept(File pathname) {  
                    if(pathname.isDirectory()){  
                        return false;  
                    }  
                    return true;  
                }  
            });

            //转成集合，便于排序  
            List<File> fileList = new ArrayList<File>(Arrays.asList(fileArray));  
            Collections.sort(fileList,new Comparator<File>() {  
                @Override  
                public int compare(File o1, File o2) {  
                    if(Integer.parseInt(o1.getName()) < Integer.parseInt(o2.getName())){  
                        return -1;  
                    }  
                    return 1;  
                }  
            });  

            //截取文件名的后缀名
            //最后一个"."的位置
            int pointIndex=fileName.lastIndexOf(".");
            //后缀名
            String suffix=fileName.substring(pointIndex);
            //合并后的文件
            File outputFile = new File(savePath+"/"+jedis.get("fileName_"+fileName)+suffix);  
            //创建文件  
            try {
                outputFile.createNewFile();
            } catch (IOException e) {
                e.printStackTrace();
            }  
            //输出流  
            FileChannel outChnnel = new FileOutputStream(outputFile).getChannel();  
            //合并  
            FileChannel inChannel;  
            for(File file : fileList){  
                inChannel = new FileInputStream(file).getChannel();  
                try {
                    inChannel.transferTo(0, inChannel.size(), outChnnel);
                } catch (IOException e) {
                    e.printStackTrace();
                }  
                try {
                    inChannel.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }  
                //删除分片  
                file.delete();  
            }  
            try {
                outChnnel.close();
            } catch (IOException e) {
                e.printStackTrace();
            }  

            //清除文件夹  
            File tempFile = new File(savePath+"/"+jedis.get("fileName_"+fileName));  
            if(tempFile.isDirectory() && tempFile.exists()){  
                tempFile.delete();  
            }  

            Map<String, String> resultMap=new HashMap<>();
            //将文件的最后上传时间和生成的文件名返回
            resultMap.put("lastUploadTime", jedis.get("lastUploadTime_"+newFilePath));
            resultMap.put("pathFileName", jedis.get("fileName_"+fileName)+suffix);
            
            /****************清除redis中的相关信息**********************/
            //合并成功后删除redis中的进度信息
            jedis.del("jindutiao_"+newFilePath);
            //合并成功后删除redis中的最后上传时间，只存没上传完成的
            jedis.del("lastUploadTime_"+newFilePath);
            //合并成功后删除文件名称与该文件上传时生成的存储分片的临时文件夹的名称在redis中的信息  key：上传文件的真实名称   value：存储分片的临时文件夹名称（由上传文件的MD5值+时间戳组成）
            //如果下次再上传同名文件  redis中将存储新的临时文件夹名称  没有上传完成的还要保留在redis中 直到定时任务生效
            jedis.del("fileName_"+fileName);
            
            Gson gson=new Gson();
            String json=gson.toJson(resultMap);
            PrintWriterJsonUtils.printWriter(response, json);
        } catch (Exception e) {
            e.printStackTrace();
        }finally{
            jedisPool.returnResource(jedis);
        }
    }else if(param.equals("checkChunk")){  
        /*************************检查当前分块是否上传成功**********************************/  
        String fileMd5 = request.getParameter("fileMd5");  
        String chunk = request.getParameter("chunk");  
        String chunkSize = request.getParameter("chunkSize");  
        String jindutiao=request.getParameter("jindutiao");//文件上传的实时进度

        Jedis jedis =null;
        try {
            jedis =jedisPool.getResource();
            //将当前进度存入redis
            jedis.set("jindutiao_"+newFilePath, jindutiao);

            //将系统当前时间转换为字符串
            Date date=new Date();  
            SimpleDateFormat formatter=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");  
            String lastUploadTime=formatter.format(date);  
            //将最后上传时间以字符串形式存入redis
            jedis.set("lastUploadTime_"+newFilePath, lastUploadTime);

            //自定义文件名： 时间戳（13位）
            String tempFileName= String.valueOf(System.currentTimeMillis());
            if(jedis.get("fileName_"+fileName)==null || "".equals(jedis.get("fileName_"+fileName))){
                //将文件名与该文件上传时生成的存储分片的临时文件夹的名称存入redis
                //文件上传时生成的存储分片的临时文件夹的名称由MD5和时间戳组成
                jedis.set("fileName_"+fileName, fileMd5+tempFileName);
            }

            File checkFile = new File(savePath+"/"+jedis.get("fileName_"+fileName)+"/"+chunk);  

            response.setContentType("text/html;charset=utf-8");  
            //检查文件是否存在，且大小是否一致  
            if(checkFile.exists() && checkFile.length()==Integer.parseInt(chunkSize)){  
                //上传过  
                try {
                    response.getWriter().write("{\"ifExist\":1}");
                } catch (IOException e) {
                    e.printStackTrace();
                }  
            }else{  
                //没有上传过  
                try {
                    response.getWriter().write("{\"ifExist\":0}");
                } catch (IOException e) {
                    e.printStackTrace();
                }  
            }  
        } catch (Exception e) {
            e.printStackTrace();
        }finally{
            jedisPool.returnResource(jedis);
        }
    }
}


//保存上传分片
public void fileSave(HttpServletRequest request, HttpServletResponse response) {
    DiskFileItemFactory factory = new DiskFileItemFactory();  
    ServletFileUpload sfu = new ServletFileUpload(factory);  
    sfu.setHeaderEncoding("utf-8");  
        
        String savePath = request.getRealPath("/");
        savePath = new File(savePath) + "/upload/";  
          
        String fileMd5 = null;  
        String chunk = null;  
    String fileName=null;
        try {  
            List<FileItem> items = sfu.parseRequest(request);  
              
            for(FileItem item:items){
                //上传文件的真实名称
                fileName=item.getName();
                if(item.isFormField()){  
                    String fieldName = item.getFieldName();  
                    if(fieldName.equals("fileMd5")){  
                        try {
                fileMd5 = item.getString("utf-8");
            } catch (UnsupportedEncodingException e) {
                e.printStackTrace();
            }  
                    }  
                    if(fieldName.equals("chunk")){  
                        try {
                chunk = item.getString("utf-8");
            } catch (UnsupportedEncodingException e) {
                e.printStackTrace();
            }  
                    }  
                }else{  
            Jedis jedis =null;
            try {
                jedis =jedisPool.getResource();
                File file = new File(savePath+"/"+jedis.get("fileName_"+fileName));  
                if(!file.exists()){  
                    file.mkdir();  
                }  
                File chunkFile = new File(savePath+"/"+jedis.get("fileName_"+fileName)+"/"+chunk);
                FileUtils.copyInputStreamToFile(item.getInputStream(), chunkFile);  
            } catch (Exception e) {
                e.printStackTrace();
            }finally{
                jedisPool.returnResource(jedis);
            }
                }  
            }  
        } catch (FileUploadException e) {  
            e.printStackTrace();  
        }  
}


//当有文件添加进队列时 通过文件名查看该文件是否上传过 上传进度是多少
public String selectProgressByFileName(String fileName) {
    String jindutiao="";
    Jedis jedis =null;
    try {
        jedis =jedisPool.getResource();
        if(null!=fileName && !"".equals(fileName)){
            jindutiao=jedis.get("jindutiao_"+fileName);
        }
    }catch(Exception e){
        e.printStackTrace();
    }finally{
        jedisPool.returnResource(jedis);
    }
    return jindutiao;
}
```

**注：webUploader断点上传多个大文件时是按队列顺序上传的，即队列中的文件一个一个上传，前一个上传完成才会开始上传下一个，不能实现同时上传。**

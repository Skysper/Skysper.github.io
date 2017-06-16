---
layout: post
tId: 1706001
title: "使用Cropper进行图片剪裁上传"
date: 2017-06-16 18:00:00 +0800
categories: 前端,图片上传,cropper
codelang: html
desc: "最近使用cropper完成了项目中的图片上传功能，方便强大，是图片上传处理的一大利器"
---
在项目中，需要多上传的图片按照比例和尺寸进行裁剪，这类场景在一些CMS系统中是比较常见的，尤其是大部分的文章现在要适配PC、Mobile两种平台，文章的封面图等便需要按照尺寸做裁剪，以便于应用到不同的场景和平台上。

项目地址：<https://github.com/fengyuanchen/cropper>  
文档：<https://github.com/fengyuanchen/cropper/blob/master/README.md>  

在页面中引入cropper.css和copper.js

```
 <div class="cut-container" style="width: 750px;height: 400px;margin:20px 30px;border:dashed #cacaca 1px;">
 </div>

 <script>
 	$img.cropper({
        aspectRatio:picScale.width/picScale.height,
        autoCrop:false,
        //minCropBoxWidth:216,
        //minCropBoxHeight:144,
        zoomable:false,
        scalable:false,
        rotatable:false,
        //autoCropArea:0.01,
        ready:function(){
            $img.cropper('crop');
            $img.cropper('setData',{
                width:picScale.bWidth,
                height:picScale.bHeight
            });
        }
    });
    $img.on('cropmove',function(e){
        var data=$img.cropper('getData');
        if(data.width<picScale.width||data.height<picScale.height){
            e.preventDefault();
        }
    });
    $img.on('cropend',function(e){
        var data=$img.cropper('getData');
        if(data.width<picScale.width||data.height<picScale.height){
            $img.cropper('setData',{ width:picScale.width,
                height:picScale.height});
        }
    });
 </script>
```

这里有几个参数需要注意下：  
aspectRatio：设定截取的结果图片是宽高比例，例如16:9、3:4或者1:1等  
minCropBoxWidth和minCropBoxHeight：截取框的最小宽度和最小高度，说截取框是因为这个是相对于页面的，也就是真是的页面像素，而不是相对于图片的
zoomable、scalable、rotatable：这几个都是图片控制的属性，从字面也比较好理解  
autoCropArea：这里是一个比例值，设定默认情况下，截取的内容相对于图片的面积，0.8代表整个图片的80%都处于截取框内。  

因为autoCropArea的原因，当你上传不同尺寸的图片时，也为界面显示大小不一样，截取框的也大小不一。如果你需要一个视觉上固定大小的截取框，这种情况下，是做不到的。  

另外，minCropBoxWidth和minCropBoxHeight也是相对于页面的像素，所以你无法设定默认要截取最小像素，例如我们像需要截取个400x400素的图片，如果我们上传的是高像素的大图，因为页面级400像素，涵盖的图片实际上的像素，可能会远大于400像素。

针对于这种情况，我通过ready事件和cropmove、cropend事件进行了处理，代码中被注释的是最初无效的方式，将autoCrop设定为false，通过代码的方式进行截取框的设置，通过$img.cropper('setData',params)便可以直接设定截取框的宽高，令人惊喜的是这里的宽高是相对于图片的像素控制，设定的像素值就是最终截取后图片的像素。

这种情况很便利，但是为了实现最小尺寸或者最大尺寸限制，便需要注册两个额外的事件，在cropmove通过$img.cropper('getData')获取width和height信息，判断相关限制，如果超过，则通过preventDefault()取消事件响应，但是这种情况不太精准，无法准备模拟到临界状态，有可能一次move事件，就超过临界值的，这种情况下，也因为这个判断，无法恢复到临界状态以内，所以在通过cropend事件，对于超出临界状态的情况，强制设定到临界状态。

以上是截图处理部分，另外在后期上传的时候，可以通过form表单的方式进行（另外一种是通过将图片转化为Base64编码），这里通过$img.cropper("getCroppedCanvas")获取到canvas并调用其toBlob方法将数据转化为二进制用来构造FormData。

```
$img.cropper("getCroppedCanvas").toBlob(function(blob){
    var formData=new FormData();
    formData.append('files',blob,file.name);

    $.ajax({
        method:"post",
        url: urlConfig.upload, //用于文件上传的服务器端请求地址
        data: formData,
        processData: false,
        contentType: false,
        success:function(result){
            //do something
        }
    });
});
```


不过这里有兼容性问题，toBlob方法并不是所有的浏览器都实现了的，不过好在有兼容性的解决方案。  
项目地址：<https://github.com/blueimp/JavaScript-Canvas-to-Blob>

完整示例和源码可参考demo：<{{site.url}}/demos/cropper-demo.html>

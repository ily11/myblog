---
title: bootbox.js弹框
date: 2017-02-26 11:44:00
tags: bootstrap bootbox
categories: 前端
---
最近用bootbox做弹出框的时候遇到了一个问题，就是如何获取
弹出框dialog里文本输入的内容。刚开始，我自以为是的给text和textarea设置id，根据id获取其值，然而失败了，先把代码贴上来吧：
```
<div class="row">
    <div class="col-lg-12" style="margin: 20px">
        <button id="Dialog">Dialog</button>
        <button id="Alert">Alert</button>
        <button id="Confirm">Confirm</button>
        <button id="Promp">Prompt</button>
    </div>
</div>
<div id="dialogModel" style="display: none">
    <div class="row">
        <div class="col-md-12">
            <div class="form-group">
                <span>
                    活动名称：
                </span>
                <input type="text" id="name" class="form-control" placeholder="请输入活动名称">
            </div>
            <div class="form-group">
                <span>
                    活动描述：
                </span>
                <textarea class="form-control" id="desc" placeholder="请输入活动描述"></textarea>
            </div>
        </div>
    </div>
</div>
<script>
    $(function () {
        $("#Dialog").on("click",function () {
            bootbox.dialog({
                message:$("#dialogModel").html(),
                title:"创建活动",
                className:"modal-blue",
                buttons:{
                    success:{
                        label:"确定",
                        className:"btn-blue",
                        callback:function () {
                            var name = $("#name").val();
                            var desc = $("#desc").val();
                            alert(name +"    "+desc);
                        }
                    },
                    cannel:{
                        label:"取消",
                        className:"btn-default",
                        callback:function () {

                        }
                    }
                }
            })
        })
    })
</script>
```
弹出的效果如下，alert出来的是空：![](/images/notGetData.png)
多次查找无果后，请教了师兄，师兄说不能用Id来获取值的，因为我在页面写的id="dialogModel"这一块代码是在弹出对话框时填充到对话框里的message里的，这样的话就有两个id都是“name”的text和两个"id='desc'"的textarea了，用id肯定是取不出来值的，因为目前只有一个弹出框，所以下图标注的"class='modal-dialog'"肯定是唯一的，可以通过这个来找到值：![](/images/modaldialog.png)
修改代码如下：首先将text和textarea的id去了，然后将确定里的回调函数取值方式修改，值贴出回调函数的修改代码：
```
buttons:{
    success:{
        label:"确定",
        className:"btn-blue",
        callback:function () {
            var name = $(".modal-dialog .bootbox-body input[type='text']").val();
            var desc = $(".modal-dialog .bootbox-body textarea").val();
            alert(name +"    "+desc);
        }
    },
    cannel:{
        label:"取消",
        className:"btn-default",
        callback:function () {

        }
    }
}
```
此时就获取到文本框里的值了：![](/images/haveGetData.png)
下面就附带着整理一下bootbox的几种弹框方式，以便后面使用时查找。
### Dialog
dialog弹框就像上面写的那样，先把要弹出的内容写出来，一般写在页面的最后，然后就是`bootbox.dialog({})`里面配置dialog的title、message（就是将刚刚写的要弹出的内容填充进来）、className（可以引用自己写的样式，用beyongAdmin的话是有自己封装好的样式的，可以直接引用）、buttons（设置按钮显示文字以及样式、回调函数等）...
### Alert
alert就是弹出提示信息的，只有一个确定按钮
```
$("#alert").on("click",function () {
    bootbox.alert({
        size: "small",
        title: "提示信息",
        message: "活动描述不少于10个字！",
        callback: function(){

        }
    })
})
```
效果图如下：
![](/images/Alert1.png)
### Confirm
confirm是有两个按钮：取消和确定，主要是询问是否要做一件事的
```
$("#Confirm").on("click",function () {
    bootbox.confirm({
        size: "small",
        message: "Are you sure?",
        callback: function(result){
            /* result is a boolean; true = OK, false = Cancel*/
            if(result){
                //点击“确定”要操作的
            }
        }
    })
})
```
效果如下：
![](/images/confirm.png)
### Prompt
Prompt是让你输入信息后选择确定还是取消，它的设置没有message但是需要有title：
```
$("#Promp").on("click",function () {
    bootbox.prompt({
        size: "small",
        title: "What is your name?",
        callback: function(result){ /* result = String containing user input if OK clicked or null if Cancel clicked */ }
    })
})
```
效果如下：
![](/images/prompt.png)
也可以把文本框换成别的，textarea, email, select, checkbox, date, time, number, and password等，只要修改配置inputType即可。
以上弹出的都是英文，可以在bootbox.dialog之前加上`bootbox.setLocale("zh_CN");`，弹出的即是中文的，当然现在的按钮默认的是“确定”和“取消”，你也可以自定义按钮显示的文字，可以通过
```
buttons:{
    success:{
        label:"是",
        className:"btn-blue",
        callback:function () {

        }
    },
    cannel:{
        label:"否",
        className:"btn-default",
        callback:function () {

        }
    }
}
```
label来修改，当然这样子也可以直接将英文改成中文的，而不用加上面那句话了`bootbox.setLocale("zh_CN");`。

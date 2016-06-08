---
layout: post
tId: 1606021
title: "Knockout表单提交最佳实践"
date: 2016-06-08 17:46:00 +0800
categories: knockout,表单,最佳实践
codelang: javascript
desc: "前端MVC技术的流行，使得界面编写更加灵活、方便，本文结合实际中项目Knockout以及Knockout Validation的实践，总结下自己在后台管理中如何更加快速的实现表单验证与数据提交"
---
### 背景介绍

Knockout在实际项目中给了我很大的便利，让我从繁琐的Dom元素操作中解放出来。最开始从网上摘取的关于分页模块的代码，这个在列表页面给了我很大的帮助，现在的后台管理界面，首选便是Knockout。

首先记录下最近使用Knockout在表单提交上的实践，自认为是我目前做到的最佳，所以题目也起名叫最佳实践了

### 具体实践

这是涉及到前端和后台完整流程的一个过程。服务端业务在一定程度会有一些特定的限制，所以首先设计服务端数据结构（数据结构有所精简）。

```
public class CarEvaluation:IEntity
    {

        #region Id

        private string m_Gid;

        /// <summary>Gets or sets Id</summary>
        public string Gid
        {
            get { return m_Gid; }
            set { m_Gid = value; }
        }

        #endregion

        #region CmId

        private int m_cmId;

        /// <summary>Gets or sets CmId</summary>
        public int CmId
        {
            get { return m_cmId; }
            set { m_cmId = value; }
        }

        #endregion


        private String m_cmName;

        /// <summary>Gets or sets CmName</summary>
        public String CmName
        {
            get { return m_cmName; }
            set { m_cmName = value; }
        }

        public Dictionary<string,object> Extends { set; get; }

    }
```

我们据此设计前端数据实体，此处为了简便，并没有对属性进行序列化调整，后面对此有一定依赖，亦可以另外调整。

```
function carEvaluationInfoModel(item)
        {
            var self = this;

            if (item.Gid) {
                self.Gid = ko.observable(item.Gid);
            } else self.Gid = ko.observable("");

            self.CmId = item.CmId;
            self.CmName = item.CmName;

            @{
                string jsBuilder = "";
                //list为页面需要操作的控件配置
                foreach (var item in list)
                {//下拉框，必须有数据源
                    if (item.InputType == InputTypeEnum.Combobox && string.IsNullOrEmpty(item.InputData)) {
                        continue;
                    }
                    jsBuilder += "if(item.Extends&&item.Extends.P__"+item.ApiId+")";
                    jsBuilder += "{";
                    jsBuilder += "self.P__" + item.ApiId + "=ko.observable(item.Extends.P__" + item.ApiId + ")";
                    string extender = string.Empty;
                    if (!string.IsNullOrEmpty(item.RegexRule))
                    {
                        extender += ".extend({pattern:{message:'" + item.RegexWarning + "',"+"params:'" + item.RegexRule + "'}})";
                    }
                    if (item.IsRequired==1) {
                        extender += ".extend({required:true})";
                    }

                    jsBuilder += extender+";}";
                    jsBuilder += "else{self.P__" + item.ApiId + "=ko.observable()" + extender + ";}";
                }
            }
            @Html.Raw(jsBuilder)
        }
```

@语法是C# MVC中的Razor语法，页面请求编辑页时，将获取到的数据进行初始化，并通过服务器端语法，在页面加载时，动态定义js脚本内容
而我们list是动态配置的页面元素集合，我们通过集合对象的ApiId作为属性识别的主要依据，并在carEvaluationInfoModel中构造P__@item.ApiId的属性，在html部分进行的knockout绑定，也是结合服务器端语法，将属性绑定到对应的input元素上

```
@foreach (var item in list)
{
    bool isNeed = false;
    if (!string.IsNullOrEmpty(item.RegexRule)||item.IsRequired==1)
    {
        isNeed = true;
    }
    if (item.InputType == InputTypeEnum.Textbox)
    {
        <div class="form-group form-inline">
            <label class="control-label w200">@item.Title：</label>
            <input type="text" data-bind="value:carModel().P__@item.ApiId" class="form-control" />
            <span class="text-info">@item.Unit</span>
            @if (isNeed)
            {
                <span class="text-info" data-bind="validationMessage:carModel().P__@item.ApiId"></span>
            }
        </div>
    }else ...
}
```

ko.validatedObservable是Knockout Validation的语法，页面加载时，调用ko.validation.init({insertMessages:false});来启用并初始化
carEvaluationInfo是将页面加载时通过id获取的服务器端数据进行Json.net序列化得到的。以参数的方式传入carEvaluationModel，构造Knockout绑定的carModel属性（carEvaluationInfoModel），html中的属性绑定，都使用carModel的属性。

```
ko.validation.init({insertMessages:false});

 function carEvaluationModel(item) {
            var self = this;

            self.carModel=ko.validatedObservable(new carEvaluationInfoModel(item));
            console.log(self.carModel());

            self.addNew = function () {
                if(!self.carModel.isValid()){
                    return;
                }

				var master=$("#master4");

                self.CmId=master.val();
                if(self.CmId<=0){
                    self.carModel().CmName="";
                    return;
                }else{
                    self.carModel().CmName=master.find("option:selected").text().substr(2);
                }

                var str=ToJson(self.carModel());
                console.log(str);
                $.post(urlConfig.save,{json:str},function(data){
                    console.log(data.msg);
                    if(data.error==1)
                    {
                        return;
                    }
                    else{
                        window.location.href=urlConfig.list;
                    }
                });
            }
        }

        var carEvaluationInfo=@Html.Raw(Newtonsoft.Json.JsonConvert.SerializeObject(Model));
        var model = new carEvaluationModel(carEvaluationInfo);

        ko.validatedObservable(model);
        ko.applyBindings(model);
```

在添加数据，并提交时，首先使用self.carModel.isValid()进行用户输入验证，更多用法参考Knockout Validation的git库。
而此时的关键点就是另外一个js函数ToJson了

```
function ToJson(obj) {
    var array = [];
    array.push("{");
    for (var sProp in obj) {
        if (sProp[0] >= 'A' && sProp[0] <= 'Z') {
            var value = "";
            if (typeof obj[sProp] == 'function') {
                value = obj[sProp]();
            } else {
                value = obj[sProp];
            }
            if (value == null || typeof value === 'undefined') {
                value = "";
            } else if (typeof (value) === "boolean") {
                value = value == true ? 1 : 0;
            }

            array.push("\"" + sProp + "\"");
            array.push(":\"" + value + "\"");
            array.push(",");
        }
    }
    array.pop();
    array.push("}");
    var str = array.join("");
    //console.log(str);
    return str;
}
```

该方法，将carModel属性的对象按照我们预想，构造为json字符，sProp[0] >= 'A' && sProp[0] <= 'Z'通过这个判断，我们只转换对象属性中首字母大写的属性，我通过这种方式来识别文本框关联的属性数据，也就是我们之前说的前后端数据结构上有点依赖（也算是犯懒了）

```
public static Mongo.Model.CarEvaluation JsonToEvaluation(string json)
{
    object obj = Newtonsoft.Json.JsonConvert.DeserializeObject(json);
    JObject jData = (JObject)obj;
    Mongo.Model.CarEvaluation ce = new Mongo.Model.CarEvaluation();
    Type t = ce.GetType();
    var properties = t.GetProperties();
    foreach (var item in jData)
    {
        var prop = t.GetProperty(item.Key);
        if (prop != null)
        {
            var pType = prop.PropertyType;
            prop.SetValue(ce, Convert.ChangeType(((Newtonsoft.Json.Linq.JValue)item.Value).Value, pType));
        }
        else {
            if (ce.Extends == null) ce.Extends = new Dictionary<string, object>();

            string value = (string)Convert.ChangeType(((Newtonsoft.Json.Linq.JValue)item.Value).Value, typeof(string));
            string regexInt = "^\\d+$";
            string regexFloat = "\\d+\\.\\d+";
            if (!string.IsNullOrEmpty(value))
            {
                if (Regex.IsMatch(value, regexInt))
                {
                    ce.Extends[item.Key] = Convert.ToInt32(value);
                }
                else if (Regex.IsMatch(value, regexFloat))
                {
                    ce.Extends[item.Key] = Convert.ToDouble(value);
                }
                else {
                    ce.Extends[item.Key] = value;
                }
            }
            else {
                ce.Extends[item.Key] = value;
            }
        }
    }
    return ce;
}
```

这里负责将前端提交的数据进行反向序列化为特定对象，这里的处理使用了Json.net，对int和float类型进行了单独处理（数据采用mongodb进行保存，为了防止通过string
的方式对数值类型排序，造成错误），由于字典属性Extends的存在，在加上犯了点懒，所以没有重构，只实现了转为为CarEvaluation特定类。

更加优美的实践，其实是重构以后，可以转换为任何类对象，但其实那些没有Extends字典动态属性的类结构，可直接使用Json.net反序列化）

转换后的便得到提交的form表单数据，在Save的函数中进行对应的保存和更新操作即可。

### 总结

通过前后端的绑定和一些特定依赖（相对灵活的依赖，必将前后端彼此数据结构要一致，以便于反序列化），我们可以迅速构造转换前后端的数据，实现快速的form表单数据提交的工作。






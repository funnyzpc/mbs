### js api 之 fetch、querySelector、form、atob及btoa


js api即为JavaScript内置函数，本章就说说几个比较实用的内置函数，内容大致如下：

+ fecth http请求函数
+ querySelector 选择器
+ form 表单函数
+ atob与btoa Base64函数


#### Base64之atob与btoa

以前，在前端，我们是引入Base64.js后调用api实现数据的Base64的编码和解码的运算，现在新的ES标准为我们提供了Base64
的支持，主要用法如下：
+ 编码：__window.btoa__(_param_);
```
   输入> window.btoa("hello");
   输出> "aGVsbG8="
```
+ 解码：__window.atob__(_param_)
```
   输入：window.atob("aGVsbG8=");
   输出："hello"
```

#### DOM选择器之 querySelector

DOM选择器在jQuery中用的十分广泛，极大地方便了前端开发，现在你有了__querySelector__,不用引入恼人的js及
各种js依赖，一样便捷开发～

+ ID选择
```
    // 获取DOM中的内容
    document.querySelector("#title").innerText;
    // 将DOM设置为粉红色背景
    document.querySelector("#title").style.backgroundColor="pink";
    // 获取DOM的class属性
    document.querySelector("#title").getAttribute("class");
    // 移除DOM
    document.querySelector("#title").remove();
    // 获取子DOM
    document.querySelector("#title").childNodes;
    // 给DOM添加click事件(点击后弹出 "success")
    document.querySelector("#title").onclick = function(){alert("success")};
    // 给DOM添加属性(添加一个可以为name，value为hello的属性)
    document.querySelector("#title").setAttribute("name","hello");
```
+ class选择
```
    // 获取DOM中的内容
    document.querySelector(".title").innerText;
    // 将DOM设置为粉红色背景
    document.querySelector(".title").style.backgroundColor="pink";
    // 获取DOM的class属性
    document.querySelector(".title").getAttribute("class");
    // 移除DOM
    document.querySelector(".title").remove();
    // 获取子DOM
    document.querySelector(".title").childNodes;
    // 给DOM添加click事件(点击后弹出 "success")
    document.querySelector(".title").onclick = function(){alert("success")};

```
+ tag选择器(DOM名称)
```
    // 获取DOM中的内容
    document.querySelector("h4").innerText;
    // 将DOM设置为粉红色背景
    document.querySelector("h4").style.backgroundColor="pink";
    // 获取DOM的class属性
    document.querySelector("h4").getAttribute("class");
    // 移除DOM
    document.querySelector("h4").remove();
    // 获取子DOM
    document.querySelector("h4").childNodes;
    // 给DOM添加click事件(点击后弹出 "success")
    document.querySelector("h4").onclick = function(){alert("success")};
    // 给DOM添加属性(添加一个可以为name，value为hello的属性)
    document.querySelector("h4").setAttribute("name","hello");
```
+ 自定義屬性選擇(多用於表單)
```
    // 获取DOM的value值
    document.querySelector("input[name=age]").value;
    // 将DOM设置为粉红色背景
    document.querySelector("input[name=age]").style.backgroundColor="pink";
    // 获取DOM的class属性
    document.querySelector("input[name=age]").getAttribute("class");
    // 移除DOM
    document.querySelector("input[name=age]").remove();
    // 获取子DOM
    document.querySelector("input[name=age]").childNodes;
    // 给DOM添加click事件(点击后弹出 "success")
    document.querySelector("input[name=age]").onclick = function(){alert("success")};
    // 给DOM添加属性(添加一个可以为name，value为hello的属性)
    document.querySelector("input[name=age]").setAttribute("name","hello");
```

#### form表單函數

以前我們是沒有表單函數的時候，如果做表單的提交大多定義一個提交按鈕，用jQuery+click函數實現表單提交，
或者獲取參數後使用ajax提交，對於後者暫且不說，對於前者 ES標準提供了新的函數 form函數，當然這個只是
document的一個屬性而已，需要提醒的是這個函數使用的前提是需要給form標籤定義一個name属性，这个name属性
的值即为表单函数的函数名字(也可为属性),具体用法如下；

比如我们的表单是这样的：

```
   // html表单
   <form name="fm" method="post" action="/submit">
       <input type="text" name="age" placeholder="请输入年龄"/>
   </form>
```

这个时候我们可以这样操作表单：

```
    // 提交表单
    document.fm.submit();
    // 获取表单的name属性值
    document.fm.name;
    // 获取表单的DOM
    document.fm.elements;
    // resetb表单
    document.fm.reset();
    // ...更多操作请在chrome控制台输入命令
```

#### fetch

 fetch 为js 新内置的http请求函数，用于替代ajax及原始的XMLHttpRequest，与ajax相似的是它提供了请求头，异步或同步方法，同时也提供了GET、PUT、DELETE、OPTION等
 请求方式,唯一缺憾的是除了POST(json)方式提交外，其他方式均需要自行组装参数，这里仅给出几个简单样例供各位参考。

#####  fetch：GET请求

html:
```
    <form method="GET" style="margin-left:5%;">
        <label>name:</label><input type="text" name="name"/>
        <label>price:</label><input type="number" name="price"/>
        <label><button type="button" onclick="getAction()">GET提交</button></label>
    </form>
```

javaScript:
```
    function getAction() {
            // 组装请求参数
            var name = document.querySelector("input[name=name]").value;
            var price = document.querySelector("input[name=price]").value;

            fetch("/get?name="+name+"&price="+price, {
                method: 'GET',
                headers: {
                    'Content-Type': 'application/json'
                },
                // body: JSON.stringify({"name":name,"price":price})
            })
            .then(response => response.json())
            .then(data =>
                document.getElementById("result").innerText = JSON.stringify(data))
            .catch(error =>
                console.log('error is:', error)
            );
        }
```

这里的GET请求(如上)，注意如下：

+ 需手动拼接参数值`/get?name=name&price=price`
+ 由于GET请求本身是没有请求体的，所以fetch的请求配置中一定不能有body的配置项
+ 由于GET请求本身是没有请求体的，所以headers项可以不配置
+ 请求结果在第一个then的时候，数据是一个steam，所以需要转换成json(调用json()方法)
+ 请求结果在第二个then的时候仍然是一个箭头函数，这个时候如需要对数据进行处理请调用自定义函数处理

#####  fetch：POST(json)请求

html:
```
    <form method="GET" style="margin-left:5%;">
        <label>name:</label><input type="text" name="name"/>
        <label>price:</label><input type="number" name="price"/>
        <label><button type="button" onclick="getAction()">GET提交</button></label>
    </form>
```

javaScript:
```
    function getAction() {
            // 组装请求参数
            var name = document.querySelector("input[name=name]").value;
            var price = document.querySelector("input[name=price]").value;
            price = Number(price)
            fetch("/post", {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({"name":name,"price":price})
            })
            .then(response => response.json())
            .then(data =>
                document.getElementById("result").innerText = JSON.stringify(data))
            .catch(error =>
                console.log('error is:', error)
            );
       }
```

这里需要注意对是：
+ Post请求的请求头的内容类型必须是`application/json`,至于`application/x-www-form-urlencoded`我一直没测通过，请各位指点
+ 请求体中的数据对象必须使用JSON.stringify() 函数转换成字符串

#####  fetch：POST(form)请求

html：
```
       <form method="GET" style="margin-left:5%;" name="fm" action="/form">
            <label>name:</label><input type="text" name="name"/>
            <label>price:</label><input type="number" name="price"/>
        </form>
```
javaScript:
```
        function getAction() {
            // 组装请求参数
            let name = document.querySelector("input[name=name]").value;
            let price = document.querySelector("input[name=price]").value;
            // price = Number(price)
            /*
            let formdata = new FormData();
            formdata.append("name",name);
            formdata.append("price",price);
            */
            let data = new URLSearchParams();
            data.set("name",name);
            data.set("price",price);
            fetch("/form", {
                method: 'POST',
                headers: {
                     "Content-Type":"application/x-www-form-urlencoded;charset=UTF-8"
                },
                body: data
            })
            .then( response =>response.json() )
            .then(function (data){
                this.success(data);
            })
            .catch(error =>
                console.log('error is:', error)
            );
        }
        function success(data) {
            document.getElementById("result").innerText = JSON.stringify(data)
            alert(window.atob(data.sign))
        }
```

可以看到中间改过几次，实在不理想，后有改成URLSearchParams来拼装请求数据，后来成功了，各位要有其他方式请指点一二。
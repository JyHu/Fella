# 简单脚本交互



在Fella中，提供了一些简单的JS脚本交互支持，用于方便您的数据导入和文本翻译等，这一章内容，我会做一个简单的说明。

Fella目前提供了两个简单方法供JS调用，Fella在加载JS脚本代码的时候会向JS中注入一个`Fella`对象，您可以通过该对象来调用原生方法：

```
/// 在JS中调用Fella发起网络请求，请求结果会同时把所有参数一并带回
/// - Parameters:
///   - urlString: 请求地址
///   - params: 参数
///   - method: 方法
///   - headers: 请求头
///   - verification: 校验内容
///   - callback: 回调函数，在Fella请求到结果后，会调用这个JS函数把结果回传给JS
func request(_ urlString: String,
             _ params: [String: Any]?,
             _ method: String,
             _ headers: [String: String]?,
             _ verification: Any?,
             _ callback: String)

/// JS显示调试日志
/// - Parameter log: 调试日志
func showLog(_ log: Any)
```



## 1. 翻译脚本

用于在编辑字符串时快速的对输入文案做翻译，这里用到了Javascript脚本语言。

**1** 在JS中添加一个翻译方法：

```
// s 需要翻译的内容
// t 需要翻译到的语言
// v 用来校验的返回参数
function translate(s,t,v)
```

在点击翻译按钮时，Fella内部会调用JS中的`translate`方法，JS内可以根据自己的习惯或者用到的开放API执行翻译。

**2** 请求Fella翻译接口，调用方法如：

```
Fella.request(u,p,m,h,v,c)
```

**3** 通过callback回传请求结果，内容包含发起请求时的各种参数，格式如下:

```
{
    "method": xxx,
    "verification":,
    "params": xxx,
    "headers": xxx,
    "success": xxx,
    "error": xxx
}
```

**4** 在JS脚本中接收到请求结果并解析后，将解析内容返回给Fella：

```
/// 回调结果方法
/// - Parameters:
///   - result: 翻译结果
///   - language: 语言
///   - verification: 校验内容
func translateResult(_ result: String, _ language: String, _ verification: Any)
```

**5** 这里是一个我使用**[百度翻译开放API](https://fanyi-api.baidu.com/product/153)**实现的例子：

```
var MD5 = function (string) // 百度SDK里提供的MD5算法

function translate(s,t,v) {
    Fella.showLog("begin request");

    var appid = '你申请的appid';
    var key = '你申请的key';
    var salt = (new Date).getTime();
    var query = s;
    var sign = MD5(appid + query + salt +key);
    
    Fella.request('https://fanyi-api.baidu.com/api/trans/vip/translate',
                {
                    q: query,
                    appid: appid,
                    salt: salt,
                    from: "auto",
                    to: t,
                    sign: sign
                },
                "get",
                null,
                v,
                "parse")
}

function parse(res) {
    var r = res["success"]["trans_result"][0]["dst"]
    var l = res["params"]["to"]
    var v = res["verification"]
    Fella.showLog(res)
    Fella.translateResult(r, l, v)
}
```

## 2. 内容导入JS

使用Javascript脚本语言提供给使用者自定义导入本地化资源的方式。

**1.** 在JS中添加一个方法，由Fella调用：

```
function importI18ns()
```

**2.** Fella中提供了额外的以下几个方法：

```
/// 更新本地化内容
/// - Parameters:
///   - i18nJson: 本地化内容的json内容
///   - fileName: 文件名的相对路径，在Fella中切换文件的地方显示在文件名后
func update(_ i18nJson: [String: Any], _ fileName: String)
    
/// 获取本地i18nString内容
/// - Parameters:
///   - identifier: 字符串标识
///   - fileName: 文件名的相对路径，在Fella中切换文件的地方显示在文件名后
///   - callback: 回调方法
func i18nString(_ identifier: String, _ fileName: String, _ callback: String)
    
/// 刷新字符串列表，在所有数据遍历结束后调用
func reloadI18nList()
```


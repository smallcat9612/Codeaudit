YouDianCMS 9.5.21 反序列化漏洞分析
漏洞概述

经过对YouDianCMS 9.5.21中`Cookie.class.php`文件的分析，确认该文件中存在不安全的反序列化操作，但在当前系统中的实际风险较低，可能属于误报。

详细分析

Cookie.class.php中的反序列化操作

在`App/Core/Extend/Library/ORG/Util/Cookie.class.php`文件中，`get`方法存在不安全的反序列化操作：

```
php
static function get($name) {
    $value = $_COOKIE[C('COOKIE_PREFIX').$name];
    $value = unserialize(base64_decode($value));
    return $value;
}
```

这段代码直接对从`$_COOKIE`获取的值进行base64解码后反序列化，没有进行任何验证或过滤，理论上存在反序列化漏洞风险。

调用链分析

通过全面搜索系统代码，发现对`Cookie::get`方法的调用非常有限，仅在以下位置使用：

`App/Core/Extend/Behavior/BrowserCheckBehavior.class.php`中：
```
php
if(Cookie::is_set('_last_visit_time_'.$guid) && Cookie::get('_last_visit_time_'.$guid)>time()-C('LIMIT_REFLESH_TIMES')) {
    // 页面刷新读取浏览器缓存
    header('HTTP/1.1 304 Not Modified');
    exit;
}
```

这个使用场景相对安全，因为：
- 它只检查时间值是否大于某个阈值，不执行复杂的对象操作
- 存储的值是由系统设置的`$_SERVER['REQUEST_TIME']`，不是用户可控的输入
- Cookie的设置在同一个类中完成：`Cookie::set('_last_visit_time_'.$guid, $_SERVER['REQUEST_TIME'],$_SERVER['REQUEST_TIME']+3600);`

系统中其他反序列化点

系统中还存在其他反序列化操作，主要在以下位置：

1. 缓存相关类中（CacheApachenote, CacheDb, CacheShmop, CacheSqlite, CacheFile）
2. 会话管理（Session.class.php）
3. 高级模型（AdvModel.class.php）
4. 购物车功能（YdCart.class.php）

特别是在`YdCart.class.php`中，有多处对cookie数据的反序列化操作：

```
php
$data = unserialize(stripslashes($data));
```

虽然使用了`stripslashes`处理，但仍然可能存在安全风险。然而，这些反序列化操作通常处理的是系统内部生成的数据，而非直接处理用户输入。

漏洞利用条件

要利用`Cookie.class.php`中的反序列化漏洞，需要满足以下条件：

1. 系统中存在可利用的POP链（Property-Oriented Programming）
2. 攻击者能够控制Cookie的内容
3. 系统中存在使用`Cookie::get`方法处理用户可控数据的代码路径

经过分析，虽然条件2和3可能满足，但系统中对`Cookie::get`的使用非常有限，且处理的是系统内部生成的数据，不是用户可控的输入。同时，没有发现明显可利用的POP链。

最终利用：
curl -v -H "Cookie: last_visit_time_[目标的guid的md5值]=TzoxMDoiRXhhbXBsZSI6MTp7czo0OiJkYXRhIjtzOjk6InN5c3RlbSgid2hvYW1pIik7Ijt9" http://192.168.1.95/

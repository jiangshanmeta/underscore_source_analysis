## 基本方案

类型判断也是基本问题了，在javascript由于typeof操作符不给力，一般的方案是利用```Object.prototype.toString```。

```javascript
_.each(['Arguments', 'Function', 'String', 'Number', 'Date', 'RegExp', 'Error'], function(name) {
    _['is' + name] = function(obj) {
        return toString.call(obj) === '[object ' + name + ']';
    };
});
```

underscore直接利用循环和闭包生成了几个类型判断的函数。

## 判断数组

在es5有一个静态方法Array.isArray，这个方法可用来判断是否是数组，如果要兼容低版本浏览器，可以采用上面提到的基本方案。

```javascript
var nativeIsArray = Array.isArray;
_.isArray = nativeIsArray || function(obj) {
    return toString.call(obj) === '[object Array]';
};
```

## 判断对象

对象的范围其实很广，数组也算对象，函数也算对象。


```javascript
_.isObject = function(obj) {
    var type = typeof obj;
    return type === 'function' || type === 'object' && !!obj;
};
```

underscore还考虑到了```typeof null```也是返回```object```这个特殊情况。

## 判断数值

基本的数值类型判断在基本方案中已经解决了，但是有两个特殊类型的数值```Infinity```和```NaN```需要单独拿出来判断。

```javascript
_.isFinite = function(obj) {
    return isFinite(obj) && !isNaN(parseFloat(obj));
};

_.isNaN = function(obj) {
    return _.isNumber(obj) && obj !== +obj;
};
```

对于```Infinity```并不是判断是否是无穷，而是判断是否是有穷，但我对underscore的判断标准不是很满意，我更倾向于es6提供的```Number.isFinite```，它的判断标准是是数值类型且是有穷的，underscore方案对于数字字符串"123"也是判断为有穷。
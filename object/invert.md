在PHP中有一个函数叫```array_flip```,它的作用是交换键与值。underscore的invert方法就是完成同样的任务。

实现这个方法思路也很简单，只需一次遍历交换键值构建一个新的对象即可：

```javascript
_.invert = function(obj) {
    var result = {};
    var keys = _.keys(obj);
    for (var i = 0, length = keys.length; i < length; i++) {
        result[obj[keys[i]]] = keys[i];
    }
    return result;
};
```

作为对比我们看一下phpjs这个项目中是如何实现相同功能的：

```javascript
module.exports = function array_flip (trans) {
    var key
    var tmpArr = {}
    for (key in trans) {
        if (!trans.hasOwnProperty(key)) {
            continue
        }
        tmpArr[trans[key]] = key
    }
    return tmpArr
}
```

在phpjs中使用了```for in```进行循环，在循环内部判断是否是自身属性，而在underscore中通过```keys```方法已经把非自身属性筛除了，只对自身可枚举属性进行循环。两者其实都是限定在自身可枚举属性身上，只是具体细节上略有差异。
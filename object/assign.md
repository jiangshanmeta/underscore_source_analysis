在javascript中有个静态方法Object.assign，它的作用是把自身可枚举属性的值从一个或多个源对象复制到目标对象。关于这个方法，我曾经解读过[如何进行polyfill](https://jiangshanmeta.gitbooks.io/javascript-polyfill/content/object/assign.html)。

在underscore中有三个相关方法：extend、extendOwn、defaults。其中extendOwn与Object.assign功能一致，都是复制自身可枚举属性的值到目标对象上。extend方法所拷贝的不仅仅是自身可枚举的属性，原型链上的属性也可以复制。defaults方法与extend方法类似，所要复制的属性包含原型链上的，但只有目标对象没这个属性或者属性值为undefined时才拷贝到目标对象上。

这三个功能相近的方法，underscore依然采用高阶函数的形式，通过控制不同的输入返回具有不同功能的函数：


```javascript
var createAssigner = function(keysFunc, undefinedOnly) {
    return function(obj) {
        var length = arguments.length;
        if (length < 2 || obj == null) return obj;
        for (var index = 1; index < length; index++) {
            var source = arguments[index],
                // 复制哪些属性，由keysFunc决定
                keys = keysFunc(source),
                l = keys.length;
            for (var i = 0; i < l; i++) {
                var key = keys[i];
                // undefinedOnly是为了defaults而设立的
                // 对于defaults方法，只有目标对象对应属性值为undefined才复制
                if (!undefinedOnly || obj[key] === void 0) obj[key] = source[key];
            }
        }
        return obj;
    };
};
_.extend = createAssigner(_.allKeys);
_.extendOwn = _.assign = createAssigner(_.keys);
_.defaults = createAssigner(_.allKeys, true);
```

其实见多了underscore类似的代码，再结合上面提到的polyfill，理解这些代码没什么太大的难度。

这几个方法还有一个共同点是都是浅复制，underscore没有提供深复制的方法。如果想要了解深复制可以看下jQuery的```$.extend```或者lodash的```cloneDeep```，基本思路是类型判断引用类型递归拷贝。
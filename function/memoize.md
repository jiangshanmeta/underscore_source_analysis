将一个函数的结果缓存起来这需求还是挺常见的，比如在处理浏览器兼容性的时候缓存下带有前缀的CSS属性名，减少不必要的运算。underscore的momoize函数是一个函数加工工厂，它将传入的函数加工成了具有缓存结果能力的新函数。

```javascript
_.memoize = function(func, hasher) {
    var memoize = function(key) {
        var cache = memoize.cache;
        var address = '' + (hasher ? hasher.apply(this, arguments) : key);
        if (!_.has(cache, address)) cache[address] = func.apply(this, arguments);
        return cache[address];
    };
    memoize.cache = {};
    return memoize;
};
```

memoize方法返回新函数memoize，这个新函数memoize有一个静态属性cache用来缓存结果。缓存默认采用传入的第一个参数作为索引，不过我们也可以传入```hasher```函数根据需要返回索引值。
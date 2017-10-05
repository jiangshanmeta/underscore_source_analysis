在underscore中针对集合的方法许多都用到了iteratee。我们传参时iteratee可以有不同的类型，在underscore内部它会首先被统一处理成迭代函数的形式。

```javascript
var cb = function(value, context, argCount) {
    if (value == null) return _.identity;
    if (_.isFunction(value)) return optimizeCb(value, context, argCount);
    if (_.isObject(value)) return _.matcher(value);
    return _.property(value);
};
```

#### iteratee为null

当我们不传iteratee(相当于传入undefined)或者传入null时，此时迭代函数的行为是原样返回当前被迭代元素：

```javascript
_.identity = function(value) {
    return value;
};
```

#### iteratee为函数

我们需要将iteratee转换成一个函数，如果传入的iteratee就是个函数，我们似乎什么事情都不用做。underscore对于这种情况做了一些底层性能上的优化：

```javascript
var optimizeCb = function(func, context, argCount) {
    // 如果没有context(我遇到的基本就是这种情况),原样返回
    if (context === void 0) return func;

    // 有context的，针对参数数量做优化
    switch (argCount == null ? 3 : argCount) {
        case 1: return function(value) {
            return func.call(context, value);
        };
        case 2: return function(value, other) {
            return func.call(context, value, other);
        };
        case 3: return function(value, index, collection) {
            return func.call(context, value, index, collection);
        };
        case 4: return function(accumulator, value, index, collection) {
            return func.call(context, accumulator, value, index, collection);
        };
    }
    return function() {
        return func.apply(context, arguments);
    };
};
```

从最终结果上看，switch语句里返回的函数和最终返回的函数起到的是同一个作用，那为什么不直接用最后的使用apply方法的函数呢？因为arguments对象开销比较大，能不用就不要用，而且call方法也比apply方法快，这两方面因素决定了如果知道迭代函数有几个参数，就用几个参数的版本，而不是一味使用apply+arguments。

#### iteratee为对象

当传入的iteratee是一个对象时，转换成的迭代函数的行为是返回当前被迭代元素是否与iteratee匹配(即iteratee是否是当前被迭代元素的子集)。

```javascript
_.matcher = _.matches = function(attrs) {
    // 只考虑自身属性
    attrs = _.extendOwn({}, attrs);
    return function(obj) {
        return _.isMatch(obj, attrs);
    };
};

_.isMatch = function(object, attrs) {
    var keys = _.keys(attrs), length = keys.length;
    if (object == null) return !length;
    var obj = Object(object);
    for (var i = 0; i < length; i++) {
        var key = keys[i];
        if (attrs[key] !== obj[key] || !(key in obj)) return false;
    }
    return true;
};

```


#### iteratee为字符串、数字

当传入的iteratee是一个字符串或者数字时，转换成的迭代函数的行为是返回当前被迭代元素的对应属性。

```javascript
var property = function(key) {
    return function(obj) {
        return obj == null ? void 0 : obj[key];
    };
};
```

经过处理，iteratee最终都会转换成为迭代函数的形式。同时这一内部方法也被暴露出来(虽然感觉用处不大)

```javascript
_.iteratee = function(value, context) {
    return cb(value, context, Infinity);
};
```
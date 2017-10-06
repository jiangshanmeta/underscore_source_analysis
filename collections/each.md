在javascript的数组原型对象上，有```forEach```、```map```、```reduce```、```reduceRight```这几个方法，underscore将这几个方法拓展到了对象上。


## each

如果你试图polyfill过数组原型对象上的forEach方法，将其拓展到对象上也没什么特别的难度，无非是索引从数字索引变成了对象的键。

```javascript
_.each = _.forEach = function(obj, iteratee, context) {
    iteratee = optimizeCb(iteratee, context);
    var i, length;
    if (isArrayLike(obj)) {
        // 数组、类数组，按照数字下标遍历
        for (i = 0, length = obj.length; i < length; i++) {
            iteratee(obj[i], i, obj);
        }
    } else {
        // 对象，按照Object.keys返回的键的顺序进行遍历
        var keys = _.keys(obj);
        for (i = 0, length = keys.length; i < length; i++) {
            iteratee(obj[keys[i]], keys[i], obj);
        }
    }
    return obj;
};
```

underscore默认each方法传入的iteratee是一个函数形式，毕竟each方法仅仅负责遍历而已，其核心业务逻辑在于iteratee函数内部，而不需要考虑每一次迭代的返回值。

## map

map方法是根据原数组将其映射为一个新数组，它关注的是每一次迭代的返回值。

```javascript
_.map = _.collect = function(obj, iteratee, context) {
    iteratee = cb(iteratee, context);
    var keys = !isArrayLike(obj) && _.keys(obj),
        length = (keys || obj).length,
        results = Array(length);
    for (var index = 0; index < length; index++) {
        var currentKey = keys ? keys[index] : index;
        results[index] = iteratee(obj[currentKey], currentKey, obj);
    }
    return results;
};
```

map方法从实现上和each方法非常类似，它比each多做的是把每一次迭代的结果按序放入到了一个数组中。

我很好奇的一点是，为什么each方法不安map方法的格式来写？写成这样：

```javascript
_.each = function(obj, iteratee, context) {
    iteratee = optimizeCb(iteratee, context);
    var keys = !isArrayLike(obj) && _.keys(obj),
        length = (keys || obj).length,
    for (var index = 0; index < length; index++) {
        var currentKey = keys ? keys[index] : index;
        iteratee(obj[currentKey], currentKey, obj);
    }
    return obj;
};
```

或许是考虑到了每次判断是否有keys对性能的损耗吧，那既然如此map方法为什么不按照each方法的格式来写呢？

## reduce reduceRight

有了上面的示例，其实实现reduce或者reduceRight没有什么技术上的问题，underscore高明在没有去生写这两个方法，而是抽象出这两个方法的共同点和差异点，利用一个高阶函数返回这两个具体的方法。

```javascript
function createReduce(dir) {
    function iterator(obj, iteratee, memo, keys, index, length) {
        // 按序遍历规约处理
        for (; index >= 0 && index < length; index += dir) {
            // 针对数组和对象确定不同的索引
            var currentKey = keys ? keys[index] : index;
            memo = iteratee(memo, obj[currentKey], currentKey, obj);
        }
        return memo;
    }

    return function(obj, iteratee, memo, context) {
        iteratee = optimizeCb(iteratee, context, 4);
        var keys = !isArrayLike(obj) && _.keys(obj),
        length = (keys || obj).length,
        index = dir > 0 ? 0 : length - 1;
        // 默认起始规约值是第一个要迭代的元素
        if (arguments.length < 3) {
            memo = obj[keys ? keys[index] : index];
            index += dir;
        }
        return iterator(obj, iteratee, memo, keys, index, length);
    };
}
```

reduce和reduceRight方法唯一的差别是迭代处理的方向，分别用1和-1表示，通过这种表示方法我们也可以轻松获得下一个要迭代元素的索引。


## invoke pluck

这两个方法其实是对map方法的运用。

```javascript
// method可以是具体的方法，也可以是方法名
_.invoke = function(obj, method) {
    var args = slice.call(arguments, 2);
    var isFunc = _.isFunction(method);
    return _.map(obj, function(value) {
        // 如果传入的是方法名，会调用被迭代元素的对应方法
        var func = isFunc ? method : value[method];
        return func == null ? func : func.apply(value, args);
    });
};

_.pluck = function(obj, key) {
    return _.map(obj, _.property(key));
};
```

invoke获取的新数组，是由旧数组每个元素经过传入方法处理构成的。pluck获得的新数组，是由旧数组每一项特定属性构成的。
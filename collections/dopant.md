## max min

max、min两个方法是用来求集合中的最值的。求最值这个问题本身没有什么特别的难度，唯一要说的是作者使用了Infinity这个特殊的数值，在javascript中任何合法的数值都介于-Infinity和Infinity之间。

```javascript
_.max = function(obj, iteratee, context) {
    var result = -Infinity, lastComputed = -Infinity,
        value, computed;
    if (iteratee == null && obj != null) {
        obj = isArrayLike(obj) ? obj : _.values(obj);
        for (var i = 0, length = obj.length; i < length; i++) {
            value = obj[i];
            if (value > result) {
                result = value;
            }
        }
    } else {
        iteratee = cb(iteratee, context);
        _.each(obj, function(value, index, list) {
            computed = iteratee(value, index, list);
            // 如果当前计算值比前面最大计算值还大，更新最大值
            if (computed > lastComputed || computed === -Infinity && result === -Infinity) {
                result = value;
                lastComputed = computed;
            }
        });
    }
    return result;
};
```

作者还特地把没有回调函数的情况拿了出来，没有回调函数其实就是使用元素本身的值作比较，如果没有这个判断，通过cb方法回调函数会变为```_.identity```，对于最终结果没有影响，作者特意摘出来这一特殊情况应该是出于性能上的考虑。

## toArray

在ES6时代，将对象转成数组，只需调用Object.values方法，将类数组转换成数组，只需调用Array.from方法，而浅复制一个数组，只需使用结构赋值即可。虽说ES6给我们提供了诸多便利，但是其polyfill我们也许了解

```javascript
_.toArray = function(obj) {
    if (!obj) return [];
    if (_.isArray(obj)) return slice.call(obj);
    if (isArrayLike(obj)) return _.map(obj, _.identity);
    return _.values(obj);
};
```

虽然Object.values方法是ES6才支持的，但是它的polyfill真的很好实现，underscore的values方法也实现了同样的功能。

类数组转数组，我更习惯与使用slice方法，underscore的做法是手动遍历了一遍类数组按序放到新数组中。

数组的浅复制用slice方法，这是个常见的方案。

## size

对于一个数组/类数组，其大小就是length属性，对于一个对象，其大小就是自身拥有的属性的数量：

```javascript
_.size = function(obj) {
    if (obj == null) return 0;
    return isArrayLike(obj) ? obj.length : _.keys(obj).length;
};
```
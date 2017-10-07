在underscore集合相关方法中，我觉得写得最好的就是就是聚合相关的，这一部分包括三个方法：groupBy、indexBy、countBy。然而重点并不是这三个方法本身，而是生成这三个函数的高阶函数group。

## group

如果自己实现上面提到的三个方法，似乎没什么特别的难度，groupBy是遍历根据iteratee返回的key聚合，indexBy是遍历按照iteratee返回的key重新组织数据，countBy是遍历根据iteratee返回的key计数。这三者的关联都是遍历根据iteratee返回的key做事情，而三者所做的具体事情又不一样。


```javascript
var group = function(behavior) {
    return function(obj, iteratee, context) {
        var result = {};
        iteratee = cb(iteratee, context);
        _.each(obj, function(value, index) {
            var key = iteratee(value, index, obj);
            behavior(result, value, key);
        });
        return result;
    };
};
```

group方法就把三者不同点当成一个参数behavior，由外部传入，而遍历和通过iteratee获取key这种共同点统一处理。

在underscore中高阶函数大量运用，这一点很值得我们去学习。

## groupBy indexBy countBy


```javascript
// 根据key聚合
_.groupBy = group(function(result, value, key) {
    if (_.has(result, key)) result[key].push(value); else result[key] = [value];
});

// 根据key重新组织
_.indexBy = group(function(result, value, key) {
    result[key] = value;
});

// 根据key计数
_.countBy = group(function(result, value, key) {
    if (_.has(result, key)) result[key]++; else result[key] = 1;
});
```
## delay defer

delay方法的作用是延迟一定时间调用函数，实现延迟的是定时器。

```javascript
_.delay = function(func, wait) {
    // 缓存原函数调用时需要的参数
    var args = slice.call(arguments, 2);
    return setTimeout(function(){
        return func.apply(null, args);
    }, wait);
};
```

在javascript中setTimeout更准确的功能描述是延迟一定时间把任务添加到任务队列中(而不是调用)，不过一般场景下这种细微的差别也不是太重要

defer的功能是保证一个函数异步调用。这个方法是基于delay方法利用偏函数生成工厂partial生成的

```javascript
_.defer = _.partial(_.delay, _, 1);
```

## after before once

这三个方法是控制函数调用次数的。after方法经过n次才会调用原来的函数，before是保证原函数最多调用不超过n次，once是before的特例，保证原函数只调用一次。

```javascript
_.after = function(times, func) {
    return function() {
        if (--times < 1) {
            return func.apply(this, arguments);
        }
    };
};
_.before = function(times, func) {
    // memo的作用是缓存最后一次调用结果，达到次数限制后就直接返回该结果
    var memo;
    return function() {
        if (--times > 0) {
            memo = func.apply(this, arguments);
        }
        if (times <= 1) func = null;
        return memo;
    };
};
// once是利用偏函数工厂生成的
_.once = _.partial(_.before, 2);
```

这几个方法都是高阶函数，将原函数加工。在具体实现上内部维护一个计数器统计被调用次数。


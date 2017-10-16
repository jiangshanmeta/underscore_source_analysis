throttle的功能是控制函数执行频率，使其在某个时间段内最多只能执行两次(含一次)。与debounce方法相对比，debounce是使函数在某个时间段内只执行一次，但是debounce的时间段的长短是不固定的，throttle的时间段是固定的。

underscore的throttle默认是原函数在时间段的两个边沿都可以被调用，我们可以通过设置属性改变这一行为：设置```{leading: false}```使得原函数只在时间段的结束边沿被调用，设置```{trailing: false}```使得原函数只在时间段的开始边沿被调用，如果leading和trailing都被设置为false，那么原函数根本不会被调用。

```javascript
_.throttle = function(func, wait, options) {
    var context, args, result;
    var timeout = null;
    var previous = 0;
    if (!options) options = {};
    var later = function() {
        previous = options.leading === false ? 0 : _.now();

        // 类似于debounce的timeout，它既保存定时器，又是是否设定新定时器的标志
        timeout = null;
        result = func.apply(context, args);

        // 这个判断有必要？
        // 把缓存的实行上下文和参数干掉，防止内存泄漏
        if (!timeout) context = args = null;
    };
    return function() {
        var now = _.now();

        if (!previous && options.leading === false) previous = now;
        var remaining = wait - (now - previous);

            // 缓存新函数被调用最后一次的执行上下文和参数
            context = this;
            args = arguments;

        // remaining>wait应该是考虑到了有人修改了本地时间
        if (remaining <= 0 || remaining > wait) {
            if (timeout) {
                clearTimeout(timeout);
                timeout = null;
            }
            previous = now;
            result = func.apply(context, args);
            if (!timeout) context = args = null;
        } else if (!timeout && options.trailing !== false) {
        // 结尾调用的定时器只设置一次就好，多次调用只改变执行上下文和参数
            timeout = setTimeout(later, remaining);
        }
        return result;
    };
};
```

对于```{leading: false}```的情况，在运行中remaining所处的区间是(0，wait]，这使得它不会立即被调用，而是走定时器的路线。对于```{trailing: false}```，根本不会走到设定定时器这一分支上，只能在开始边沿被调用。
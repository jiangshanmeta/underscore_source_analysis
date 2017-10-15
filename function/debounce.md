debounce的作用是在一个较短的时间内连续多次调用某个函数，只让其被调用一次。一般而言，和DOM相关的操作用这个方法的比较多，比如[element-ui的input-number组件就用它控制加减操作函数被调用次数](https://jiangshanmeta.gitbooks.io/elementui_source_analysis/content/input_number.html)。

underscore为这个方法提供了两种模式，一种是在时间段的开始边沿原函数被调用，一种是在结束边沿函数被调用(默认)。控制这两种模式的是immediate变量。


关于这个方法其实拿出笔画个时间轴比较好理解。以immediate为假值(默认条件)为例，在t0时刻调用debounce返回的新函数，则原函数最早调用时刻为t0+wait(wait是间隔时间),在原函数被调用之前任何一次调用新函数都会更新这个最终调用时间，换句话说原函数调用前wait时间没有调用过新函数。


```javascript
_.debounce = function(func, wait, immediate) {
    var timeout, args, context, timestamp, result;

    var later = function() {
        var last = _.now() - timestamp;

        // 时间间隔在[0,wait)时，说明没到触发时间，重设定时器
        // last正常情况下一定会大于0吧，除非有人修改本地时间
        if (last < wait && last >= 0) {
            timeout = setTimeout(later, wait - last);
        } else {
            // timeout一方面缓存定时器，一方面也标示是否需要添加定时器
            timeout = null;

            // 如果immediate为假值，则在结束边沿调用原函数
            if (!immediate) {
                result = func.apply(context, args);
                if (!timeout) context = args = null;
            }
        }
    };

    return function() {
        // 缓存函数被调用时的执行上下文和参数
        context = this;
        args = arguments;

        // 记录调用时的时间戳
        timestamp = _.now();

        // 如果immediate为真值，则在开始边沿调用原函数
        var callNow = immediate && !timeout;
        if (!timeout) timeout = setTimeout(later, wait);
        if (callNow) {
            result = func.apply(context, args);
            context = args = null;
        }

        return result;
    };
};
```
underscore的bind方法其实就是模拟Function原型对象上的bind方法，我在总结javascript的polyfill时，曾经提及过如何[polyfill bind方法](https://jiangshanmeta.gitbooks.io/javascript-polyfill/content/function/bind.html)。

bind方法实现的核心逻辑是利用apply方法绑定this，但是在调用apply时会直接调用函数，所以需要包一层函数bound。

```javascript
var executeBound = function(sourceFunc, boundFunc, context, callingContext, args) {
    // 正常调用函数，利用apply方法绑定this
    if (!(callingContext instanceof boundFunc)) return sourceFunc.apply(context, args);

    // 使用new关键字调用时，this不受bind的影响，指向一个新的实例
    var self = baseCreate(sourceFunc.prototype);
    var result = sourceFunc.apply(self, args);
    // 使用new关键字时，要返回一个对象类型
    if (_.isObject(result)) return result;
    return self;
};


_.bind = function(func, context) {
    // 如果有原生的方法就用原生的
    if (nativeBind && func.bind === nativeBind) return nativeBind.apply(func, slice.call(arguments, 1));
    if (!_.isFunction(func)) throw new TypeError('Bind must be called on a function');

    // bind方法可以预填充参数
    var args = slice.call(arguments, 2);

    // 绑定this的新函数
    var bound = function() {
        return executeBound(func, bound, context, this, args.concat(slice.call(arguments)));
    };
    return bound;
};
```

和MDN上bind方法的polyfill一比较，我们其实可以发现MDN上提供的方案有点小问题，就是如果我们使用new关键字调用时，函数可能是返回的不是一个对象类型(比如构造函数没有返回值则默认返回undefined)，underscore对此作了处理。不过通常使用场景下不会对bind返回的函数调用new关键字。
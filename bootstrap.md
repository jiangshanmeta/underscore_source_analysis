阅读一个库的源码是需要预备知识的，这里我不会去讲诸如闭包、立即执行的匿名函数之类的基础知识，我想说几点我之前没注意到的。

## 高阶函数

高阶函数本身是一个函数，它的参数中包含函数，或者返回一个函数。听起来比较抽象，我们举一个实际的例子吧：

```javascript
var property = function(key) {
    return function(obj) {
        return obj == null ? void 0 : obj[key];
    };
};
```

property函数就是一个高阶函数，他其实是一个函数生成工厂，生成的新函数的功能是返回传入对象的key属性，我们也不难看到闭包在这里的应用。使用示例如下：

```javascript
var getLength = property('length');
```

getLength函数underscore内部使用的一个方法，用于返回一个对象的length属性。

## 偏函数

偏函数是相对于原函数而言的，偏的意思是部分，原函数的部分参数或者变量被预置形成的新函数就是偏函数。underscore.js中提供了```_.partial```方法，它相当于一个偏函数工厂，原料是原函数，生成偏函数。看下代码：

```
var executeBound = function(sourceFunc, boundFunc, context, callingContext, args) {
    if (!(callingContext instanceof boundFunc)) return sourceFunc.apply(context, args);
    var self = baseCreate(sourceFunc.prototype);
    var result = sourceFunc.apply(self, args);
    if (_.isObject(result)) return result;
    return self;
};
_.partial = function(func) {
    var boundArgs = slice.call(arguments, 1);
    var bound = function() {
        var position = 0, length = boundArgs.length;
        var args = Array(length);
        for (var i = 0; i < length; i++) {
            args[i] = boundArgs[i] === _ ? arguments[position++] : boundArgs[i];
        }
        while (position < arguments.length) args.push(arguments[position++]);
        return executeBound(func, bound, this, this, args);
    };
    return bound;
};
```

可以看到，我们在partial中可以预填一些参数，如果想跳过某些参数可以用```_```占位，在执行的的时候会用相应的参数替换的。
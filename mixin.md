mixin方法允许我们扩展underscore

```javascript
_.mixin = function(obj) {
    _.each(_.functions(obj), function(name) {
        // 添加静态方法
        var func = _[name] = obj[name];
        // 添加原型方法，链式调用也可以用到
        // 虽然我一直认为underscore的链式调用真的没用
        _.prototype[name] = function() {
            var args = [this._wrapped];
            push.apply(args, arguments);
            return result(this, func.apply(_, args));
        };
    });
};
```

underscore的```_```其实是一个function，我们可以获取对应的实例，然后通过原型链调用方法(静态方法在原型对象上有对应方法)。因此，mixin功能不仅需要添加静态方法，还要在prototype上添加对方方法。

然而这个实现一个比较大的问题是我们可以轻易覆盖underscore提供的方法，这样是非常危险的，所以mixin逻辑中应该加上是否已有该方法的判断。在ES5时代完成该功能可以使用Object.defineProperty方法，置为不可写即可(然而underscore还要支持IE8)。
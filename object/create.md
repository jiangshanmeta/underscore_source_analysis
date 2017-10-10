javascript中有一个静态方法Object.create，它的作用是创建具有指定原型对象的新对象。

underscore有一个相同功能的函数，create:

```javascript
var Ctor = function(){};
var baseCreate = function(prototype) {
    if (!_.isObject(prototype)) return {};
    if (nativeCreate) return nativeCreate(prototype);
    Ctor.prototype = prototype;
    var result = new Ctor;
    Ctor.prototype = null;
    return result;
};

_.create = function(prototype, props) {
    var result = baseCreate(prototype);
    if (props) _.extendOwn(result, props);
    return result;
};
```

如果你了解过[Object.create的polyfill](https://jiangshanmeta.gitbooks.io/javascript-polyfill/content/object/create.html)，上面代码所做的没什么难以理解的，create主要所做的就是把传入的对象作为我们要返回对象的原型对象，因此需要借助Ctor这个构造函数进行关联。
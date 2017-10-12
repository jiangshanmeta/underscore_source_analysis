## mapObject

underscore虽然把map功能从数组扩展到了集合上，但是即使是对象，最终结果也是数组的形式，mapObject方法类似于map方法，只是最终结果以对象的形式呈现:

```javascript
_.mapObject = function(obj, iteratee, context) {
    iteratee = cb(iteratee, context);
    var keys =  _.keys(obj),
        length = keys.length,
        results = {},
        currentKey;
    for (var index = 0; index < length; index++) {
        currentKey = keys[index];
        results[currentKey] = iteratee(obj[currentKey], currentKey, obj);
    }
    return results;
};
```

这和实现和map的实现对比一下，除了map方法为了兼容数组和对象相关的代码，可以说是几乎一模一样。

## functions  

functions方法的作用是获取一个对象所有方法名，想实现这个功能也不难，遍历判断属性值是否是函数即可：

```javascript
_.functions = _.methods = function(obj) {
    var names = [];
    // 不仅仅是自身的方法，原型链上的方法也考虑进来了
    for (var key in obj) {
        if (_.isFunction(obj[key])) names.push(key);
    }
    return names.sort();
};
```

## pick omit

pick方法的作用是返回一个新对象，新对象所具有的属性根据传入的参数决定。它有两种模式，第一种是传入一个判断函数，根据这个函数的返回值决定新对象是否有这个属性，第二种是传入一组属性，新对象的属性由这一组属性决定。


```javascript
_.pick = function(object, oiteratee, context) {
    var result = {}, obj = object, iteratee, keys;
    if (obj == null) return result;
    // 传入判断函数
    if (_.isFunction(oiteratee)) {
        // 迭代的属性范围是全部属性，包括原型链上的属性
        keys = _.allKeys(obj);
        iteratee = optimizeCb(oiteratee, context);
    } else {
    // 传入属性模式
        keys = flatten(arguments, false, false, 1);
        // 该模式下仅需判断原始对象是否有对应属性
        iteratee = function(value, key, obj) { return key in obj; };
        obj = Object(obj);
    }
    for (var i = 0, length = keys.length; i < length; i++) {
        var key = keys[i];
        var value = obj[key];
        if (iteratee(value, key, obj)) result[key] = value;
    }
    return result;
};
```

omit方法是pick方法的取反，类似于reject和filter方法的关系，omit方法也利用了pick方法：

```javascript
_.omit = function(obj, iteratee, context) {
    // 传入判断函数模式，将传入函数包装成取反的函数
    if (_.isFunction(iteratee)) {
        iteratee = _.negate(iteratee);
    } else {
        // 传入属性模式，将其包装成传入判断函数模式
        var keys = _.map(flatten(arguments, false, false, 1), String);
        iteratee = function(value, key) {
            return !_.contains(keys, key);
        };
    }
    return _.pick(obj, iteratee, context);
};
```

## has

underscore的has方法几乎可以认为是原生的hasOwnProperty方法的别名：

```javascript
var hasOwnProperty = ObjProto.hasOwnProperty;
_.has = function(obj, key) {
    return obj != null && hasOwnProperty.call(obj, key);
};
```

对于一般的对象，has和直接使用hasOwnProperty没有区别，但是对于使用```Object.create(null)```创建出来的新对象，由于其原型链上没有hasOwnProperty方法，所以直接使用hasOwnProperty会出问题。

## property propertyOf

property方法我最开始提高阶函数的时候就说过了，它传入一个key，返回一个新函数，新函数的功能是返回传入对象的key属性的值。

propertyOf方法有点不知所云了，它是传入一个对象obj，返回一个新函数，新函数的功能是传入key返回对象key属性的值。正常情况下和直接获取对象的属性没什么区别，大概是考虑对象obj为非法值时的容错吧。


```javascript
var property = function(key) {
    return function(obj) {
        return obj == null ? void 0 : obj[key];
    };
};
_.property = property;

_.propertyOf = function(obj) {
    return obj == null ? function(){} : function(key) {
        return obj[key];
    };
};
```
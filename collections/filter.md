筛选其实可以分为许多子问题：找到所有满足/不满足条件的元素，找到第一个满足条件的元素/索引，查看是否全部/个别元素满足某个条件。

从现在开始iteratee有了一个更为贴切的称呼predicate，毕竟筛选需要判断。

## 筛出所有满足/不满足条件的元素

#### filter

filter其实类似于map方法，只不过map方法是把所有的iteratee返回的结果都放到数组中，filter是把iteratee方法返回真的时候对应的元素加入到数组中。

```javascript
_.filter = _.select = function(obj, predicate, context) {
    var results = [];
    predicate = cb(predicate, context);
    _.each(obj, function(value, index, list) {
        if (predicate(value, index, list)) results.push(value);
    });
    return results;
};
```

#### reject

reject方法是用来获取所有不满足条件的元素的。有了filter方法，似乎我们只需要复制粘贴然后修改一下判断就可以得到reject方法了，但是underscore采用了negate方法，将predicate方法加工得到了一个功能相反的函数。negate方法就是最开始提到的高阶函数，它的参数是一个函数，返回的也是一个函数，新函数的功能的是调用时返回原函数调用结果的相反值。

```javascript
_.negate = function(predicate) {
    return function() {
        return !predicate.apply(this, arguments);
    };
};

_.reject = function(obj, predicate, context) {
    return _.filter(obj, _.negate(cb(predicate)), context);
};
```


#### where

where方法其实就是filter方法的一种特殊情况，它限定了传入的predicate是一个对象：

```javascript
_.where = function(obj, attrs) {
    return _.filter(obj, _.matcher(attrs));
};
```


#### partition

partition方法的作用是根据是否满足条件尽心分类。其实我们实现了filter方法，我们就可以实现按照是否满足某个条件分类。filter方法是只保留满足条件的元素，而分类要求我们既保留满足条件的，又保留不满足条件的，只是两者存放的位置不一样。

在underscore中，完成这个任务的是partition方法：

```javascript
_.partition = function(obj, predicate, context) {
    predicate = cb(predicate, context);
    var pass = [], fail = [];
    _.each(obj, function(value, key, obj) {
        // 根据是否满足条件，将元素放入不同的数组中
        (predicate(value, key, obj) ? pass : fail).push(value);
    });
    return [pass, fail];
};
```


## 找到第一个满足条件的元素/索引

要找到第一个满足条件的元素，我们首先要找到其对应的索引。由于数组和对象的差异，underscore对这两种情况是分别处理的。

对于对象：

```javascript
_.findKey = function(obj, predicate, context) {
    predicate = cb(predicate, context);
    // 按照for in 的顺序搜索对象的属性
    var keys = _.keys(obj), key;
    for (var i = 0, length = keys.length; i < length; i++) {
        key = keys[i];
        // 第一个满足条件的返回对应的key
        if (predicate(obj[key], key, obj)) return key;
    }
    // 没有找到默认返回undefined
};
```

然而对于对象，查询的顺序是按照Object.key返回数组的顺序来的，再实质一点是按照for in的顺序来的，但是不同浏览器for in的顺序又不一致，所以我一直很怀疑这个方法的现实意义。

对于数组，类似于reduce和reduceRight方法，从数组中查找元素也是有先后顺序的，所以underscore按照同样的套路实现了一个用于处理不同迭代方向的高阶函数createPredicateIndexFinder

```javascript
function createPredicateIndexFinder(dir) {
    return function(array, predicate, context) {
        predicate = cb(predicate, context);
        var length = getLength(array);
        var index = dir > 0 ? 0 : length - 1;
        for (; index >= 0 && index < length; index += dir) {
            // 找到第一个满足条件的元素 返回对应下标
            if (predicate(array[index], index, array)) return index;
        }
        // 查询不到返回-1，类似于原生的indexOf的返回值
        return -1;
    };
}

_.findIndex = createPredicateIndexFinder(1);
_.findLastIndex = createPredicateIndexFinder(-1);
```

关于数组寻找第一个满足条件的方法，还有indexOf、lastIndexOf、sortedIndex、contains这么几个，我会在讲数组方法时再提这几个方法。

有了查询索引的方法，根据索引查询值就是水到渠来的事：

```javascript
_.find = _.detect = function(obj, predicate, context) {
    var key;
    if (isArrayLike(obj)) {
        key = _.findIndex(obj, predicate, context);
    } else {
        key = _.findKey(obj, predicate, context);
    }
    // 对于数组、类数组，查询不到返回-1，对于对象查询不到返回undefined
    if (key !== void 0 && key !== -1) return obj[key];
};
```


## 查看是否全部/个别元素满足某个条件

在javascript的数组原型对象上，有every和some两个方法，underscore将这两个功能扩展到了对象上

```javascript
_.every = _.all = function(obj, predicate, context) {
    predicate = cb(predicate, context);
    var keys = !isArrayLike(obj) && _.keys(obj),
        length = (keys || obj).length;
    for (var index = 0; index < length; index++) {
        // 兼容数组和对象
        var currentKey = keys ? keys[index] : index;

        // 一个不满足就返回false
        if (!predicate(obj[currentKey], currentKey, obj)) return false;
    }

    // 所有元素都满足才返回true
    return true;
};

_.some = _.any = function(obj, predicate, context) {
    predicate = cb(predicate, context);
    var keys = !isArrayLike(obj) && _.keys(obj),
        length = (keys || obj).length;
    for (var index = 0; index < length; index++) {
        // 兼容数组和对象
        var currentKey = keys ? keys[index] : index;

        // 一个满足就返回true
        if (predicate(obj[currentKey], currentKey, obj)) return true;
    }

    // 所有元素都不满足条件才返回false
    return false;
};
```

every方法要求集合中所有元素都满足条件才返回true，而some方法只要求一个元素满足条件就返回true。

underscore中和数组相关的方法基本差不多了，就剩下几个小方法了。

## first initial last rest

这几个方法是用来获取数组的子集的。前两个方法是获取头几个元素，后两个方法是获取后几个元素。

```javascript
_.first = _.head = _.take = function(array, n, guard) {
    if (array == null) return void 0;
    // 没有指定获取几个，返回第一个元素
    if (n == null || guard) return array[0];
    // 返回头n个元素
    return _.initial(array, array.length - n);
};

_.initial = function(array, n, guard) {
    // 利用slice方法获取头n个元素
    return slice.call(array, 0, Math.max(0, array.length - (n == null || guard ? 1 : n)));
};


_.last = function(array, n, guard) {
    if (array == null) return void 0;
    // 没指定获取几个，返回最后一个元素
    if (n == null || guard) return array[array.length - 1];
    // 获取后n个元素
    return _.rest(array, Math.max(0, array.length - n));
};

// 与initial方法类似，获取从n开始的元素
_.rest = _.tail = _.drop = function(array, n, guard) {
    return slice.call(array, n == null || guard ? 1 : n);
};
```

## object

这个方法如果只传入第一个参数，可以理解为是Object.entries的逆方法。如果传入第二个数组，会将两个数组合并为一个对象，第一个数组的元素做key，第二个数组的元素做value。

```javascript
_.object = function(list, values) {
    var result = {};
    for (var i = 0, length = getLength(list); i < length; i++) {
        //将两个数组合并为一个对象
        if (values) {
            result[list[i]] = values[i];
        } else {
        // Object.entries方法的逆方法
            result[list[i][0]] = list[i][1];
        }
    }
    return result;
};
```


数组原型上有indexOf和lastIndexOf两个方法，用来查找元素在数组中的索引。underscore也实现了同名方法，功能也是基本上一致(可以理解为是实现了这两个方法的polyfill)，略有不同的一点是原生的方法无法正确处理```NaN```这一特殊情况(内部判断相等用的是```===```，而```NaN```真的不等于它本身，ES6提供的新方法includes修正了这一问题)，还有一点是正序查找时，如果我们知道数组是按照从小到大已排好序，那么会采用二分查找的方式优化。

由于indexOf和lastIndexOf实现上有许多相似之处，类似于之前提及的findIndex和findLastIndex的关系，于是underscore按照套路写了一个高阶函数。

```javascript
function createIndexFinder(dir, predicateFind, sortedIndex) {
    return function(array, item, idx) {
        var i = 0, length = getLength(array);
        if (typeof idx == 'number') {
            if (dir > 0) {
                // 正向查询，i是起始索引
                i = idx >= 0 ? idx : Math.max(idx + length, i);
            } else {
                // 看起来是重设长度，更好地理解方式是设置终止的索引(从右向左看似乎叫起始索引更合适)
                length = idx >= 0 ? Math.min(idx + 1, length) : idx + length + 1;
            }
        } else if (sortedIndex && idx && length) {
            // 正序查找且已排序的数组，使用二分查找优化
            idx = sortedIndex(array, item);
            return array[idx] === item ? idx : -1;
        }

        // 对于NaN的情况特殊处理
        if (item !== item) {
            idx = predicateFind(slice.call(array, i, length), _.isNaN);
            return idx >= 0 ? idx + i : -1;
        }
        // 正常的数据，遍历比较即可
        for (idx = dir > 0 ? i : length - 1; idx >= 0 && idx < length; idx += dir) {
            if (array[idx] === item) return idx;
        }
        // 找不到元素返回-1，与原生的方法行为一致
        return -1;
    };
}
_.indexOf = createIndexFinder(1, _.findIndex, _.sortedIndex);
_.lastIndexOf = createIndexFinder(-1, _.findLastIndex);
```

一个相关的方法是contains，他其实是语义化的indexOf方法，有一个对应的数组的原生方法叫includes。

```javascript
_.contains = _.includes = _.include = function(obj, item, fromIndex, guard) {
    if (!isArrayLike(obj)) obj = _.values(obj);
    if (typeof fromIndex != 'number' || guard) fromIndex = 0;
    return _.indexOf(obj, item, fromIndex) >= 0;
};
```

虽然underscore把contains方法归类为集合相关方法，但是我依然想把它看成是数组相关方法，毕竟对于对象，contains是先将其转换为数组，然后按照数组去处理的。
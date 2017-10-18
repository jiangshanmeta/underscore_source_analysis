二分查找适用于有序序列，它的原理是这样的：在序列中找到中间值，通过中间值和查询值进行比较排除出一半的元素，再在剩余的一半有序序列中查找。二分查找的时间复杂度是O(logn)。

underscore的sortIndex用到了二分法的原理，但是sortIndex的作用是找到一个位置，将查询值插入到该位置不会影响序列的有序性。

```javascript
_.sortedIndex = function(array, obj, iteratee, context) {
    iteratee = cb(iteratee, context, 1);
    var value = iteratee(obj);
    var low = 0, high = getLength(array);
    while (low < high) {
        // 中间值的索引
        var mid = Math.floor((low + high) / 2);

        // 中间值与查询值进行比较
        if (iteratee(array[mid]) < value) low = mid + 1; else high = mid;
    }
    return low;
};
```

严格来说sortIndex找的是从小到大第一个满足插入该位置不影响有序性的位置，这个限制有点严格了。对于有多个插入不改变有序性的位置情况，我个人认为任何一个位置都可以，对应代码中比较时加一个等于的判断就可以。
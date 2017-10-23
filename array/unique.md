数组的去重是个经典问题了，在这里我想首先谈一下underscore的实现方案，然后再谈几个相关的方案


## underscore去重

underscore的去重方案其实是最朴素的方案，我们找一个新数组，然后去遍历原数组，如果新数组没有当前被迭代元素就把当前被迭代元素放入新数组。很显然，这个方案的时间复杂度是O(n2)。对于已排序的序列，underscore还做了优化，主要是利用了已排序序列相同元素位置相邻这一特性。


```javascript
_.uniq = _.unique = function(array, isSorted, iteratee, context) {
    if (!_.isBoolean(isSorted)) {
        context = iteratee;
        iteratee = isSorted;
        isSorted = false;
    }
    if (iteratee != null) iteratee = cb(iteratee, context);
    var result = [];
    var seen = [];
    for (var i = 0, length = getLength(array); i < length; i++) {
        var value = array[i],
        computed = iteratee ? iteratee(value, i, array) : value;
        if (isSorted) {
            // 已排序数组，和前一个元素比较就能知道是否重复
            if (!i || seen !== computed) result.push(value);
            seen = computed;
        } else if (iteratee) {
            // 遍历查找是否有重复元素
            if (!_.contains(seen, computed)) {
                seen.push(computed);
                result.push(value);
            }
        } else if (!_.contains(result, value)) {
            result.push(value);
        }
    }
    return result;
};
```


## 利用哈希去重

利用哈希去重并不是一种通用的方案。传统上在javascript中模拟哈希结构需要利用对象(想用ES6提供的Map的请看下面的ES6方案)，但是适用对象有个严重的问题是键的类型都会被转换为字符串，像```[1,'1']```，这样的数组就不能使用哈希结构去重了。这一般要求数组结构比较规整(对于一般的业务逻辑来说符合这个条件似乎没什么难度)。

下面是这个方案对应的一个实现：

```javascript
function unique(arr,hasher){
    if(!hasher){
        hasher = function(value){
            return value
        }
    }
    var obj = {};
    arr.forEach(function(item){
        var key = hasher(item);
        obj[key] = item;
    })
    var rst = [];
    var keys = Object.keys(obj);
    keys.forEach(function(item){
        rst.push(obj[item]);
    })
    return rst;
}
```



## ES6去重

利用ES6提供的API实现数组去重非常简单：

```javascript
function unique(arr){
    return Array.from(new Set(arr));
}
```

这主要是利用了集合中的元素不重复这一特性。
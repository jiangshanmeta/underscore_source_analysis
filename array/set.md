集合的并集、交集、差集的数学含义大家应该很明白了。underscore用数组模拟集合，实现了这三个基本运算。

## 并集

并集是把所有集合的元素合并在一起。underscore很巧妙的实用到了数组展开这一方法，将集合的元素合并到一起，合并到一起之后再去重保证集合的性质。

```javascript
_.union = function() {
    // arguments的每一项应该对应一个集合(set)
    // 先将arguments展开，所有元素放到一个数组中
    // 然后去重，保证集合数据不重复这一特征
    return _.uniq(flatten(arguments, true, true));
};
```



## 交集

交集的意思是所有集合都含有的元素，它一定是第一个集合的子集。

```javascript
_.intersection = function(array) {
    var result = [];
    var argsLength = arguments.length;
    // 交集的话一定是第一个集合的子集
    for (var i = 0, length = getLength(array); i < length; i++) {
        var item = array[i];
        // 考虑到array中有重复元素
        if (_.contains(result, item)) continue;

        // 遍历其余集合，查看其他集合是否含有该元素
        for (var j = 1; j < argsLength; j++) {
            if (!_.contains(arguments[j], item)) break;
        }
        // j 如果不等于 argsLength，说明提前退出循环，有一个集合不含该元素
        if (j === argsLength) result.push(item);
    }
    return result;
};
```

## 差集

差集的含义是第一个集合有的其他集合没有的元素。

```javascript
_.difference = function(array) {
    // 将其他集合的元素合并
    var rest = flatten(arguments, true, true, 1);
    // 筛选出其他集合没有的元素
    return _.filter(array, function(value){
        return !_.contains(rest, value);
    });
};
```
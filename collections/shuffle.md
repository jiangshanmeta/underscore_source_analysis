在PHP中有个函数```shuffle```，它的作用是将数组打乱顺序。underscore实现了相同功能的同名函数，它采用了**Fisher–Yates Shuffle**算法。

```javascript
_.shuffle = function(obj) {
    // 保证是针对数组乱序
    var set = isArrayLike(obj) ? obj : _.values(obj);
    var length = set.length;

    var shuffled = Array(length);
    for (var index = 0, rand; index < length; index++) {
        rand = _.random(0, index);

        // 遍历时 与之前的随机元素交换
        if (rand !== index) shuffled[index] = shuffled[rand];
        huffled[rand] = set[index];
    }
    return shuffled;
};
```

这个算法可以这么理解：遍历数组，将其与该元素之前的随机元素交换，这样的得到的新数组就是乱序排列的了。这个算法的时间复杂度是O(n)，因为只循环一次。

在phpjs这个项目中，作者使用了sort方法：

```javascript
valArr.sort(function () {
    return 0.5 - Math.random()
})
```

这个算法看起来很巧妙，但是算法是有问题的。[这篇文章](https://www.h5jun.com/post/array-shuffle.html)指出，用 JavaScript 内置的排序算法(快排、冒泡、插入)这么排序，通常肯定是不完全随机的。从时间复杂度上，也是underscore的最好，因为即使是快速排序，其时间复杂度也是O(nlogn)。
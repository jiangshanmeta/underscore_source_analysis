数组展开的意思是将数组中的元素从层层嵌套中剥离出来，减少嵌套层数。在我有限的经验中没有直接用到过这个方法，但是在underscore的并集、差集运算中用到了这一方法。我们就直接看源码实现吧：

```javascript
var flatten = function(input, shallow, strict, startIndex) {
    // input是一个要展开的数组/类数组
    // output是最终结果
    // startIndex是起始索引，默认为0
    var output = [], idx = 0;
    for (var i = startIndex || 0, length = getLength(input); i < length; i++) {
        var value = input[i];
        // 严格模式只考虑数组/类数组类型的元素
        if (isArrayLike(value) && (_.isArray(value) || _.isArguments(value))) {
            // shallow控制是否深度展开
            if (!shallow) value = flatten(value, shallow, strict);
            var j = 0, len = value.length;

            // 知道数组长度，用下标赋值，比push性能好
            output.length += len;
            while (j < len) {
                output[idx++] = value[j++];
            }
        } else if (!strict) {
        // 非严格模式直接把非数组/类数组类型元素添加进去
            output[idx++] = value;
        }
    }
    return output;
};
// 对外暴露的flatten方法，指定非严格模式
_.flatten = function(array, shallow) {
    return flatten(array, shallow, false);
};
```
_.keys、_.values和_.pairs这三个方法其实是对应Object.keys、Object.values、Object.entries这三个标准方法。Object.keys是es5就提出的方法，目前来说可以大胆使用了，后两个有严重的兼容性问题，一般是经过babel编译后使用。

之前我在看[javascript的polyfill](https://jiangshanmeta.gitbooks.io/javascript-polyfill/content/object/entries.html)时，已经提及过这几个方法如何模拟了，其基本思路是利用```for in```循环和```hasOwnProperty```方法。underscore也是基于同样的思路。

```javascript
var nativeKeys = Object.keys;
_.keys = function(obj) {
    if (!_.isObject(obj)) return [];
    if (nativeKeys) return nativeKeys(obj);
    var keys = [];
    for (var key in obj) if (_.has(obj, key)) keys.push(key);
    // Ahem, IE < 9.
    if (hasEnumBug) collectNonEnumProps(obj, keys);
    return keys;
};
_.values = function(obj) {
    var keys = _.keys(obj);
    var length = keys.length;
    var values = Array(length);
    for (var i = 0; i < length; i++) {
        values[i] = obj[keys[i]];
    }
    return values;
};
_.pairs = function(obj) {
    var keys = _.keys(obj);
    var length = keys.length;
    var pairs = Array(length);
    for (var i = 0; i < length; i++) {
        pairs[i] = [keys[i], obj[keys[i]]];
    }
    return pairs;
};
```

与_.keys相类似的一个方法是_.allKeys，两者的区别是前者只考虑自身可枚举属性，后者考虑到了原型链上的可枚举属性。

```javascript
_.allKeys = function(obj) {
    if (!_.isObject(obj)) return [];
    var keys = [];
    for (var key in obj) keys.push(key);
    // Ahem, IE < 9.
    if (hasEnumBug) collectNonEnumProps(obj, keys);
    return keys;
};
```

在原生API中还有一个```Object.getOwnPropertyNames```，它与_.keys类似，都是返回自身属性，但是```Object.getOwnPropertyNames```无论是可枚举还是不可枚举的都返回。
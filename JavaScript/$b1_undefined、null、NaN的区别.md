#### undefined、null与NaN的区别
##### 1. 含义
undefined: 表示变量应该有值，但未赋值。其类型是`undefined`。

null: 表示定义了一个空对象 ( 内存地址指向为空 )。其类型是`object`。

NaN: 表示非数字 ( Not a number )。其类型是`number`。( Q: 既然表示非数字，为何其类型却是number？ )

##### 2. typeof、\==、===的区别
```javascript
/** typeof */
typeof undefined;     // undefined
typeof null;          // object
typeof NaN;           // number

/** == */
undefined == null;   // true
undefined == NaN/** NaN跟NaN都不相等，跟其他值当然更不相等啦 **/;  // false
NaN == null;         // false

/** === */
undefined === null;  // false
undefined === NaN;   // false
NaN === null;        // false
```

##### 3. JSON.stringify、toString的区别
```javascript
const obj = { property1: undefined, property2: null, property3: NaN };
const arr = [ undefined, null, NaN ];

/** JSON */
console.log(JSON.stringify(obj));   // {"[property2":null,"[property3":null,}
console.log(JSON.stringify(arr));   // [null,null,null]

/** toString */
console.log(obj.toString());        // [object Bbject]
console.log(arr.toString());        // [,,NaN]
```
因`JSON.stringify`、`Array.prototype.toString`中的参数中有undefined、null、NaN时，<font color="red">结果可能非我们预期</font>。故使用`JSON.stringfy`深拷贝对象或使用`Array.prototype.toString`扁平化数组时，应评估其结果的影响。
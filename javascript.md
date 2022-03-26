## 数据类型
### 对象

对象的所有键名都是字符串（ES6 又引入了 Symbol 值也可以作为键名），所以加不加引号都可以。如果键名是数值，会被自动转为字符串。
如果键名不符合标识符的条件（比如第一个字符为数字，或者含有空格或运算符），且也不是数字，则必须加上引号，否则会报错。

```js
eval('{foo: 123}') // 123
eval('({foo: 123})') // {foo: 123}
```

```js
var obj = {
  key1: 1,
  key2: 2
};

Object.keys(obj); // ['key1', 'key2']
```

`delete` 命令用于删除对象的属性，删除成功后返回 `true`；删除一个不存在的属性，`delete` 不报错，而且返回 `true`，因此不能根据 `delete` 命令的结果，认定某个属性是存在的；只有一种情况，`delete` 命令会返回 `false`，那就是该属性存在，且不能被删除。

```js
var obj = Object.defineProperty({}, 'p', {
  value: 123,
  configurable: false
});

obj.p // 123
delete obj.p // false
```

此外，`delete` 命令只能删除对象本身的属性，无法删除继承的属性。

```js
var obj = {};
delete obj.toString // true
obj.toString // function toString() { [native code] }
```

### 函数

可以传递任意数量的参数给 `Function` 构造函数，只有最后一个参数会被当做函数体，如果只有一个参数，该参数就是函数体。

```js
var foo = new Function(
  'x',
  'y',
  'return x + y'
);

// 等同于
function add(x, y) {
  return x + y;
}
```

如果同一个函数被多次声明，后面的声明就会覆盖前面的声明。而且，**由于函数名的提升，前一次声明在任何时候都是无效的**。JavaScript 引擎将函数名视同变量名，所以采用 `function` 命令声明函数时，整个函数会像变量声明一样，被提升到代码头部。

```js
function f() {
  console.log(1);
}
f() // 2

function f() {
  console.log(2);
}
f() // 2
```

```js
f();
var f = function (){};
// TypeError: undefined is not a function

// 等同于
var f;
f();
f = function (){};
```

函数的 `length` 属性返回函数**预期传入**的参数个数，即函数定义之中的参数个数；`arguments` 对象包含了函数运行时的所有参数，这个对象只有在函数体内部，才可以使用；`arguments` 虽然很像数组，但它是一个对象；`arguments` 对象带有一个 `callee` 属性，返回它所对应的原函数。

```js
var f = function () {
  console.log(arguments.callee === f);
}

f() // true
```

函数的 `toString()` 方法返回一个字符串，内容是函数的源码。

与全局作用域一样，函数作用域内部也会产生“变量提升”现象。`var` 命令声明的变量，不管在什么位置，**变量声明都会被提升到函数体的头部**。
函数的作用域与变量一样，就是**其声明时所在的作用域，与其运行时所在的作用域无关**。

`eval` 没有自己的作用域，都在当前作用域内执行，因此可能会修改当前作用域的变量的值，造成安全问题。

#### 闭包

JavaScript 有两种作用域：全局作用域和函数作用域。函数内部可以直接读取全局变量。
但是，正常情况下，函数外部无法读取函数内部声明的变量。
如果出于种种原因，需要得到函数内的局部变量。正常情况下，这是办不到的，只有通过变通方法才能实现。那就是在函数的内部，再定义一个函数。
内部函数可以获取外部函数的局部变量；这是 JavaScript 特有的“链式作用域”结构，子对象会一级一级地向上寻找所有父对象的变量；所以，父对象的所有变量，对子对象都是可见的，反之则不成立。

闭包的最大用处有两个，一个是可以读取外层函数内部的变量，另一个就是让这些变量始终保持在内存中，即闭包可以使得它诞生环境一直存在。
闭包函数用到了外层变量，导致外层函数不能从内存释放。只要闭包函数没有被垃圾回收机制清除，外层函数提供的运行环境也不会被清除，它的内部变量就始终保存着当前值，供闭包读取。

闭包的另一个用处，是封装对象的私有属性和私有方法。外层函数每次运行，都会产生一个新的闭包函数，而这个闭包函数又会保留外层函数的内部变量，所以内存消耗很大。因此不能滥用闭包，否则会造成网页性能问题。

### 数组

本质上，数组属于一种特殊的对象。`typeof` 运算符会返回数组的类型是 `object`。JavaScript 语言规定，对象的键名一律为字符串，所以，数组的键名其实也是字符串。之所以可以用数值读取，是因为非字符串的键名会被转为字符串。值得注意的是，由于数组本质上是一种对象，所以可以为数组添加属性，但是这不影响 `length` 属性的值。

检查某个键名是否存在的运算符 `in`，适用于对象，也适用于数组。如果数组的某个位置是空位，`in` 运算符返回 `false`。

## 运算符

### 加法运算符

加法运算符是在运行时决定，到底是执行相加，还是执行连接。如果是两个字符串相加，这时加法运算符会变成连接运算符，返回一个新的字符串，将两个原字符串连接在一起。除了加法运算符，其他算术运算符（比如减法、除法和乘法）都不会发生重载。它们的规则是：**所有运算子一律转为数值，再进行相应的数学运算**。

如果运算子是对象，必须先转成原始类型的值，然后再相加。
对象转成原始类型的值，规则如下：

- 首先，自动调用对象的 `valueOf` 方法

```js
var obj = { p: 1 };
obj.valueOf() // { p: 1 }
```

一般来说，对象的 `valueOf` 方法总是返回对象自身，这时再自动调用对象的 `toString` 方法，将其转为字符串；对象的 `toString` 方法默认返回 `[object Object]`。

```js
var obj = { p: 1 };
obj.valueOf().toString() // "[object Object]"
```

如果运算子是一个 `Date` 对象的实例，那么会优先执行 `toString` 方法。

### 余数运算符

余数运算符（`%`）返回前一个运算子被后一个运算子除，所得的余数；**运算结果的正负号由第一个运算子的正负号决定**。

### 指数运算符

**指数运算符是右结合，而不是左结合。即多个指数运算符连用时，先进行最右边的计算。**

### 比较运算符

算法是先看两个运算子是否都是字符串，如果是的，就按照字典顺序比较（实际上是比较 Unicode 码点）；否则，将两个运算子都转成数值，再比较数值的大小。

**字符串和布尔值都会先转成数值，再进行比较。**

## 错误处理机制

JavaScript 解析或运行时，一旦发生错误，引擎就会抛出一个错误对象。JavaScript 原生提供 `Error` 构造函数，所有抛出的错误都是这个构造函数的实例。

JavaScript 语言标准只提到，`Error` 实例对象必须有 `message` 属性，表示出错时的提示信息。

- message：错误提示信息
- name：错误名称（非标准属性）
- stack：错误的堆栈（非标准属性）

原生错误类型：

- SyntaxError

解析代码时发生的语法错误；一般在开发阶段就可以发现，线上多半是因为 `JSON` 解析出错和浏览器兼容性问题抛出此错误；
`window.onerror` 和 `try...catch` 捕获不到 `SyntaxError`；

- ReferenceError

引用一个不存在的变量时发生的错误

- RangeError

一个值超出有效范围时发生的错误

- TypeError

变量或参数不是预期类型时发生的错误

- URIError

URI 相关函数的参数不正确时抛出的错误，主要涉及 `encodeURI()`, `decodeURI()`, `encodeURIComponent()`, `decodeURIComponent()`, `escape()` 和 `unescape()`

- EvalError

`eval` 函数没有被正确执行时，会抛出 `EvalError` 错误。该错误类型已经不再使用，只是为了保证与以前代码兼容，才继续保留。

### 前端监控原理

#### 性能监控

#### 异常监控

- `try...catch`

可以捕获常规运行时错误，对**语法错误**和**异步错误**无能为力

- `window.onerror`

可以捕获常规运行时错误、**异步错误**，不能捕获**语法错误**和**资源错误**；可以获得错误发生的行号、列号和堆栈信息。

```js
window.onerror = function(msg, url, line, column, error) {}
```

- `window.addEventListener('error')` 或 `obj.onerror`

当一项资源（如图片、脚本或样式）加载失败，加载的资源元素会触发一个 Event 接口的 `error` 事件，这些 `error` 事件不会向上冒泡到 window，但能被捕获。而 `window.onerror` 不能监测捕获。

由于**网络请求异常不会事件冒泡**，因此必须在捕获阶段将其捕捉到。

```js
window.addEventListener('error', err => {
  console.log('捕获到异常：', err);
}, true)
```

局限性：

> + `new Image()` 错误不能被捕获
> 
> ```html
> // new Image 错误，不能捕获 ❌
> <script>
>   window.addEventListener('error', err => {
>     console.log('捕获到异常：', error);
>   }, true)
> </script>
> <script>
>   new Image().src = 'https://yun.tuia.cn/image/lll.png';
> </script>
> ```
> 
> + `fetch` 错误不能被捕获
> 
> ```html
> // fetch 错误，不能捕获 ❌
> <script>
>   window.addEventListener('error', err => {
>     console.log('捕获到异常：', error);
>   }, true)
> </script>
> <script>
>   fetch('https://tuia.cn/test')
> </script>
> ```

- `window.addEventListener('unhandledrejection')`

用于捕获 Promise 类型错误，可以用来全局监听 Uncaught Promise Error（捕获到未处理的 promise 错误）

- react `componentDidCatch`

捕获 react 组件错误，不会捕获 React 事件处理、异步代码、“error boundaries” 自己抛出的错误



捕获到的异常信息为`script error`时，一般是脚本存在跨域问题

`XMLHttpRequest` 可以重写 `onerror` 函数

<https://mp.weixin.qq.com/s/TzTmu5FS5XGDgQKcLbNqgw>
<https://mp.weixin.qq.com/s/O2ANZE1vYG-DT6eCB_hf4A>

#### 数据上报

1. 为什么不能直接用 GET/POST/HEAD 请求接口进行上报？

一般而言，上报的域名都不是当前域名，所以所有的接口请求都会容易出现跨域问题。

2. 为什么不能用请求其他的文件资源（js/css/ttf）的方式进行上报？

创建资源节点后只有将对象注入到浏览器DOM树后，浏览器才会实际发送资源请求。而且载入js/css资源还会阻塞页面渲染，影响用户体验。

构造图片打点（上传）不仅不用插入DOM，只要在js中new出Image对象就能发起请求，而且还没有阻塞问题，在没有js的浏览器环境中也能通过img标签正常打点。

3. 为什么上报时选用1x1的透明gif，而不是其他的png/jpeg/bmp格式图片

首先，1x1像素是最小的合法图片。而且，因为是通过图片打点，所以图片最好是透明的，这样一来不会影响页面本身展示效果，二来表示图片透明只需要使用一个二进制标记图片是透明色即可，不用存储色彩空间数据，可以节约体积。因为需要透明色，所以可以直接排除jpeg。

同样的响应，gif可以比bmp节约40%的流量，比png节约35%的流量，所以gif才是最佳选择。

### Console 对象

- console.assert()

`console.assert`方法主要用于程序运行过程中，进行条件判断，如果不满足条件，就显示一个错误，但**不会中断程序执行**。

它接受两个参数，第一个参数是表达式，第二个参数是字符串。只有当第一个参数为`false`，才会提示有错误，在控制台输出第二个参数，否则不会有任何结果。

## 标准库

### Object 对象

JavaScript 的所有其他对象都继承自`Object`对象，即那些对象都是`Object`的实例。

`Object`对象的原生方法分成两类：`Object`本身的方法（静态方法）与`Object`的实例方法。

- `Object`对象本身的方法（静态方法）

```js
console.log(Object)
// Object()
//   assign: ƒ assign()
//   create: ƒ create()
//   defineProperties: ƒ defineProperties()
//   defineProperty: ƒ defineProperty()
//   entries: ƒ entries()
//   freeze: ƒ freeze()
//   fromEntries: ƒ fromEntries()
//   getOwnPropertyDescriptor: ƒ getOwnPropertyDescriptor()
//   getOwnPropertyDescriptors: ƒ getOwnPropertyDescriptors()
//   getOwnPropertyNames: ƒ getOwnPropertyNames()
//   getOwnPropertySymbols: ƒ getOwnPropertySymbols()
//   getPrototypeOf: ƒ getPrototypeOf()
//   hasOwn: ƒ hasOwn()
//   is: ƒ is()
//   isExtensible: ƒ isExtensible()
//   isFrozen: ƒ isFrozen()
//   isSealed: ƒ isSealed()
//   keys: ƒ keys()
//   length: 1
//   name: "Object"
//   preventExtensions: ƒ preventExtensions()
//   prototype: {constructor: ƒ, __defineGetter__: ƒ, __defineSetter__: ƒ, hasOwnProperty: ƒ, __lookupGetter__: ƒ, …}
//   seal: ƒ seal()
//   setPrototypeOf: ƒ setPrototypeOf()
//   values: ƒ values()
```

- `Object`的实例方法

指定义在`Object`原型对象`Object.prototype`上的方法，它将被所有实例对象共享。

```js
console.log(Object.prototype)
// constructor: ƒ Object()
// hasOwnProperty: ƒ hasOwnProperty()
// isPrototypeOf: ƒ isPrototypeOf()
// propertyIsEnumerable: ƒ propertyIsEnumerable()
// toLocaleString: ƒ toLocaleString()
// toString: ƒ toString()
// valueOf: ƒ valueOf()
// __defineGetter__: ƒ __defineGetter__()
// __defineSetter__: ƒ __defineSetter__()
// __lookupGetter__: ƒ __lookupGetter__()
// __lookupSetter__: ƒ __lookupSetter__()
// __proto__: （…）
// get __proto__: ƒ __proto__()
// set __proto__: ƒ __proto__()
```

`instanceof`运算符用来验证，一个对象是否为指定的**构造函数**的实例。

```js
({}) instanceof Object;          // true
Function instanceof Object;      // true
Array instanceof Object;         // true
Boolean instanceof Object;       // true
String instanceof Object;        // true

// 将`undefined`和`null`转为对象，得到一个空对象
Object(null);      // {}
Object(undefined); // {}

// 如果参数是原始类型的值，`Object`方法将其转为对应的包装对象的实例
Object(1) instanceof Number;     // true
Object(true) instanceof Boolean; // true

// 如果参数是一个对象，它总是返回该对象，即不用转换
var arr = [];
Object(arr) === arr; // true
```



#### Object的静态方法

`Object.keys`方法只返回可枚举的属性，`Object.getOwnPropertyNames`方法还返回不可枚举的属性名。

1. 对象属性模型的相关方法
   - `Object.getOwnPropertyDescriptor()`：获取某个属性的描述对象
   - `Object.defineProperty()`：通过描述对象，定义某个属性
   - `Object.defineProperties()`：通过描述对象，定义多个属性
2. 控制对象状态的方法
   - `Object.preventExtensions()`：防止对象扩展
   - `Object.isExtensible()`：判断对象是否可扩展
   - `Object.seal()`：禁止对象配置
   - `Object.isSealed()`：判断一个对象是否可配置
   - `Object.freeze()`：冻结一个对象
   - `Object.isFrozen()`：判断一个对象是否被冻结
3. 原型链相关方法
   - `Object.create()`：该方法可以指定原型对象和属性，返回一个新的对象
   - `Object.getPrototypeOf()`：获取对象的`Prototype`对象

#### Object的实例方法

定义在`Object.prototype`对象的方法称为实例方法，所有`Object`的实例对象都继承了这些方法；实例方法主要有以下六个：

- `Object.prototype.valueOf()`：返回当前对象对应的值，默认情况下返回对象本身
- `Object.prototype.toString()`：返回当前对象对应的字符串形式

```js
var obj = { a: 1 };
obj.toString() // "[object Object]"

// Object 对象覆盖了 Object.prototype.toString 方法
Object.toString() // "function Object() { [native code] }"
```

PS：**第二个 `Object` 表示该值的构造函数**，利用该类型字符串可以有效的判断数据类型。

由于实例对象可能会自定义`toString`方法（数组、字符串、函数、Date 对象都分别部署了自定义的`toString`方法），覆盖掉`Object.prototype.toString`方法，所以为了得到类型字符串，最好直接使用`Object.prototype.toString`方法。通过函数的`call`方法，可以在任意值上调用这个方法，帮助我们判断这个值的类型。

```js
Object.prototype.toString.call(2)             // "[object Number]"
Object.prototype.toString.call('')            // "[object String]"
Object.prototype.toString.call(true)          // "[object Boolean]"
Object.prototype.toString.call(undefined)     // "[object Undefined]"
Object.prototype.toString.call(null)          // "[object Null]"
Object.prototype.toString.call([])            // "[object Array]"
Object.prototype.toString.call(function() {}) // "[object Function]"
Object.prototype.toString.call(new Date)      // "[object Date]"
Object.prototype.toString.call(/test/ig)      // "[object RegExp]"
Object.prototype.toString.call(Math)          // "[object Math]"
Object.prototype.toString.call({})            // "[object Object]"

typeof Object   // "function"
typeof Object() // "object"
typeof Math     // "object"
```

- `Object.prototype.toLocaleString()`：返回当前对象对应的本地字符串形式

该方法和`toString`的返回结果相同，也是返回一个值的字符串形式；该方法的主要作用是留出一个接口，让各种不同的对象实现自己版本的`toLocaleString`，用来返回针对某些地域的特定的值。

- `Object.prototype.hasOwnProperty()`：判断某个属性是否为当前对象自身的属性，还是继承自原型对象的属性

该方法接受一个字符串作为参数，返回一个布尔值，表示**该实例对象自身是否具有该属性**。

- `Object.prototype.isPrototypeOf()`：判断当前对象是否为另一个对象的原型
- `Object.prototype.propertyIsEnumerable()`：判断某个属性是否可以枚举

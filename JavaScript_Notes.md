// Functions
function name(parameter1, parameter2, ... parameterN) {
 //body
}
// Local variables
A variable declared inside a function is only visible inside that function
For example:
function showMessage() {
let message = "Hello,World"; // local variable
alert( message );
}
showMessage(); // Hello, World
alert( message ); // Error! The variable is local to the function

// Variable shadowing
let userName = "John";

function showMessage() {
let userName = "Bob"; // declare a local variable
let message = 'Hello, ' + userName; // Bob
alert( message )
}

showMessage();
alert(userName); // John


// Function Expression
let sayHi = function() {
alert("Hello");
}
Please note, there’s no name after the function keyword. Omitting a name is allowed for Function Expressions.
Here we immediately assign it to the variable, so the meaning of these code samples is the same: “create a function and put it into the variable sayHi”.

We can copy a function to another variable:
function sayHi() {   // (1) create
  alert( "Hello" );
}

let func = sayHi;    // (2) copy

func(); // Hello     // (3) run the copy (it works)!
sayHi(); // Hello    //     this still works too (why wouldn't it)

Line (2) copies it into the variable func. Please note again: there are no parentheses after sayHi. If there were, then func = sayHi() would write the result of the call sayHi() into func, not the function sayHi itself.

Callback functions
function ask(question, yes, no) {
if
}
Regular values like strings or numbers represent the data.

A function can be perceived as an action.

Function Expression vs Function Declaration
// Function Declaration
function sum(a, b) {
  return a + b;
}

// Function Expression
let sum = function(a, b) {
  return a + b;
};

A Function Expression is created when the execution reaches it and is usable only from that moment.
A Function Declaration can be called earlier than it is defined.
sayHi("John"); // Hello, John

function sayHi(name) {
  alert( `Hello, ${name}` );
}

sayHi("John"); // error!

let sayHi = function(name) {  // (*) no magic any more
  alert( `Hello, ${name}` );
};

A Function Declaration is hoisted and can be called before it's defined, while a Function Expression can only be used after it's assigned to a variable.

Arrow Functions
let func = (arg1, arg2, ..., argN) => expression;

In other words, it’s the shorter version of:
let func = function(arg1, arg2, ..., argN) {
  return expression;
};

let sum = (a, b) => a + b;

/* This arrow function is a shorter form of:

let sum = function(a, b) {
  return a + b;
};
*/

alert( sum(1, 2) ); // 3

Regular functions have their own this, which depends on how they are called. Arrow functions don't have their own this; they inherit it from the surrounding scope.
For instance, we can use it to iterate inside an object method:
let group = {
  title: "Our Group",
  students: ["John", "Pete", "Alice"],

  showList() {
    this.students.forEach(
      student => alert(this.title + ': ' + student)
    );
  }
};

group.showList();

If we used a “regular” function, there would be an error:
let group = {
  title: "Our Group",
  students: ["John", "Pete", "Alice"],

  showList() {
    this.students.forEach(function(student) {
      // Error: Cannot read property 'title' of undefined
      alert(this.title + ': ' + student);
    });
  }
};

group.showList();

BDD(Beahavior Driven Development)

describe("pow", function() {
it("raises to n-th power", function() {
assert.equal(pow(2,3), 8)
})
})

An empty object (“empty cabinet”) can be created using one of two syntaxes:
let user = new Object(); // "object constructor" syntax
let user = {};  // "object literal" syntax


浅拷贝
let user = {
name: "John",
age: 30
}

let clone = {}; // the new empty object

// let's copy all properties into it
for (let key in user) {
clone[key] = user[key];
}

// now clone is fully independent object with the same content
clone.name = "Pete"; // changed the data in it

alert( user.name );// still John in the original object

# JavaScript 对象的浅拷贝与深拷贝

> 核心结论：如果对象里只有原始类型属性（字符串、数字等），浅拷贝和深拷贝的结果完全一样。区别只在“对象里面还有对象”时才会出现。

---

## 1. 变量里存的是什么

把变量想成一个格子：

- **原始类型**（string、number、boolean 等）：格子里直接放**值**。
- **对象**：对象本身放在内存别处，格子里只放它的**地址**（引用）。

```js
let user = { name: "John", age: 30 };
```

```
变量 user：  [ #100 ]

地址 #100 的对象：
   name: "John"
   age:  30
```

---

## 2. 浅拷贝做了什么

```js
let clone = {};   // 新建一个空对象，放在新地址 #200

for (let key in user) {
  clone[key] = user[key];   // 把每个格子的内容原样抄过去
}
```

```
变量 user：  [ #100 ]       变量 clone： [ #200 ]

#100：                      #200：
   name: "John"                name: "John"
   age:  30                    age:  30
```

现在有**两个不同的对象**，各有自己的 `name` 格子。

---

## 3. 修改 clone.name

```js
clone.name = "Pete";
alert(user.name);   // "John"
```

```
#100：                      #200：
   name: "John"   ← 没动       name: "Pete"   ← 改了这里
   age:  30                    age:  30
```

#100 根本没被碰到，所以 `user.name` 不变。

**到这里，浅拷贝和深拷贝没有任何区别。**

---

## 4. 区别出现在“对象里有对象”时

```js
let user = {
  name: "John",
  sizes: { height: 182 }
};
```

`sizes` 也是对象，所以 `user` 里的 `sizes` 格子只存它的地址：

```
#100（user）：              #300（sizes 对象）：
   name:  "John"               height: 182
   sizes: #300  ───────────►
```

做同样的浅拷贝，原样抄格子内容：

```
#100（user）：              #200（clone）：
   name:  "John"               name:  "John"
   sizes: #300                 sizes: #300     ← 抄过来的是地址！
```

`sizes` 对象本身**没有被复制**，两边指向同一个：

```
user.sizes  ──┐
              ├──► #300：height: 182
clone.sizes ──┘
```

```js
clone.name = "Pete";        // 改 #200 的 name → user 不受影响
clone.sizes.height = 190;   // 改 #300 的 height → user 也受影响！

alert(user.name);                    // "John"
alert(user.sizes.height);            // 190
alert(user.sizes === clone.sizes);   // true
```

**“浅”的意思：只复制了最外面一层的格子；格子里如果是地址，只抄了地址，没有复制地址指向的对象。**

---

## 5. 深拷贝会怎样

深拷贝遇到属性是对象时，会**再新建一个对象**，把里面的内容也复制过去：

```
#100（user）：     #300：height: 182
   sizes: #300

#200（clone）：    #400：height: 182   ← 新复制的
   sizes: #400
```

此时 `clone.sizes.height = 190` 只改 #400，`user` 完全不受影响。

---

## 6. 三种情况对比

| 方式 | 外层对象 | 原始类型属性（name） | 对象属性（sizes） |
|---|---|---|---|
| 直接赋值 `clone = user` | 同一个 | 共享 | 共享 |
| 浅拷贝 | 新的 | 独立 | **共享** |
| 深拷贝 | 新的 | 独立 | 独立 |

> `clone.name = "Pete"` 不影响 `user`，**不能证明是深拷贝**，因为浅拷贝也能做到。判断深浅要看嵌套对象是否共享。

---

## 7. 常用方法

### 浅拷贝

```js
let clone1 = Object.assign({}, user);
let clone2 = { ...user };

// 手动循环
let clone3 = {};
for (let key in user) clone3[key] = user[key];
```

### 深拷贝

```js
// 1. 原生方法（推荐）：支持循环引用、Date、Map、Set；不能复制函数和 DOM 节点
let deep1 = structuredClone(user);

// 2. JSON 方法：有缺陷
let deep2 = JSON.parse(JSON.stringify(user));

// 3. lodash
let deep3 = _.cloneDeep(user);
```

**JSON 方法的缺陷（面试常问）：**

- 函数、`undefined`、`Symbol` 会丢失
- `Date` 变成字符串
- `Map`、`Set`、`RegExp` 处理不正确
- 遇到循环引用直接报错
- JSON.parse(JSON.stringify(obj)) 的缺陷都来自同一个原因：它要先把对象转成 JSON 字符串，而 JSON 只支持字符串、数字、布尔、null、普通对象和数组，超出这个范围的东西都会丢失或变形。
- const obj = {
  fn() {},              // 函数
  u: undefined,         // undefined
  s: Symbol('x'),       // Symbol
  d: new Date(),        // Date
  r: /abc/g,            // 正则
  m: new Map([[1, 2]]), // Map
  st: new Set([1, 2]),  // Set
  n: NaN,
  i: Infinity,
  arr: [undefined, () => {}],
};

console.log(JSON.parse(JSON.stringify(obj)));
// {
//   d: "2026-10-28T...Z",   ← Date 变成了字符串
//   r: {},                  ← 正则变成空对象
//   m: {}, st: {},          ← Map、Set 变成空对象
//   n: null, i: null,       ← NaN、Infinity 变成 null
//   arr: [null, null]       ← 数组里的 undefined/函数变成 null
// }
// fn、u、s 三个属性直接消失了
具体来说：

函数、undefined、Symbol 会丢失：作为对象属性时整个属性消失，在数组里则变成 null。
Date 变成字符串：解析回来后不再是 Date 对象，不能调用 getTime() 等方法。
正则、Map、Set 变成 {}，数据全丢。
NaN、Infinity、-Infinity 变成 null。
BigInt 直接报错：TypeError: Do not know how to serialize a BigInt。
循环引用直接报错：
js

### 手写深拷贝（处理循环引用）

```js
function deepClone(obj, map = new WeakMap()) {
  if (obj === null || typeof obj !== "object") return obj;
  if (map.has(obj)) return map.get(obj);   // 处理循环引用

  let result = Array.isArray(obj) ? [] : {};
  map.set(obj, result);

  for (let key in obj) {
    if (obj.hasOwnProperty(key)) {
      result[key] = deepClone(obj[key], map);
    }
  }
  return result;
}
```
function deepClone(obj, map = new WeakMap()) {
  // 基本类型和函数直接返回
  if (obj === null || typeof obj !== 'object') return obj;

  // 特殊对象
  if (obj instanceof Date) return new Date(obj);
  if (obj instanceof RegExp) return new RegExp(obj.source, obj.flags);

  // 循环引用
  if (map.has(obj)) return map.get(obj);

  // Map / Set
  if (obj instanceof Map) {
    const result = new Map();
    map.set(obj, result);
    obj.forEach((v, k) => result.set(deepClone(k, map), deepClone(v, map)));
    return result;
  }
  if (obj instanceof Set) {
    const result = new Set();
    map.set(obj, result);
    obj.forEach(v => result.add(deepClone(v, map)));
    return result;
  }

  // 数组或普通对象，保留原型
  const result = Array.isArray(obj) ? [] : Object.create(Object.getPrototypeOf(obj));
  map.set(obj, result);

  // Reflect.ownKeys 能拿到 Symbol 键
  for (const key of Reflect.ownKeys(obj)) {
    result[key] = deepClone(obj[key], map);
  }
  return result;
}

> 用 `WeakMap` 而不是 `Map`：WeakMap 的键是弱引用，不会阻止垃圾回收。
> Shallow Copy（浅拷贝）

面试可以这样回答：

A shallow copy creates a new object, but nested objects still share the same references.

function deepClone(obj) {

if(obj === null || typeof obj !== "object") {
return obj;
}

const result = Array.isArray(obj) ? [] : {};

  return result;
}
为什么单独判断 null？

因为 JavaScript 存在一个历史遗留问题：
typeof null // "object"
但 null 并不是我们要递归复制的对象。
为什么函数也直接返回？
typeof function() {} // "function"
typeof obj !== "object"
对于函数也成立。

这段实现不会复制函数，而是直接返回原函数引用。


## Airbnb JavaScript代码规范（完整）

### 类型Types
#### 基本数据类型

- string  
- number
- boolean
- null
- undefined
- symbol
  
```
const foo = 1;
let bar = foo;

bar = 9;

console.log(foo, bar); // => 1, 9
```

- Symbols不能真正的被polyfilled，因此当目标浏览器或者环境本地不支持时，不应当使用Symbols.

#### 复杂数据类型

- object
- array
- function

```
const foo = [1, 2];
const bar = foo;

bar[0] = 9;

console.log(foo[0], bar[0]); // => 9, 9
```

#### 引用References
- 尽量使用const，避免使用var。
```
// bad
var a = 1;
var b = 2;

// good
const a = 1;
const b = 2;
```

- 如果你必须对变量重新赋值，使用let.
  
> why? let属于块作用域，var是函数作用域

```
// bad
var count = 1;
if (true) {
  count += 1;
}

// good, use the let.
let count = 1;
if (true) {
  count += 1;
}
```

- 记住let和const都属于块作用域

```
// const and let only exist in the blocks they are defined in.
{
  let a = 1;
  const b = 1;
}
console.log(a); // ReferenceError
console.log(b); // ReferenceError
```

#### 对象Objects

- 使用字面量方式创建对象。

```
/ bad
const item = new Object();

// good
const item = {};
```
- 在创建对象时，定义对象的所有属性
> Why?这样对象所有的属性都在同一处定义

```
function getKey(k) {
  return `a key named ${k}`;
}

// bad
const obj = {
  id: 5,
  name: 'San Francisco',
};
obj[getKey('enabled')] = true;

// good
const obj = {
  id: 5,
  name: 'San Francisco',
  [getKey('enabled')]: true,
};
```

- 使用简写方式定义对象方法。

```
// bad
const atom = {
  value: 1,

  addValue: function (value) {
    return atom.value + value;
  },
};

// good
const atom = {
  value: 1,

  addValue(value) {
    return atom.value + value;
  },
};
```

- 使用简写方式定义对象属性。
> why?书写更简洁

```
const lukeSkywalker = 'Luke Skywalker';

// bad
const obj = {
  lukeSkywalker: lukeSkywalker,
};

// good
const obj = {
  lukeSkywalker,
};
```

- 把简写的属性放在对象定义的开头
> why?这样更容易判断哪些属性使用了简写

```
const anakinSkywalker = 'Anakin Skywalker';
const lukeSkywalker = 'Luke Skywalker';

// bad
const obj = {
  episodeOne: 1,
  twoJediWalkIntoACantina: 2,
  lukeSkywalker,
  episodeThree: 3,
  mayTheFourth: 4,
  anakinSkywalker,
};

// good
const obj = {
  lukeSkywalker,
  anakinSkywalker,
  episodeOne: 1,
  twoJediWalkIntoACantina: 2,
  episodeThree: 3,
  mayTheFourth: 4,
};
```

- 只对不合法的标志符使用引号
> why?通常这样代码更可读，同时语法高亮，且js引擎更容易优化。

```
// bad
const bad = {
  'foo': 3,
  'bar': 4,
  'data-blah': 5,
};

// good
const good = {
  foo: 3,
  bar: 4,
  'data-blah': 5,
};
```

- 不要直接调用Object.prototype，同理： hasOwnProperty, propertyIsEnumerable, isPrototypeOf
> why?这些方法可能被吞没，比如用Object.create(null)方式创建的对象

```
// bad
console.log(object.hasOwnProperty(key));

// good
console.log(Object.prototype.hasOwnProperty.call(object, key));

// best
const has = Object.prototype.hasOwnProperty; // cache the lookup once, in module scope.
/* or */
import has from 'has';
// ...
console.log(has.call(object, key));
```

- 使用对象的spread操作符而不是Object.assign方法来浅拷贝对象。使用rest操作符获得一个去除某些属性新的对象。

```
// very bad
const original = { a: 1, b: 2 };
const copy = Object.assign(original, { c: 3 }); // this mutates `original` ಠ_ಠ
delete copy.a; // so does this

// bad
const original = { a: 1, b: 2 };
const copy = Object.assign({}, original, { c: 3 }); // copy => { a: 1, b: 2, c: 3 }

// good
const original = { a: 1, b: 2 };
const copy = { ...original, c: 3 }; // copy => { a: 1, b: 2, c: 3 }

const { a, ...noA } = copy; // noA => { b: 2, c: 3 }
```

#### 数组Arrays

- 使用字面量定义数组 

```
// bad
const items = new Array();

// good
const items = [];
```
- 使用Array#push方法添加元素

```
const someStack = [];

// bad
someStack[someStack.length] = 'abracadabra';

// good
someStack.push('abracadabra');
```
- 使用...操作拷贝数组

```
// bad
const len = items.length;
const itemsCopy = [];
let i;

for (i = 0; i < len; i += 1) {
  itemsCopy[i] = items[i];
}

// good
const itemsCopy = [...items];
```

- 转换array-like对象为array时，使用...而不是Array.from

```
const foo = document.querySelectorAll('.foo');

// good
const nodes = Array.from(foo);

// best
const nodes = [...foo];
```
- 对数组进行map时，使用Array.from替代...。因为前者的方式能避免创建中间数组

```
// bad
const baz = [...foo].map(bar);

// good
const baz = Array.from(foo, bar);
```
- 在数组遍历处理的回调函数中，使用返回语句。当回调函数中函数体只有单条语句且不会产生副作用时，可以省略return。

```
// good
[1, 2, 3].map((x) => {
  const y = x + 1;
  return x * y;
});

// good
[1, 2, 3].map(x => x + 1);

// bad - no returned value means `memo` becomes undefined after the first iteration
const flat = {};
[[0, 1], [2, 3], [4, 5]].reduce((memo, item, index) => {
  const flatten = memo.concat(item);
  memo[index] = flatten;
});

// good
const flat = {};
[[0, 1], [2, 3], [4, 5]].reduce((memo, item, index) => {
  const flatten = memo.concat(item);
  memo[index] = flatten;
  return flatten;
});

// bad
inbox.filter((msg) => {
  const { subject, author } = msg;
  if (subject === 'Mockingbird') {
    return author === 'Harper Lee';
  } else {
    return false;
  }
});

// good
inbox.filter((msg) => {
  const { subject, author } = msg;
  if (subject === 'Mockingbird') {
    return author === 'Harper Lee';
  }

  return false;
});
```
- 当数组有多行时，在开始和结束符号均换行

```
// bad
const arr = [
  [0, 1], [2, 3], [4, 5],
];

const objectInArray = [{
  id: 1,
}, {
  id: 2,
}];

const numberInArray = [
  1, 2,
];

// good
const arr = [[0, 1], [2, 3], [4, 5]];

const objectInArray = [
  {
    id: 1,
  },
  {
    id: 2,
  },
];

const numberInArray = [
  1,
  2,
];
```

#### 解构Destructuring

- 当访问对象的多个属性时，使用解构方式 

> why?解构可以减少临时变量的定义

```
// bad
function getFullName(user) {
  const firstName = user.firstName;
  const lastName = user.lastName;

  return `${firstName} ${lastName}`;
}

// good
function getFullName(user) {
  const { firstName, lastName } = user;
  return `${firstName} ${lastName}`;
}

// best
function getFullName({ firstName, lastName }) {
  return `${firstName} ${lastName}`;
}
```
- 使用数组解构

```
const arr = [1, 2, 3, 4];

// bad
const first = arr[0];
const second = arr[1];

// good
const [first, second] = arr;
```
- 当返回多个值时，使用对象解构方式，而不是数组解构

> why?当添加字段或者顺序发生变化时，不依赖于位置

```
// bad
function processInput(input) {
  // then a miracle occurs
  return [left, right, top, bottom];
}

// the caller needs to think about the order of return data
const [left, __, top] = processInput(input);

// good
function processInput(input) {
  // then a miracle occurs
  return { left, right, top, bottom };
}

// the caller selects only the data they need
const { left, top } = processInput(input);
```

#### 字符串Strings
- 字符串使用单引号' '

```
// bad
const name = "Capt. Janeway";

// bad - template literals should contain interpolation or newlines
const name = `Capt. Janeway`;

// good
const name = 'Capt. Janeway';
```

- 字符串超过100个字符时，不应该跨行

> 断开的字符串不易搜索，且不方便

```
// bad
const errorMessage = 'This is a super long error that was thrown because \
of Batman. When you stop to think about how Batman had anything to do \
with this, you would get nowhere \
fast.';

// bad
const errorMessage = 'This is a super long error that was thrown because ' +
  'of Batman. When you stop to think about how Batman had anything to do ' +
  'with this, you would get nowhere fast.';

// good
const errorMessage = 'This is a super long error that was thrown because of Batman. When you stop to think about how Batman had anything to do with this, you would get nowhere fast.';
```

- 拼接字符串时，使用模板字符串

> why?模板字符串可读性更强，语法更简洁

```
// bad
function sayHi(name) {
  return 'How are you, ' + name + '?';
}

// bad
function sayHi(name) {
  return ['How are you, ', name, '?'].join();
}

// bad
function sayHi(name) {
  return `How are you, ${ name }?`;
}

// good
function sayHi(name) {
  return `How are you, ${name}?`;
}
```
- 永远不要使用eval()字符串
- 不要多余的转义

> why？影响可读性

```
// bad
const foo = '\'this\' \i\s \"quoted\"';

// good
const foo = '\'this\' is "quoted"';
const foo = `my name is '${name}'`;
```

#### 函数Functions
- ?使用命名函数表达式，而不是函数声明

> 函数声明的方式存在提升，即：无论在哪里声明，效果等同于在函数顶部声明，只要在同一个作用域范围，就视为已经声明，哪怕在声明前就使用，也不会报错。
如果使用函数声明方式定义函数，会影响可读性和可维护性。当函数足够大或者复杂时，对阅读其余代码造成困扰。
```
// bad
function foo() {
  // ...
}

// bad
const foo = function () {
  // ...
};

// good
// lexical name distinguished from the variable-referenced invocation(s)
const short = function longUniqueMoreDescriptiveLexicalFoo() {
  // ...
};
```

- 要求 IIFE 使用括号括起来

```
// immediately-invoked function expression (IIFE)
(function () {
  console.log('Welcome to the Internet. Please follow me.');
}());
```
- ？Never declare a function in a non-function block (if, while, etc). Assign the function to a variable instead. Browsers will allow you to do it, but they all interpret it differently, which is bad news bears.

- ？ Note: ECMA-262 defines a block
as a list of statements. A function declaration is not a statement. 

```
// bad
if (currentUser) {
  function test() {
    console.log('Nope.');
  }
}

// good
let test;
if (currentUser) {
  test = () => {
    console.log('Yup.');
  };
}
```

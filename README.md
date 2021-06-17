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


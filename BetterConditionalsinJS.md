> * 原文地址：[5 Tips to Write Better Conditionals in JavaScript](https://scotch.io/tutorials/5-tips-to-write-better-conditionals-in-javascript)
> * 译文来自：[Github:EmilyQiRabbit](https://github.com/EmilyQiRabbit/CodingRepository)
> * 译者：[Yuqi](https://github.com/EmilyQiRabbit)
> * **欢迎校对** 

# 五个妙招教你如何更优雅的书写 JS 条件判断

## 1. 用 Array.includes 辅助多条件判断

常见代码，例一：

```js
// condition
function test(fruit) {
  if (fruit == 'apple' || fruit == 'strawberry') {
    console.log('red');
  }
}
```

似乎看上去还不错，但是如果条件很多呢？比如还需要判断是不是 cherry 和 cranberries？

改进的方法是使用 Array.includes：

```js
function test(fruit) {
  // extract conditions to array
  const redFruits = ['apple', 'strawberry', 'cherry', 'cranberries'];

  if (redFruits.includes(fruit)) {
    console.log('red');
  }
}
```

代码看上去整洁很多。

## 2. 少嵌套，return 要尽早写

为例一添加两个需求：

* 如果没有提供水果，抛出错误

* 打印超过 10 个的水果

常见代码：

```js
function test(fruit, quantity) {
  const redFruits = ['apple', 'strawberry', 'cherry', 'cranberries'];

  // condition 1: fruit must has value
  if (fruit) {
    // condition 2: must be red
    if (redFruits.includes(fruit)) {
      console.log('red');

      // condition 3: must be big quantity
      if (quantity > 10) {
        console.log('big quantity');
      }
    }
  } else {
    throw new Error('No fruit!');
  }
}

// test results
test(null); // error: No fruits
test('apple'); // print: red
test('apple', 20); // print: red, big quantity
```

上面这段代码用了一个 if/else 语句来找出了不符合条件的，同时使用了三层 if 嵌套。

个人比较推崇的方法是：**如果发现了异常，应该尽早抛出。**

```js
/_ return early when invalid conditions found _/

function test(fruit, quantity) {
  const redFruits = ['apple', 'strawberry', 'cherry', 'cranberries'];

  // condition 1: throw error early
  if (!fruit) throw new Error('No fruit!');

  // condition 2: must be red
  if (redFruits.includes(fruit)) {
    console.log('red');

    // condition 3: must be big quantity
    if (quantity > 10) {
      console.log('big quantity');
    }
  }
}
```

这样的好处之一是减少了一层 if 嵌套；其次，如果 if 语句很长，那么就需要翻到很下面去看 else 写了什么，可读性就相对差。

进一步，还可以继续减少 if 嵌套的层数：

```js
/_ return early when invalid conditions found _/

function test(fruit, quantity) {
  const redFruits = ['apple', 'strawberry', 'cherry', 'cranberries'];

  if (!fruit) throw new Error('No fruit!'); // condition 1: throw error early
  if (!redFruits.includes(fruit)) return; // condition 2: stop when fruit is not red

  console.log('red');

  // condition 3: must be big quantity
  if (quantity > 10) {
    console.log('big quantity');
  }
}
```

这样，代码中已经没有 if 的嵌套了。但是这并不是硬性规定，也不用做的太过头了。

## 3. 巧用函数的参数默认值和解构

这段代码你一定很熟悉：

```js
function test(fruit, quantity) {
  if (!fruit) return;
  const q = quantity || 1; // if quantity not provided, default to one

  console.log(`We have ${q} ${fruit}!`);
}

//test results
test('banana'); // We have 1 banana!
test('apple', 2); // We have 2 apple!
```

事实上，我们可以通过设置函数参数的默认值来省略掉 q 变量：

```js
function test(fruit, quantity = 1) { // if quantity not provided, default to one
  if (!fruit) return;
  console.log(`We have ${quantity} ${fruit}!`);
}

//test results
test('banana'); // We have 1 banana!
test('apple', 2); // We have 2 apple!
```

更进一步的，如果 fruit 是一个对象呢？我们要怎么做？

常见的写法是这样的：

```js
function test(fruit) { 
  // printing fruit name if value provided
  if (fruit && fruit.name)  {
    console.log (fruit.name);
  } else {
    console.log('unknown');
  }
}

//test results
test(undefined); // unknown
test({ }); // unknown
test({ name: 'apple', color: 'red' }); // apple
```

这时候，借助解构的方法，我们可以省略掉 fruit && fruit.name 的判断：

```js
// destructing - get name property only
// assign default empty object {}
function test({name} = {}) {
  console.log (name || 'unknown');
}

//test results
test(undefined); // unknown
test({ }); // unknown
test({ name: 'apple', color: 'red' }); // apple
```

而如果可以借助外部库（比如 lodash）：

```js
// Include lodash library, you will get _
function test(fruit) {
  console.log(__.get(fruit, 'name', 'unknown'); // get property name, if not available, assign default value 'unknown'
}

//test results
test(undefined); // unknown
test({ }); // unknown
test({ name: 'apple', color: 'red' }); // apple
```

## 4. 使用 Map/Object 来代替 Switch 语句

使用 Switch 语句的写法：

```js
function test(color) {
  // use switch case to find fruits in color
  switch (color) {
    case 'red':
      return ['apple', 'strawberry'];
    case 'yellow':
      return ['banana', 'pineapple'];
    case 'purple':
      return ['grape', 'plum'];
    default:
      return [];
  }
}

//test results
test(null); // []
test('yellow'); // ['banana', 'pineapple']
```

代码显得很冗长，更简洁的写法是：

```js
// use object literal to find fruits in color
  const fruitColor = {
    red: ['apple', 'strawberry'],
    yellow: ['banana', 'pineapple'],
    purple: ['grape', 'plum']
  };

function test(color) {
  return fruitColor[color] || [];
}
```

或者使用 Map：

```js
// use Map to find fruits in color
  const fruitColor = new Map()
    .set('red', ['apple', 'strawberry'])
    .set('yellow', ['banana', 'pineapple'])
    .set('purple', ['grape', 'plum']);

function test(color) {
  return fruitColor.get(color) || [];
}
```

## 5. 使用 Array.every 和 Array.some 来做判断

例如，我们需要判断是否所有的水果都是红色：

```js
const fruits = [
    { name: 'apple', color: 'red' },
    { name: 'banana', color: 'yellow' },
    { name: 'grape', color: 'purple' }
  ];

function test() {
  let isAllRed = true;

  // condition: all fruits must be red
  for (let f of fruits) {
    if (!isAllRed) break;
    isAllRed = (f.color == 'red');
  }

  console.log(isAllRed); // false
}
```

更好的写法是：

```js
const fruits = [
    { name: 'apple', color: 'red' },
    { name: 'banana', color: 'yellow' },
    { name: 'grape', color: 'purple' }
  ];

function test() {
  // condition: short way, all fruits must be red
  const isAllRed = fruits.every(f => f.color == 'red');

  console.log(isAllRed); // false
}
```

同理，如果你想检测数组中是否有水果是红色的，则可以用 Array.some：

```js
const fruits = [
    { name: 'apple', color: 'red' },
    { name: 'banana', color: 'yellow' },
    { name: 'grape', color: 'purple' }
];

function test() {
  // condition: if any fruit is red
  const isAnyRed = fruits.some(f => f.color == 'red');

  console.log(isAnyRed); // true
}
```

Happy coding～

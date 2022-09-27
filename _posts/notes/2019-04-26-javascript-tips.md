---
layout: post
title: JavaScript Tips
date: 2019-04-26 20:44 +0800
tags:
  - Development
---

Tips for JavaScript.

{% include toc.html %}

### What's weird

- Number comparison

  ```js
  console.log(1 < x < 3); // true (this syntax is valid)
  // (1 < x) < 3
  // (true or false) < 3
  // (1 or 0) < 3
  // true

  console.log(1 < 5 && 5 < 3); // false
  // true && false
  // false
  ```

- Some unexpected behaviors

  ```js
  console.log(1 + 2 + '3'); // 33
  console.log('1' + 2 + 3); // 123

  console.log(0.1 + 0.2); // 0.30000000000000004
  console.log(0.1 + 0.2 === 0.3); // false
  // you should use decimal.js: https://github.com/MikeMcl/decimal.js/

  console.log(true == 1); // true
  console.log(true == '1'); // true
  console.log(true === 1); // false

  console.log(Boolean([])); // true
  console.log([] == true); // false
  console.log([] == false); // true
  if ([]) { /* this will be executed */ }
  if ([] == true) { /* but this won't */ }

  console.log(Boolean({})); // true
  console.log({} == true); // false
  console.log({} == false); // false
  if ({}) { /* this will be executed */ }
  ```

- Not so asynchronous

  ```js
  setTimeout(() => {
    console.log('hi');
  }, 0);

  // It will say hi after this line finishes,
  // and this line takes about 1 second.
  for (let i = 0; i < 1e9; i++) {}
  ```

### `var`, `let` and `const`

- `var` and `let`

  ```js
  { var a = 1; }
  console.log(a); // 1

  { let b = 2; }
  console.log(b); // error

  for (var i = 0; i < 10; i++);
  console.log(i); // 10

  for (let j = 0; i < 10; j++);
  console.log(j); // error

  var x = 1;
  console.log(window.x); // 1

  let y = 2;
  console.log(window.y); // undefined
  ```

- `const`

  ```js
  const c = 10;
  c = 20; // error

  const o = {c: 10};
  o.c = 20; // ok
  o.d = 30; // ok
  o = 40;   // error

  { const k = 10; }
  console.log(k); // error
  ```

### What the fuck is `this`

- If `this` is used in a function, `this` will be the object where the function is called from, else the global object.
- Global object for browsers is `window`, for Node.js is `global`.

```javascript
window.value = 'global';
let value = 'local';
let obj = {value: 'object'};


function printThisValue() {
  console.log(this.value);
}

function printValue() {
  console.log(value);
}

obj.printThisValue = printThisValue;


printValue();         // local
printThisValue();     // global
obj.printThisValue(); // object

[1].forEach(obj.printThisValue);        // global
[1].forEach(_ => obj.printThisValue()); // object
```

### Promise

- `resolve()` calls the function inside `.then()`:
  
  ```js
  function sleep(seconds) {
    return new Promise(resolve => {
      setTimeout(() => {
        resolve();
      }, seconds * 1000)
    });
  }

  sleep(1).then(() => {
    console.log('hello');
  });

  // chaining then()
  sleep(1)
    .then(() => 123)
    .then(console.log); // 123
  ```

- `reject()` is used to handle exceptions:

  ```js
  function divide(a, b) {
    return new Promise((resolve, reject) => {
      if (b == 0) {
        reject('divided by zero');
      }
      resolve(a / b);
    });
  }

  divide(1, 2)
    .then(console.log) // 0.5
    .catch(console.log);

  divide(1, 0)
    .then(console.log)
    .catch(console.log); // divided by zero

  divide(1, 0)
    .catch(() => 'oh my god')
    .then(console.log); // oh my god
  ```

### Asynchronous

- JavaScript code runs in one thread, so if theres an infinite loop in it, it blocks forever.
- Sync function calls will not interrupt the execution.
- An async function call will be interrupted whenever it reachs the `await` statement.
- `await` is only available in async functions.
- `Promise` is awaitable.
- Actually what async functions return are just `Promise`.
- In async functions, `return` is equivalent to `resolve()`, and `throw` is equivalent to `reject()`.

```js
function sleep(seconds) {
  return new Promise((resolve, reject) => {
    setTimeout(resolve, seconds * 1000);
  });
}

async function sleepAsync(seconds) {
  await sleep(seconds);
}

async function kitty() {
  return 'kitty';
}

async function main() {
  console.log('hello');
  console.log(await kitty());

  // Sleep 1 second
  await sleep(1);
  console.log('hi');

  // Sleep 1 second
  await sleepAsync(1)
  console.log('hihi');

  // Sleep 1 second
  await new Promise(r => setTimeout(r, 1000));

  if (Math.random() > 0.5)
    return 'hihihi'; // resolve('hihihi')
  else
    throw 'hahaha'; // reject('hahaha')
}

main() // this is a Promise
  .then(console.log)
  .catch(console.log);

console.log('this will appear between hello and kitty because main() awaits kitty()');

```



### Generators

- Basic usage:

  ```js
  function* gen() {
    console.log('start');

    let x = yield 'one';
    yield x;

    return 'two';
    return 'three';
  }

  let g = gen();

  g.next(); // prints 'start'
  //=> {value: 'one', done: false}

  g.next('this is x');
  //=> {value: 'this is x', done: false}

  g.next();
  //=> {value: 'two', done: true}

  ```

- Tips

  ```js
  function* iter(n) {
    for (let i = 0; i < n; i++)
      yield i;
  }
  
  console.log(...iter(4));      // 0 1 2 3

  [...iter(4)];                 //=> [0, 1, 2, 3]
  Array.from(iter(4));          //=> [0, 1, 2, 3]

  [...Array(4).keys()];         //=> [0, 1, 2, 3]
  Array.from(Array(4).keys());  //=> [0, 1, 2, 3]
  ```

### Copying

- Array

  ```js
  let a = [1, 2, 3];
  let b = [...a];     // [1, 2, 3]
  let c = a.slice();  // [1, 2, 3]

  a == a; // true
  a == b; // false
  a == c; // false
  ```

- Object
  ```js
  let o = {name: 'djosix'};
  let p = Object.assign({}, o); // {name: 'djosix'}

  p == o; // false
  ```

### Class

- The old way

  ```js
  function Test(value) {
    this.value = value;

    this.print = function() {
      console.log(this.value);
    };
  }

  new Test('hello').print(); // hello
  Test(); //=> undefined

  // Inheritance
  function Child(...args) { Test.call(this, ...args); }
  new Child('kitty').print(); // kitty
  ```

- The new syntactic sugar

  ```js
  class Test {
    value = 'original value';
    
    constructor(value) {
      this.value = value;
    }

    print() {
      console.log(this.value);
    }
  }

  new Test('hello').print(); // hello
  Test(); // error


  class Child extends Test {}
  new Child('kitty').print(); // kitty
  ```


### Others

- Find Vue component from DevTools console
  ```js
  function findComponent(filter) {
    function traverse(root) {
        if (filter(root)) return root;
        for (const child of (root.$children || [])) {
            const result = traverse(child);
            if (result) return result;
        }
    }
    const root = document.querySelector('#app').__vue__;
    return traverse(root);
  }

  findComponent(c => c.someUniqueMethod).someUniqueMethod();
  findComponent(c => c.$options.name === 'SomeTable').selected = 0;
  ```
- Force opening popup windows in the same tab
  ```js
  [].forEach.call(document.querySelectorAll('a'), function (link) {
    if (link.attributes.target) link.attributes.target.value = '_self';
  });

  window.open = function (url) { location.href = url; };
  ```

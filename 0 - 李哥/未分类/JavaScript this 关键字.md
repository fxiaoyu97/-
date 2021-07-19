# JavaScript this 关键字

## 概述

任何执行 `JavaScript` 的环境称之为 **执行上下文**，`JavaScript` **运行时** 维护这些执行上下文的堆栈，并且当正在执行存在于该堆栈顶部的执行上下文。`this` 变量引用的对象每次更改执行上下文时都会更改。

默认情况下，执行上下文是全局的，这意味着如果代码作为简单函数调用的一部分执行，则该 `this` 变量将引用 **全局对象** 。在浏览器的情况下，全局对象是 `windows` 对象。但在 `NodeJS` 环境中，`this` 值是一个特殊的 `global` 对象。

## 简单函数

```javascript
  // 案例 1，简单函数，浏览器下的 this -> window，NodeJS 下的 this -> global
  function simple1() {
    console.log("案例 1，简单函数");
    console.log("我是简单函数，this 指向 window");
    console.log(this);
  }
  simple1();
```

## 严格模式

```javascript
  // 案例 2，严格模式，this -> undefined
  function simple2() {
    'use strict';
    console.log("案例 2，严格模式");
    console.log("我是严格模式，this 指向 undefined");
    console.log(this);
  }
  simple2();
```

## 构造函数

当我们调用 `new User`，JavaScript 会在 User 函数内创建一个新对象并把它保存为 `this` 。接着，`name`, `age` 和 `info` 属性会被添加到新创建的 `this` 对象上。

```javascript
  // 案例 3，构造函数，此时在执行上下文中创建了 this 对象，并在这个 this 对象中增加了 name, age, info 三个属性
  function User(name, age) {
    this.name = name;
    this.age = age;

    this.info = function () {
      console.log("我是 User 对象中的 info 属性")
      console.log(`${this.name} ${this.age}`);
    };
    console.log("案例 3，构造函数");
    console.log("我是 User 对象，我在执行上下文中创建了 this 对象，并增加了 name, age, info 三个属性");
    console.log(this);
  }
  let andy = new User('Andy', 22);
  andy.info();
```

## 简单对象 1

当某个属性被发起普通函数调用时，则 `this` 指向 `window` 对象

```javascript
  // 案例 4，简单对象
  let user = {
    name: '小明',
    age: userAgeFunction,
    sex: function () {
      console.log("我是 user 对象的 sex 属性，this 指向 user 对象，而不是全局对象");
      console.log(this);
    },
    talk: function () {
      console.log(this);
    }
  };
  function userAgeFunction() {
    console.log("我是 user 对象的 age 属性，this 指向 user 对象，而不是全局对象");
    console.log(this);
  }
  console.log("案例 4，简单对象");
  user.age();
  user.sex();

  let talk = user.talk;
  console.log("我是 talk，我把 user 对象的 talk 属性当普通函数调用了，所以这里的 this 指向 window");
  talk();
```

## 简单对象 2

`age` 函数内部， `this` 指向 `lee` 对象，但是，在 `innerAge` 函数内部，`this` 指向全局对象，规则告诉我们无论何时一个普通函数被调用时，那么 `this` 将指向全局对象。

```javascript
  // 案例 5，简单对象 2
  let lee = {
    name: 'lee',
    birth: 1998,
    age: function () {
      console.log("我是 lee 对象的 age 属性，this 指向 lee 对象，而不是全局对象");
      console.log(this);
      function innerAge() {
        console.log("我虽然是 lee 对象中的 age 属性中的 innerAge() 函数，但我会被普通调用，所以这里的 this 指向 window，而不是 lee 对象");
        console.log(this);
      }
      // 这里发起 lee 对象 age 属性内部的 innerAge() 普通调用，所以该函数中的 this 指向 window
      innerAge();
    }
  };
  console.log("案例 5，简单对象");
  lee.age();
```

## call、apply、bind

- call：将 `this` 指向目标对象
- apply：将 `this` 指向目标对象，在参数形式为数组时使用该方法
- bind：将 `this` 指向传递的第一个参数并返回一个新方法

```javascript
    function User(username, password) {
        this.username = username;
        this.password = password;
        this.displayUser = function() {
            console.log(`User: ${this.username} ${this.password}`);
        }
    }

    let user1 = new User("admin", "123456");
    let user2 = new User("guest", "abcdef");

    console.log("案例 6，call apply bind");
    console.log("我是 user1 对象");
    user1.displayUser();
    console.log("我是 user2 对象")
    user2.displayUser();
    console.log("我是 user1 对象，我使用了 call 方法将我的 this 指向了 user2")
    user1.displayUser.call(user2);
    console.log("我是 user1 对象，我使用了 apply 方法将我的 this 指向了 user2")
    user1.displayUser.apply(user2);
    console.log("我是 user1 对象，我使用了 bind 方法将我的 this 指向了 user2 并返回了一个新的方法")
    let user2Display = user1.displayUser.bind(user2)
    user2Display()
```

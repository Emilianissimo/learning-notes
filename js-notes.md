# Callbacks

Callbacks are just functuons, which sending as parametrs for our higher functions, f.e.:
```js
const doHomework = (subject, callback) => {
  alert(`Starting my ${subject} homework.`);
  callback();
};

doHomework('math', () => {
  alert('Finished my homework');
});
```
It will guarantee that our "Finished homework" alert will appear only after ending previos function. This is need to be in case of if first function will 
be stopped by smth, JS will not wait it and will return other func's value. So, if we need strict order, we gonna use callbacks.

# Arrays

### Reduce
Reducer need if we need to do something with our array values and each iteration save the result of pervious iteration, like:
```js
const array1 = [2, 4, 3, 4];

// 0 + 1 + 2 + 3 + 4
const initialValue = 0;
const sumWithInitial = array1.reduce(
  (accumulator, currentValue) => {
    console.log(accumulator, currentValue)
  	return accumulator + currentValue
  },
  initialValue
);

console.log(sumWithInitial);
```

Here I have array of values and I need to sum the all, so by using reduce method, I'm getting current value as 0 (started value) and each iteration add it to current value and store it inside accumulator variable (in first iteration my second reduce param gets initialValue and storing in inside accumulator).

# Objects 

### Comparing objects
```js
function isEqual(object1, object2) {
  const props1 = Object.getOwnPropertyNames(object1);
  const props2 = Object.getOwnPropertyNames(object2);

  if (props1.length !== props2.length) {
    return false;
  }

  for (let i = 0; i < props1.length; i += 1) {
    const prop = props1[i];
    const bothAreObjects = typeof(object1[prop]) === 'object' && typeof(object2[prop]) === 'object';

    if ((!bothAreObjects && (object1[prop] !== object2[prop]))
    || (bothAreObjects && !isEqual(object1[prop], object2[prop]))) {
      return false;
    }
  }

  return true;
}
const object1 = {
  title: "Title",
  id: 1
};

const object2 = {
  id: 1
  title: "Title",
};

console.log(isEqual(object1, object2)); // true
```
Custom function, where we check each key and value inside and if there is a nested object, recursing same function, or loadash lib instead:
```js
const object1 = {
  title: "Title",
  id: 1
};

const object2 = {
  id: 1
  title: "Title",
};

console.log(_.isEqual(object1, object2)); // true
```
Or we can just stringify it and check equality as a string, but if there is no the same order, it could return false.

### Copying objects
You cannot copy object via variable, just creating a link on it into memory. To fully copying it, you have to use Object.assign, or create a brand new object by your hands:
```js
let a = {};
let b = a;
// Just a link
// Object.assign
let user = {
  name: "John",
  age: 30
};

let clone = Object.assign({}, user);
```
But if we have nested, object, we'll need to make deep copying, that uses loop for checking each key if it is an object. So to not recreate bicycle, just use _.cloneDeep(obj) from loadash.

<b>REMARK: if our object is a const, our properties still can be changed.</b>

# THIS

In the lax (not strict) it is a link to the current object (just as in php), but in strict mode can me different.

### Global context
In global context (outside functions) <b>this</b> is linking to the global object, no matter the mode (strict or not):
```js
console.log(this === window); // true
a = 37;
console.log(window.a); // 37

this.b = "MDN";
console.log(window.b)  // "MDN"
console.log(b)         // "MDN"
```
<b>NOTES: we can always force JS to have global object no matter the current context -> just using <i>globalThis</i></b>

### Function context
In no strict mode function call still be an global object:
```js
function f1(){
  return this;
}

// В браузере:
f1() === window; // window - global browser obj

// В Node:
f1() === global; // global - Node's global obj
```
But in strict mode we need to set value of this in the context of running or it will be undefined:
```js
function f2(){
  "use strict"; // см. strict mode
  return this;
}

f2() === undefined; // true
```
For set behaviour of our program, we gonna set value via apply or call methods:
```js
// As a first argument of call or apply we can provide an object, which will replace global object. After we gonna pass some args
var obj = {a: 'Custom'};

// Global's prop
var a = 'Global';

function whatsThis(somearg) {
  return this.a;  // value of this is according to the context of call

}

whatsThis();          // 'Global'
whatsThis.call(obj, 228);  // 'Custom'
whatsThis.apply(obj, 228); // 'Custom'
```

But in ES5 we had a new method -> bind(), which provides us opportunity to set its first argument as a THIS context of function forever (by making new function with the same props):
```js
function f() {
  return this.a;
}

var g = f.bind({a: 'azerty'}); // now this object will be context of the g(). Instead of f()'s global context. Forever, no need to implement call or apply.
console.log(g()); // azerty

var h = g.bind({a: 'yoo'}); // bind only works once!
console.log(h()); // azerty

var o = {a: 37, f: f, g: g, h: h};
console.log(o.a, o.f(), o.g(), o.h()); // 37,37, azerty, azerty
```

### Arrow funcs context will always have global context for 'this'

### By using 'this' inside object's method, you gonna have current object's scope and context.

# Computed props
-

You're able to create computed props, like if you need dynamic prop name just by adding [] to the prop name:
```js
let propName = 'c';

const rank = {
  a: 1,
  b: 2,
  [propName]: 3,
};

console.log(rank.c); // 3

let name = 'fullName';

class Person {
  constructor(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }
  get [name]() {
    return `${this.firstName} ${this.lastName}`;
  }
}

let person = new Person('John', 'Doe');
console.log(person.fullName);
```

# Promises and fetching
So, Promise has 3 states -> pending (on running) and returning result -> resolve (success) or reject (error).
```js
function httpGet(url) {

  return new Promise(function(resolve, reject) {

    var xhr = new XMLHttpRequest();
    xhr.open('GET', url, true);

    xhr.onload = function() {
      if (this.status == 200) {
        resolve(this.response);
      } else {
        var error = new Error(this.statusText);
        error.code = this.status;
        reject(error);
      }
    };

    xhr.onerror = function() {
      reject(new Error("Network Error"));
    };

    xhr.send();
  });

}
```
Aslo it can looks different, like fetch()
```js
fetch('link').then(
    response => response,
    reject => 'eat some'
).then(response => {
    return fetch(`link/${response.data}`)
})
```

# Event loop, promises and other stuff
Here we can see important example of JS web browser interpritator's queue of call stack:
```js
console.log('pre-pivot');
setTimeout(() => {
	return  console.log(1);
});

setTimeout(() => {
	return console.log(2);
})

const prom = new Promise(resolve => resolve());

prom.then(data => console.log(data));
console.log('pivot')
setTimeout(() => console.log(3));
```
Result will return this:
```bash
$ -> node file.js
pre-pivot
pivot
undefined
1
2
3
```

That's happening in case of JS returns everything by it's readyness. That means, our global console calls are ready first, even if they are in the different order and places (just as always in async shit). After we resolving our Promise and only after that we have setTimeout result, that means that timeout couldn't be less than 1 ms and it cannot be returned instantly. It just resolving undefined state faster than that, in other cases it will return something else first if, maybe, our promise resolving will be slower. Simple parallelisation via async shit.

If we need to control promise resolving by timeout, we'll need to use callbacks and binding by .then().

### Closer about event loop
Js is syncronous as itself, but inside browser it's fully async by using WEB API.

Here some visualisation: https://bool.dev/blog/detail/obyasnenie-event-loop-v-javascript-s-pomoshchyu-vizualizatsii

But if link does not works, here we have four things for our functions running:
- Callstack
- Eventloop
- Web API
- Queue

So, here some funcs:
```js
const foo = () => console.log("First");
const bar = () => setTimeout(() => console.log("Second"), 500);
const baz = () => console.log("Third");

bar();
foo();
baz();
```
Result
```bash
First
Third
Second
```

As I said before, we have callstack where our funcs are running and console logging. In the code, our "bar" func calling first, but it using setTimeout, which is a part of WEB API, so first we wait 500ms (that's also why our previous example works like that). But we do not wait for all funcs, only for this one "bar".

After we're calling foo and baz. They going directly into callstack and running instantly. In turn, "bar", after 500ms going to the queue, like other async funcs (using setTimeout and setInterval) and eventloop running them in order of their readyness.

Eventloop is just a worker, who getting ready func and running it to the callstack. Here the full step list:
- Calling bar, bar returns setTimeout
- In that case it's popping our the callstack and putting inside WEBAPI
- Timer tunning and now we run next func, here is no troubles, we getting "First" instantly. It's popping out the callstack and our "bar"'s callback going to the Queue
- baz logging "Third", now callstack is clear and Event Loop sees, that it's free and going to use all funcs inside Queue. Now event loop getting bar's callback and putting it to the callstack
- callback logs "Second"

# Cookies
I really dunno why they don't bring it to the standard library. They have tons of shit, but not this?
```js
function setCookie(cname, cvalue, exdays) {
  const d = new Date();
  d.setTime(d.getTime() + (exdays * 24 * 60 * 60 * 1000));
  let expires = "expires="+d.toUTCString();
  document.cookie = cname + "=" + cvalue + ";" + expires + ";path=/";
}

function getCookie(cname) {
  let name = cname + "=";
  let ca = document.cookie.split(';');
  for(let i = 0; i < ca.length; i++) {
    let c = ca[i];
    while (c.charAt(0) == ' ') {
      c = c.substring(1);
    }
    if (c.indexOf(name) == 0) {
      return c.substring(name.length, c.length);
    }
  }
  return "";
}

function checkCookie() {
  let user = getCookie("username");
  if (user != "") {
    alert("Welcome again " + user);
  } else {
    user = prompt("Please enter your name:", "");
    if (user != "" && user != null) {
      setCookie("username", user, 365);
    }
  }
}
```


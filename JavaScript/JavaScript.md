# JavaScript

## Introducing JavaScript objects

### JavaScript object basics

#### Object basics

creating an object often begins with defining and initializing a variable.

```
const person = {};
```

When the object's members are functions there's a simpler syntax. Instead of `bio:function()` we can write `bio()` .

```
const person = {
	name: ["Bob", "Smith"],
	age: 32,
	bio() {
		console.log(`${this.name[0]} ${this.name[1]} is ${this.age} years old.`);
	},
	introduceSelf() {
    console.log(`Hi! I'm ${this.name[0]}.`);
  },
};
```

Objects as object properties

```
const person = {
	name: {
		first: "Bob",
		last: "Smith",
	},
};
```

Introrducing constructors

```
function createPerson(name) {
	const obj = {};
	obj.name = name;
	obj.introduceSelf = function() {
		console.log(`Hi! I'm ${this.name}.`);
	};
	return obj;
}
```

A better way is to use a **constructor** . A constructor is just a function called using the `new` keyword. When you call a constructor, it will:

- create a new object
- bind `this` to the new object, so you can refer to `this` in your constructor code
- run the code in the constructor
- return the new object.

```
function Person(name) {
	this.name = name;
	this.introduceSelf = function() {
		console.log(`Hi! I'm ${this.name}.`);
	};
}
```

When you accessed the document object model using lines like this:

```
const myDiv = document.createElement("div");
const myVideo = document.querySelector("video");
```

### Object prototypes

#### The prototype chain

Every object in JavaScript has a built-in property, which is called its **prototype** . The prototype is itself an object, so the prototype will have its own prototype, making what's called a **prototype chain**. The chain ends when we reach a prototype that has `null` for its own prototype.

> Note: The property of an object that points to its prototype is not called `prototype` . Its name is not standard, but in practice all browsers use `__proto__` . The standard way to access an object's prototype is the `Object.getPrototypeOf()` method.

This is an object called `Object.prototype` , and it is the most basic prototype, that all objects have by default. The prototype of `Object.prototype` is `null` , so it's at the end of the prototype chain:

![](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/Object_prototypes/myobject-prototype-chain.svg)

#### Setting a prototype

Using Object.create

The `Object.create()` method creates a new object and allows you to specify an object that will be used as the new object's prototype.

```
const personPrototype = {
	greet() {
		console.log("hello!");
	},
};

const carl = Object.create(personPrototype);
carl.greet(); // hello!
```

Using a constructor

In JavaScript, all functions have a property named `prototype` . When you call a function as a constructor, this property is set as the prototype of the newly constructed object (by convention, in the property named `__proto__`).

```
const personPrototype = {
	greet() {
		console.log(`hello, my name is ${this.name}!`);
	},
};

function Person(name) {
	this.name = name;
}

Object.assign(Person.prototype, personPrototype);
```

Here we create:

- an object `personPrototype` , which has a `greet()` method
- a `Person()` constructor function which initializes the name of the person to create.

We then put the methods defined in `personPrototype` onto the `Person` function's `prototype` property using `Object.assign` .

Own properties

You can check whether a property is an own property using the static `Object.hasOwn()` method:

```
const irma = new Person("Irma");

console.log(Object.hasOwn(irma, "name")); // true
console.log(Object.hasOwn(irma, "greet")); // false
```

### Working with JSON

#### Converting between objects and text

- `parse()` : Accepts a JSON string as a parameter, and returns the corresponding JavaScript object.
- `stringify()` : Accepts an object as a parameter, and returns the equivalent JSON string.

we retrieve the response as text rather than JSON, by calling the `text()` method of the response.

we then use `parse()` to convert the text to a JavaScript object.

```
async function populate() {
  const requestURL =
    "https://mdn.github.io/learning-area/javascript/oojs/json/superheroes.json";
  const request = new Request(requestURL);

  const response = await fetch(request);
  const superHeroesText = await response.text();

  const superHeroes = JSON.parse(superHeroesText);
  populateHeader(superHeroes);
  populateHeroes(superHeroes);
}
```

```
let myObj = { name: "Chris", age: 38 };
myObj;
let myString = JSON.stringify(myObj);
myString;
```

## Asynchronous JavaScript

### How to use promises

#### Using the fetch() API

In an HTTP request, we send a request message to a remote server, and it sends us back a response.

```
const fetchPromise = fetch(
	"https://mdn.github.io/learning-area/javascript/apis/fetching-data/can-store/products.json",
);

console.log(fetchPromise);

fetchPromise.then((response) => {
	console.log(`Received response: ${response.status}`);
});

console.log("Started request...");
```

1. calling the `fetch()` API, and assigning the return value to the `fetchPromise` variable
2. immediately after, logging the `fetchPromise` variable. This should output something like: `Promise {<state>: "pending"}` , telling us that we have a `Promise` object, and it has a `state` whose value is `"pending"` . The `"pending"` state means that the fetch operation is still going on.
3. passing a handler function into the Prromise's `then()` method. When (and if) the fetch operation succeeds, the promise will call our handler, passing in a `Response` object, which contains the server's response.
4. logging a message that we have started the request.

The complete output should be something like:

```
Promise {<state>: "pending"}
Started request...
Received response: 200
```

#### Chaining promises

With the `fetch()` API, once you get a `Response` object, you need to call another function to get the response data.

```
const fetchPromise = fetch(
  "https://mdn.github.io/learning-area/javascript/apis/fetching-data/can-store/products.json",
);

fetchPromise.then((response) => {
	const jsonPromise = response.json();
	jsonPromise.then((data) => {
		console.log(data[0].name);
	});
});
```

The elegant feature of promises is that `then()` *itself returns a promise, which will be completed with the result of the function passed to it.* 

```
const fetchPromise = fetch(
  "https://mdn.github.io/learning-area/javascript/apis/fetching-data/can-store/products.json",
);

fetchPromise
	.then((response) => response.json())
	.then((data) => {
		console.log(data[0].name);
	});
```

We need to check that the server accepted and was able to handle the request, before we try to read it.

```
const fetchPromise = fetch(
  "https://mdn.github.io/learning-area/javascript/apis/fetching-data/can-store/products.json",
);

fetchPromise
	.then((response) => {
		if (!response.ok) {
			throw new Error(`HTTP error: ${response.status}`);
		}
		return response.json();
	})
	.then((data) => {
		console.log(data[0].name);
	});
```

#### Catching errors

To support error handling, `Promise` objects provide a `catch()` method. This is a lot like `then()` : you call it and pass in a handler function. However, while the handler passed to `then()` is called when the asynchronous operation *succeeds* , the handler passed to `catch()` is called when the asynchronous operation *fails* .

```
const fetchPromise = fetch(
  "bad-scheme://mdn.github.io/learning-area/javascript/apis/fetching-data/can-store/products.json",
);

fetchPromise
	.then((response) => {
		if (!response.ok) {
			throw new Error(`HTTP error: ${response.status}`);
		}
		return response.json();
	})
	.then((data) => {
		console.log(data[0].name);
	})
	.catch((error) => {
		console.error(`Could not get products: ${error}`);
	});
```

#### Promise terminology

First, a promise can be in one of three states:

- **pending** : the promise has been created, and the asynchronous function it's associated with has not succeeded or failed yet. This is the state your promise is in when it's returned from a call to `fetch()` , and the request is still being made.
- **fulfilled** : the asynchronous function has successded. When a promise is fulfilled, its `then()` handler is called.
- **rejected** : the asynchronous function has failed. When a promise is rejected, its `catch()` handler is called.

#### async and await

The `async` keyword gives you a simpler way to work with synchronous promise-based code. 

```
async function myFunction() {
	// This is an async function
}
```

Inside an async function, you can use the `await` keyword before a call to a function that returns a promise. This makes the code wait at that point until the promise is settled, at which point the fulfilled value of the promise is treated as a return value, or the rejected value is thrown.

```
async function fetchProducts() {
  try {
    // after this line, our function will wait for the `fetch()` call to be settled
    // the `fetch()` call will either return a Response or throw an error
    const response = await fetch(
      "https://mdn.github.io/learning-area/javascript/apis/fetching-data/can-store/products.json",
    );
    if (!response.ok) {
      throw new Error(`HTTP error: ${response.status}`);
    }
    // after this line, our function will wait for the `response.json()` call to be settled
    // the `response.json()` call will either return the parsed JSON object or throw an error
    const data = await response.json();
    console.log(data[0].name);
  } catch (error) {
    console.error(`Could not get products: ${error}`);
  }
}

fetchProducts();
```

Also, note that you can only use `await` inside an `async` function, unless your code is in a JavaScript module.

```
try {
  // using await outside an async function is only allowed in a module
  const response = await fetch(
    "https://mdn.github.io/learning-area/javascript/apis/fetching-data/can-store/products.json",
  );
  if (!response.ok) {
    throw new Error(`HTTP error: ${response.status}`);
  }
  const data = await response.json();
  console.log(data[0].name);
} catch (error) {
  console.error(`Could not get products: ${error}`);
}
```


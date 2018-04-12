# TypeScript

TypeScript is typed superset of JavaScript originally developed by Microsoft. It compiles to JavaScript for either Browsers or Node. This library includes three parts:

1. The language itself
2. The language service which provides a good developer experience.
3. The compiler

TypeScript also implements modern language features so it is very important to draw the distinction between what is TS and what is ES-next.

### Motivation for TypeScript

In terms of value types, JavaScript is pretty limited. It has six primitive types such as string, number, null, etc. and then everything else extends from Object. And even though the language has some types, it is a mutable type and you cannot specify a type upon variable instantiation. This makes it harder to reason about our code because we or anyone else can change the type of a variable at any time, potentially breaking code down later on in the pipeline. It also makes code more self-documenting because when we read code for the first time, we know what type a variable or parameter is supposed to be.

Put fancily in a one-liner, the power of types is that it makes code more expressive of our intent. Not only that, it works to enforce that intent through preventing type coercions and reassignments. That being said, it is important not to over-type your code. It is designed to enforce constraints, and overly constraining your code can actually make it harder to work with.

## Attribution

All of the material contained in this document has been derived from the following sources:

* [The official TypeScript website](https://www.typescriptlang.org/index.html)
* The [Front-end Masters](https://github.com/JimRottinger/learnings/tree/a0638c9e3ec90727947df413490efbdb5b942d53/TypeScript/www.frontendmasters.com) workshop on TypeScript, presented by [Mike North](https://github.com/mike-north). Many of the exercises in these notes are pulled directly from this source. I would highly recommend checking it out as Mike explains concepts at a much deeper level than I take notes on here.

## How it Works

TypeScript has **implicit typing** - it can make good guesses at types through the assignment.

```javascript
let teacherAge = 34;
teacherAge = '35'; //type string is not assignable to a number
```

It also has **explicit type assignment**.

```javascript
let teacherAge: number = 34; //type annotation form of explicit casting
let input = document.querySelector('input#name_field') as HTMLInputElement //'as' keyword
let input2 = <HTMLInputElement>document.querySelector('input#name_field') //casting
```

The **type annotation form can be used for function parameters and function return values.**

```javascript
function login(username: string, password: string) : User {
  //do something and return a User object
}
```

_At this point you should should be able to complete exercise 1 in the exercises file._

## Object Shape

So under the hood, how does it check that the type is correct? Compiler-wise, there are two approaches to this problem:

1. **Nominal type systems** - checks that the value is of the same class/type based on name
2. **Structural type systems** - only cares about the shape of an object

The second option is how TypeScript works, but what do we mean by shape? The shape of an object is the names of the properties along with the types of the properties. With that in mind, we can create an object with the same shape as another and it will pass the type checks. This is a good thing! - because we can extend some minimal set of properties and an instance of the extended object will still pass the type check. Put more clearly, in terms of typing, an object can have a superset of the required properties, but not a subset, in order for the typing to pass.

### The Object Literal Difference

I just stated that in terms of type checking, we only care about the minimal set of properties and any excess properties are okay. This is actually only true for function parameters. When it comes to defining object literals, things are a bit more strict. Here is an example.

```javascript
let myCar: { make: string, model: string, year: number};

myCar = {
  make: 'Hyundai',
  model: 'Elantra',
  year: 2015,
  color: {r: 0, g: 0, b: 200}
}; //not assignable error
```

One way to solve this is to make color an optional type \(which we haven't learned how to do yet\), the other way is through an interface.

## Interfaces

An interface describes the structure of an object and have no implementation. To frame it in terms of other programming languages, it would be a pure abstract class.

```javascript
interface Car {
  make: string,
  model: string,
  year: any //wildcard type
}
```

For an example of when interfaces would be very useful, let's talk about a front-end application in which we have two types of users, regular users and admins. Users and admins are very similar, except that admins have a few special properties that users do not have. In this case, an Admin interface can extend the User interface and add a few additional properties. Then, for all functions that take in a User as a parameter, we could pass in an Admin instead because Admins are supersets of Users. Similarly, a function that takes in an admin would reject a normal user because a user does not fit the full shape of a Admin.

```javascript
interface User {
  email: string;
  password: string;
}

interface Admin extends User {
  adminSince: Date; //any constructor can be a type
  hasBanHammer: boolean;
}
```

### Type Aliases

Sometimes, creating an entire iterface isn't necessary to define a structure. In these cases, the `type` keyword can be used to define a type alias.

```javascript
type RGBColor = [number, number, number];
let red: RGBColor = [255, 0, 0];
```

Types can be exported, just like interfaces, so they can be included wherever they are needed.

_At this point you should be able to complete exercise 2._

## Classes & Property Types

As soon as we start mentioning interfaces and extends \(inheritance\), we have to include a discussion on Classes. ES6 introduced the Class syntax in JavaScript, so how does TypeScript tie into that? Let's start off with an example of a class.

```javascript
class Car {
  constructor(make, model) {
    this.make = make;
    this.model = model;
  }

  toJSON() {
    return {
      make: this.make,
      model: this.model
    }
  }
}

class Truck extends Car {
  constructor(make, model, numWheels) {
    super(make, model);
    this.numWheels = numWheels;
  }

  toJSON() {
    let properties = super.toJSON();
    properties.numWheels = this.numWheels;

    return properties;
  }
}

let myTruck = new Truck('Ford', 'F150', 6);

console.log(myTruck.toJSON()); //{make: "Ford", model: "F150", numWheels: 6}
```

In this example, we see the class Truck extending a base-class Car. In this Truck's function, we use the keyword `super` to call up to the Car's functions. We can also see that Class's constructor defines its shape as well. We can use TypeScript to mark this up with types.

```javascript
class Car {
  make: string
  model: string
  constructor(make:string, model:string) {
    this.make = make;
    this.model = model;
  }

  toJSON() {
    return {
      make: this.make,
      model: this.model
    }
  }
}

class Truck extends Car {
  numWheels: number
  constructor(make:string, model:string, numWheels:number) {
    super(make, model);
    this.numWheels = numWheels;
  }

  toJSON() {
    let properties = super.toJSON();
    properties.numWheels = this.numWheels;

    return properties;
  }
}

let myTruck = new Truck('Ford', 'F150', 6);

console.log(myTruck.toJSON()); //{make: "Ford", model: "F150", numWheels: 6}
```

Be sure to add type annotations to both the properties and the construction arguments.

### Enums

It is pretty common to want to constrain the possible values of a property to a specific set of values. For this case, TypeScript defines the `enum` keyword. `enum` is used to define a type consisting of ordered members.

```javascript
enum ShippingStatus {
  Pending,
  BeingPrepared,
  Shipped,
  Delivered
};

class Order {
  shippingStatus: ShippingStatus

  constructor(shippingStatus:ShippingStatus) {
    this.shippingStatus = shippingStatus;
  }
}
```

### Arrays

In vanilla JavaScript, it is possible to have an array with a mixed bag of types such as `[1,2,'not a number']`. With TypeScript, it is possible to constrain an array to a single type.

```javascript
let nums: number[] = [1,2,3];
```

There is a special type of an array called a **tuple**. A tuple is an array of fixed length, usually in a specific order with a specific type for each value. In that sense, they are a lot like objects, but they can shine when used in combination with destructuring. Here is an example of an array of array of dependencies where each value is an array with a package name and a version number.

```text
let dependencies: [string, number][] = [];
```

## Functions

Up to this point, we have been talking about TypeScript in terms of defining object or class structure, but what about functions? Well, functions have a type just like any other value. It's function signature defines its expected arguments, the types of those arguments, and the return value of the function. Here is how you define a function type:

```javascript
let login: (username: string, password: string) => User;
```

Just as we saw above, in addition to types, we can also define the interface of a function.

```javascript
interface ClickListener {
  (this: Window, e: MouseEvent): void
}

const myListener: ClickListener = function(e) {
  console.log(e);
}
```

One super interesting thing to note about the above code example is that we can actually check the type of `this` for a function. For the `ClickListener` interface, we expect `this` to be a Window object. This prevents us from calling `myListener` directly. It has to be called in context of a Window.

### Function Arguments

Unless you say otherwise, TypeScript assumes every argument in the function is required. Here is an example of how to make an optional argument in TypeScript:

```javascript
function createUser(name: String, email: String, profileUrl ?: URL)
```

What this implies is that the type of profileUrl is `URL | null`. Then, if you tried to assign profileUrl to a variable of type `URL`, it will not work because profileUrl could potentially be null and you cannot assign null to a variable of type URL. This prevents us from making bad assumption about our arguments.

## Generics

Generics are a TypeScript-only construct because they have everything to do with types. Generics allow us to reuse code across many types, interfaces, and functions. Consider the following code example:

```javascript
function gimmieFive<T>(x: T): T[] {
  return [x,x,x,x,x];
}
let threes: number[] = gimmieFive(3);
let eggs: string[] = gimmieFive('egg');
```

In this example, `T` is a generic whose type is determined inplicitly by the argument that is passed into the `gimmieFive` function. If you pass in 3, T will be number and if you pass in 'egg', T will be string. It should be noted there is nothing special about the T. T could be anything.

```javascript
function gimmieFive<SomeGenericName>(x: SomeGenericName): SomeGenericName[] {
  return [x,x,x,x,x];
}
let threes: number[] = gimmieFive(3);
let eggs: string[] = gimmieFive('egg');

console.log(threes);
console.log(eggs);
```

### Constraints on Generics

Often we will want to put some kind of constraint on a generic in that in can accept some things but not all things.

```javascript
function midpoint<T extends Point2D>(p1: T, p2: T): T {
  return new Point2D(p1.x - p2.x, p1.y, p2.y);
}
```

In this example, T can still be anything so long as it conforms to the object shape of `Point2D`.

### Generic Classes

Generic classes have a generic type parameter list in angle brackets \(&lt;&gt;\) following the name of the class.

```javascript
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };
```

When you instantiate a generic class, you are supplying the type for the for that instance of the class to use. In this sense, the type\(s\) basically act like an argument to the constructor of your class.


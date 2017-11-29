# TypeScript

TypeScript is typed superset of JavaScript originally developed by Microsoft. It compiles to JavaScript for either Browsers or Node. This library includes three parts:

1. The language itself
2. The language service which provides a good developer experience.
3. The compiler

TypeScript also implements modern language features so it is very important to draw the distinction between what is TS and what is ES-next.

#### Motivation

In terms of value types, JavaScript is pretty limited. It has six primitive types such as string, number, null, etc. and then everything else extends from Object. And even though the language has some types, it is a mutable type and you cannot specify a type upon variable instantiation. This makes it harder to reason about our code because we or anyone else can change the type of a variable at any time, potentially breaking code down later on in the pipeline. It also makes code more self-documenting because when we read code for the first time, we know what type a variable or parameter is supposed to be.

Put fancily in a one-liner, the power of types is that it makes code more expressive of our intent. Not only that, it works to enforce that intent through preventing type coercions and reassignments. That being said, it is important not to over-type your code. It is designed to enforce constraints, and overly constraining your code can actually make it harder to work with.

## How it Works

TypeScript has **implicit typing** - it can make good guesses at types through the assignment.

```js
let teacherAge = 34;
teacherAge = '35'; //type string is not assignable to a number
```

It also has **explicit type assignment**.
```js
let teacherAge: number = 34; //type annotation form of explicit casting
let input = document.querySelector('input#name_field') as HTMLInputElement //'as' keyword
let input2 = <HTMLInputElement>document.querySelector('input#name_field') //casting
```

The **type annotation form can be used for function parameters and function return values.**

```js
function login(username: string, password: string) : User {
  //do something and return a User object
}
```

## Object Shape

So under the hood, how does it check that the type is correct? Compiler-wise, there are two approaches to this problem:

1. **Nominal type systems** - checks that the value is of the same class/type based on name
2. **Structural type systems** - only cares about the shape of an object

The second option is how TypeScript works, but what do we mean by shape? The shape of an object is the names of the properties along with the types of the properties. With that in mind, we can create an object with the same shape as another and it will pass the type checks. This is a good thing! - because we can extend some minimal set of properties and an instance of the extended object will still pass the type check. Put more clearly, in terms of typing, an object can have a superset of the required properties, but not a subset, in order for the typing to pass.

#### The Object Literal Difference

I just stated that in terms of type checking, we only care about the minimal set of properties and any excess properties are okay. This is actually only true for function parameters. When it comes to defining object literals, things are a bit more strict. Here is an example.

```js
let myCar: { make: string, model: string, year: number};

myCar = {
  make: 'Hyundai',
  model: 'Elantra',
  year: 2015,
  color: {r: 0, g: 0, b: 200}
}; //not assignable error
```

One way to solve this is to make color an optional type (which we haven't learned how to do yet), the other way is through an interface.

## Interfaces

An interface describes the structure of an object and have no implementation. To frame it in terms of other programming languages, it would be a pure abstract class.

```js
interface Car {
  make: string,
  model: string,
  year: any //wildcard type
}
```

## Exercises

#### Exercise 1

To start, copy the following code into a JS file.

```js
//should take in an objects with an r,g,and b property and return the hex string
function rgbToHex(/*Implement*/) {
  //implement
}

//should take in a 6 digit hex string and return an object with the properties r, g, and b
function hexToRgb(/*Implement*/) {
  //implement
}

let color = {
  r: 255,
  g: 0,
  b: 0,
  get hex() : string {
    return rgbToHex(this.r, this.g, this.b);
  },
  set hex(hex: string) {
    let { r, g, b } = hexToRgb(hex);
    this.r = r;
    this.g = g;
    this.b = b;
  }
};

console.log(rgbToHex(255,100,55)); //should be "ff6437"
console.log(hexToRgb('00ff00')); //should be {r: 0, g: 255, b: 0}
```

Then implement the two blank functions to convert from a hex string to a rgb object and vise-versa. Don't worry about covering every edge case such as if we are based in invalid string. Just implement a basic function with the correct type constraints.

#### Exercise 2

Convert the code of the solution to Exercise 1 to use an interface for the RGB object instead of listing out the properties every time.

## Solutions

#### Exercise 1 Solution

```js
//should take in an objects with an r,g,and b property and return the hex string
function rgbToHex(r: number, g: number, b: number) : string {
    return r.toString(16) + g.toString(16) + b.toString(16);
}

//should take in a 6 digit hex string and return an object with the properties r, g, and b
function hexToRgb(hexString: string) : { r: number, g: number, b: number } {
    return {
        r: parseInt(hexString.substr(0,2), 16),
        g: parseInt(hexString.substr(2,2), 16),
        b: parseInt(hexString.substr(4,2), 16)
    };
}

let color = {
  r: 255,
  g: 0,
  b: 0,
  get hex() : string {
    return rgbToHex(this.r, this.g, this.b);
  },
  set hex(hex: string) {
    let { r, g, b } = hexToRgb(hex);
    this.r = r;
    this.g = g;
    this.b = b;
  }
};

console.log(rgbToHex(255,100,55)); //should be "ff6437"
console.log(hexToRgb('00ff00')); //should be {r: 0, g: 255, b: 0}
```

#### Exercise 2 Solution

```js
interface RGBObject {
    r: number,
    g: number,
    b: number
}

//should take in an objects with an r,g,and b property and return the hex string
function rgbToHex(rgb: RGBObject) : string {
    return rgb.r.toString(16) + rgb.g.toString(16) + rgb.b.toString(16);
}

//should take in a 6 digit hex string and return an object with the properties r, g, and b
function hexToRgb(hexString: string) : RGBObject {
    return {
        r: parseInt(hexString.substr(0,2), 16),
        g: parseInt(hexString.substr(2,2), 16),
        b: parseInt(hexString.substr(4,2), 16)
    };
}

let color = {
  r: 255,
  g: 0,
  b: 0,
  get hex() : string {
      return rgbToHex({ r: this.r, g: this.g, b: this.b });
  },
  set hex(hex: string) {
    let { r, g, b } = hexToRgb(hex);
    this.r = r;
    this.g = g;
    this.b = b;
  }
};

console.log(rgbToHex({ r: 255, g: 100, b: 55})); //should be "ff6437"
console.log(hexToRgb('00ff00')); //should be {r: 0, g: 255, b: 0}
```

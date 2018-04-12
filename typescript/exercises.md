# TypeScript Exercises

## Exercise 1

To start, copy the following code into a JS file.

```javascript
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

## Exercise 1 Solution

```javascript
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

## Exercise 2

Convert the code of the solution to Exercise 1 to use an interface for the RGB object instead of listing out the properties every time.

## Exercise 2 Solution

```javascript
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


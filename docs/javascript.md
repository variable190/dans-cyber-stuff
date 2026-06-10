# JavaScript Cheat Sheet

## Variables and Strings

### Data types and variable names:

```js
let pet = {                 // Object
  name: "Fluffy",           // String
  age: 3,                   // Number
  type: "dog",
  isCat: false,             // Boolean
  dailyPooAverage: 2.3      // Floating point number
};

const veryBigNumber = 1234567890123456789012345678901234567890n; // BigInt

let variable;               // Undefined, let can be defined and reasigned later

let variable2 = null        // Null is an object data type
console.log(typeof user);   // "object", typeof returns the data type

const crypticKey1= Symbol("saltNpepper");   // Const variables are fixed and must be defined when declared
const crypticKey2= Symbol("saltNpepper");   
console.log(crypticKey1 === crypticKey2);   // returns false, Symbol data types are unique and immutable

// Variable names should use camelCase and should start only with a letter _ or $
// Variable names should only contain a-z, A-Z, 0-9, _ or $, no other special characters or spaces
// JavaScript is dynamically typed
```

### Strings

```js
// Concatination

let message = "Welcome to programming, ";
message += "Haz!";
console.log(message); // Welcome to programming, Haz!

let firstName = "John";
let lastName = "Doe";
let fullName = firstName.concat(" ", lastName);
console.log(fullName); // John Doe

const name = "Haz";
const farewell = "Bye, "; 
console.log(name + farewell + "!"); // "Bye, Haz!"

// Interpolation

const name = "Haz";
const greeting = `Hello, ${name}!`; // Interpolation uses backticks
console.log(greeting); // "Hello, Haz!"

// Manipulation

const developer = "Haz";
console.log(developer[0]); // H

const letter = "A";
console.log(letter.charCodeAt(0));  // 65, returns UTF-16 code unit of specified char

const char = String.fromCharCode(65);
console.log(char);  // A, visa versa

const text = "The quick brown fox jumps over the lazy dog.";
console.log(text.indexOf("fox")); // 16
console.log(text.indexOf("cat")); // -1

const text = "The quick brown fox jumps over the lazy dog.";
console.log(text.includes("fox")); // true
console.log(text.includes("cat")); // false

const text = "freeCodeCamp";
console.log(text.slice(0, 4));  // "free"
console.log(text.slice(4));  // "CodeCamp"
console.log(text.slice(-4));  // "Camp"

const text = "Hello, world!";
console.log(text.toUpperCase()); // "HELLO, WORLD!"

const text = "HELLO, WORLD!"
console.log(text.toLowerCase()); // "hello, world!"

const text = "I love cats and cats are so much fun!";
console.log(text.replace("cats", "dogs")); // "I love cats and dogs are so much fun!"

const text = "I love cats and cats are so much fun!";
console.log(text.replaceAll("cats", "dogs")); // "I love dogs and dogs are so much fun!"

const text = "Hello";
console.log(text.repeat(3)); // "HelloHelloHello"

const text = "  Hello, world!  ";
console.log(text.trim()); // "Hello, world!"

const text = "  Hello, world!  ";
console.log(text.trimStart()); // "Hello, world!  "

const text = " Hello, world! ";
console.log(text.trimEnd()); // "  Hello, world!"

const answer = window.prompt("What's your favorite animal?", "dog"); // variable assigned by user response, dog is default (optional 2nd arg)
```


---
layout: post
title:      "JavaScript - Scopes and Hoisting"
date:       2019-06-22 01:07:40 -0400
permalink:  javascript_-_scopes_and_hoisting
---


JavaScript is a unique programming language with several features that can trip up new programmers. One of the most fundamental characteristics of a programming lanuage is how variables are scoped. This post explores two aspects of JavaScript related to scoping that can be confusing.

## SCOPES
Scopes define the accessibility of variables and functions for a given body of code. Scopes can be nested. If a variable or function is declared in a nested scope, it's not accessible from parent scopes.  Child scopes inherit from their parent scopes.

Scoping issues can occur with the misuse of variables.  When using ES6 syntax, it's best to always use `const` or `let` to avoid the most common bugs with variable scope.  Both `const` and `let` will throw an error if you try to declare the same variable twice.  `const` declarations will not change.  `const` should be used by default.  Never use `var` or declare a variable by name only.  Variables declared by name only are `globally-scoped` and they will not throw any errors if you declare the variable again.

![](https://stephaniemills.ca/dark-table-5ec7c8c60d7f366b7812e214d58a3c3d.svg)

### FUNCTION SCOPE
A function scope is an execution context defined by a function body.  Variables declared with `var` use this scope and are visible within the entire current function.  Outside of the function, we are not able to access anything declared inside.

EXAMPLES
```
function functionScope() {
  var x = 4;
  console.log(x);
}

functionScope();
//=> 4
```
```
function functionScope() {
  if (true) {
    var x = 4;
  }
  console.log(x);
}

functionScope();
//=> 4
```

### BLOCK SCOPE
A block scope is an execution context defined by a code block.  A code block is `if`, `switch`, `for`, `while` and function statements.  Variables declared using `const` or `let` use the rules of block scoping.  Outside of the code block, we are not able to access anything declared inside.

EXAMPLES

```
function block() {
  const localVariable = 31;

  return localVariable + 2;
}
 
block();
//=> 33
```

```
function block() {
  var localVariable = "inside function";
  console.log(localVariable);
}

console.log(localVariable);  
//=> error
```
```
function block(){
  if (true) {
      const sandwich1 = 'peanut butter';
      const sandwich2 = 'blt';
  }
  console.log(sandwich1);
  console.log(sandwich2);
}

block();
//=> error: sandwich1 is not defined
//=> error: sandwich2 is not defined
```

## HOISTING
Hoisting is how functions and variable declarations are moved to the top of their current scope.  During the compliation phase, the engine parses all of the declarations before executing any code.  This allows you to call a function before you declare it in your code.  The engine than starts over at the top of the code and executes each line. 

There are two rules to stick to, to avoid any problems with your code:

1. Declare functions and variables before using them.
2. Only use `const` and `let` to declare variables.  Do not use `var`.  Variables declared using `var` are hoisted to the top of the function body. Sticking to `const` and `let` completely avoids any variable hoisting.

The two examples below work the same way.  The last one calls the function before it is written.

```
function whatsForDinner(food) {
  console.log("I'm having " + food + " for dinner.");
}

whatsForDinner("BBQ");

//=> I'm having BBQ for dinner.
```

```
whatsForDinner("BBQ");

function whatsForDinner(food) {
  console.log("I'm having " + food + " for dinner.");
}

//=> I'm having BBQ for dinner.
```

Only declarations are hoisted, not initializations.  

Declaration:
```
let dog;
```

Initialization:
```
dog = "Rupert";
```

Declaration and Initialization:
```
let dog = "Rupert";
```

### HOISTS
```
cat = "Milo"; //=> initialization
console.log(cat); //=> Milo
var cat; //=> declaration
```
```
const city1 = "Miami"; //=> declaration and initialization
city2 = "Chicago"; //=> initialization
console.log(city1, city2); //=> "Miami, Chicago"
var city2; //=> declaration
```
```
music1 = 'LCD Soundsystem'; //=> initialization
music2 = 'Interpol'; //=> initialization
console.log(music1, music2); //=> "LCD Soundsystem, Interpol"
var music1, music2; //=> declaration
```

### DOES NOT HOIST
```
console.log(dog);  //=> error
let dog; //=> declaration
dog = "Rupert"; //=> initialization
```
```
const fruit1 = "strawberries"; //=> declaration and initialization
console.log("My favorite fruits are " + fruit1 + " and  " + fruit2); //=> error
const fruit2 = "apples" //=> initialization
```

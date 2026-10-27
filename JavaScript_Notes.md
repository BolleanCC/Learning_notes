// Functions
function name(parameter1, parameter2, ... parameterN) {
 //body
}
// Local variables
A variable declared inside a function is only visible inside that function
For example:
function showMessage() {
let message = "Hello,World"; // local variable
alert( message );
}
showMessage(); // Hello, World
alert( message ); // Error! The variable is local to the function

// Variable shadowing
let userName = "John";

function showMessage() {
let userName = "Bob"; // declare a local variable
let message = 'Hello, ' + userName; // Bob
alert( message )
}

showMessage();
alert(userName); // John


// Function Expression
let sayHi = function() {
alert("Hello");
}
Please note, there’s no name after the function keyword. Omitting a name is allowed for Function Expressions.
Here we immediately assign it to the variable, so the meaning of these code samples is the same: “create a function and put it into the variable sayHi”.

We can copy a function to another variable:
function sayHi() {   // (1) create
  alert( "Hello" );
}

let func = sayHi;    // (2) copy

func(); // Hello     // (3) run the copy (it works)!
sayHi(); // Hello    //     this still works too (why wouldn't it)

Line (2) copies it into the variable func. Please note again: there are no parentheses after sayHi. If there were, then func = sayHi() would write the result of the call sayHi() into func, not the function sayHi itself.

Callback functions
function ask(question, yes, no) {
if
}
Regular values like strings or numbers represent the data.

A function can be perceived as an action.

Function Expression vs Function Declaration
// Function Declaration
function sum(a, b) {
  return a + b;
}

// Function Expression
let sum = function(a, b) {
  return a + b;
};

A Function Expression is created when the execution reaches it and is usable only from that moment.
A Function Declaration can be called earlier than it is defined.
sayHi("John"); // Hello, John

function sayHi(name) {
  alert( `Hello, ${name}` );
}

sayHi("John"); // error!

let sayHi = function(name) {  // (*) no magic any more
  alert( `Hello, ${name}` );
};

A Function Declaration is hoisted and can be called before it's defined, while a Function Expression can only be used after it's assigned to a variable.

Arrow Functions
let func = (arg1, arg2, ..., argN) => expression;

In other words, it’s the shorter version of:
let func = function(arg1, arg2, ..., argN) {
  return expression;
};

let sum = (a, b) => a + b;

/* This arrow function is a shorter form of:

let sum = function(a, b) {
  return a + b;
};
*/

alert( sum(1, 2) ); // 3

Regular functions have their own this, which depends on how they are called. Arrow functions don't have their own this; they inherit it from the surrounding scope.
For instance, we can use it to iterate inside an object method:
let group = {
  title: "Our Group",
  students: ["John", "Pete", "Alice"],

  showList() {
    this.students.forEach(
      student => alert(this.title + ': ' + student)
    );
  }
};

group.showList();

If we used a “regular” function, there would be an error:
let group = {
  title: "Our Group",
  students: ["John", "Pete", "Alice"],

  showList() {
    this.students.forEach(function(student) {
      // Error: Cannot read property 'title' of undefined
      alert(this.title + ': ' + student);
    });
  }
};

group.showList();

Debugging in the browser





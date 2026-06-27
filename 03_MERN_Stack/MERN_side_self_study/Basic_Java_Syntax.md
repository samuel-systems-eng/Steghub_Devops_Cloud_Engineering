# MERN SELF STUDY

## Question:
Practice basic javascript syntax

## Answer:

JavaScript is a versatile and widely-used programming language primarily employed for adding interactivity to web pages.  

It's a server side (requiring a runtime environment) and a client-side scripting language, meaning it can runs in the user's browser as well as on a server.  

JavaScript is the programming language used to build the MERN stack. Unlike Java, it is dynamically typed and runs directly in the browser or via Node.js.

**Variables**

1. `let`: Declares a block-scoped variable that can change.

2. `const`: Declares a block-scoped variable that cannot change.

*Example*

`let` count = 10;  

`const` greeting = "Hello World";

**Data Types**

1. `String`: Text data enclosed in quotes.  

2. `Number`: Integer or floating-point numbers. 

3. `Boolean`: true or false values. 

4. `Array`: Ordered list of values. 

5. `Object`: Key-value pairs for complex data. 

*Example*

`let` name = "Alex";  

`let` age = 25;

`let` isStudent = true;

`let` colors = ["red", "blue"];

`let` user = { id: 1, role: "admin" };  


**Conditionals**

1. `if`: Executes a block if a condition is true.  
   
2. `else if`: Tests a new condition if the first is false.  
   
3. `else`: Executes if all previous conditions are false.  

*Examples*

`if` (age >= 18)  
{console.log("Adult");}  
`else`  
 {console.log("Minor");}

**Loops**

1. `for`: Runs a block of code a set number of times.  
   
2. `while`: Runs a block of code while a condition is true.  
   
*Example*  

`for` (let i = 0; i < 3; i++) {  
    console.log(i);  
}    

**Functions**

    • Standard Function: Uses the function keyword.  

    • Arrow Function: Shorter syntax widely used in modern MERN development.  

*Example*

// Standard  
`function` add(a, b)  
{ return a + b;}

// Arrow  
`const` multiply = (a, b) `=>` a * b;  


Console Output

`console.log()`: Prints output to the developer console.

*Example*

console.log(greeting);
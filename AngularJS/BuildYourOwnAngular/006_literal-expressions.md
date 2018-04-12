# Chapter 6 - Literal Expressions

This is the first chapter in Part 2 of the book titled Expressions and Filters. Expressions provide a frictionless access to the core of Angular's scope system. They allow us to concisely access and manipular data on scopes and run computations on it.

In Angular, the value of expressions is to bind data and behavior to HTML markup in directives like ngClass or ngClick, and we use them binding data to the contents and attributes of DOM elements with the interpolation syntax `{{ }}`.

In this part, we will also be talking about filters, which are functions we run by adding the unix style pipe character `|` to modify the return value. 

## A Whole New Language

So are the expressions just JavaScript code? Actually, they can be better thought of as a whole new language that is modeled after JavaScript. In other words, expressions are almost pure JavaScript, but it actually extends JavaScript, such as the `|` operation which isn't in native JS.

This expression language is parsed by JavaScript. What we need to do now is write a parser that takes in a string and returns a JavaScript function that can be executed. This brings us into the world of parsers, lexers, and abstract syntax trees. Let's get to it.

## Literal Expressions


#  A brief review of CoffeeScript

Before we move ahead we need to review some CoffeeScript and JS concepts.
For the most part, CoffeeScript can be read by any programmer, but there are a few quirks related to JS that must be 
known in order to understand certain bits of code.

## Classes and Prototypes

The first quirk with CoffeeScript is how it handles classes. JavaScript has no concept of classes, so classes in CoffeeScript are implemented using prototypal inheritance. 

If we define a class like this:

{lang=coffeescript}
~~~~~~~  
class Cat
~~~~~~~  

CoffeeScript will generate the following JS:

{lang=js}
~~~~~~~  
    var Cat;
    Cat = (function() {
      function Cat() {}
      return Cat;
    })();
~~~~~~~

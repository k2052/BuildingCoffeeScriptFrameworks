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

We get an anonymous function that returns a Function called `Cat`. In JS everything (almost) is an object.
Which is why we can so easily add new properties and functions to data. If you ever passed a config object into 
a jQuery plugin with callback functions right there, you've experience the object nature of JS. In JS we can define a new object "type" by defining a constructor function with the same as that type. For example, we could do;

{lang=js}
~~~~~~~ 
function Cat(pawsCount)
{
  this.pawsCount = pawsCount;
}
var catBus = new Cat(6);
~~~~~~~ 

A> Notice how `this` points the current "instance" of the object.

Let's add some attributes to our class and see what happens to the resulting generated code.

{lang=coffeescript}
~~~~~~~ 
class Cat
  legs: 0
  name: null
  @class_attribute: "classy"

  constructor: (name) ->
    @name = name

  talk: () =>
    console.log("Hey, I'm a Cat. Feed me, jerk. I'm just going put these paws on your face. 
                 Yes, the paws I use in the litter box. Shut up, you know you want it.")
~~~~~~~ 

Then the following JS is generated:

{lang=js}
~~~~~~~ 
(function() {
  var Cat,
    __bind = function(fn, me){ return function(){ return fn.apply(me, arguments); }; };

  Cat = (function() {
    Cat.prototype.legs = 0;

    Cat.prototype.name = null;

    Cat.class_attribute = "classy";

    function Cat(name) {
      this.talk = __bind(this.talk, this);
      this.name = name;
    }

    Cat.prototype.talk = function() {
      return console.log("Hey, I'm a Cat. Feed me, jerk. I'm just going put these paws on your face.                  Yes, the paws I use in the litter box. Shut up, you know you want it.");
    };

    return Cat;

  })();

}).call(this);
~~~~~~~ 

Instance methods and properties are set on the `prototype` and the prototype gets passed as `this` to any of the instance methods. 

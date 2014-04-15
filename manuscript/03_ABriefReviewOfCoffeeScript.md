#  A brief review of CoffeeScript

Before we move ahead we need to review some CoffeeScript and JS concepts.
For the most part, CoffeeScript can be read by any programmer, but there are a few quirks related to JS that must be 
known in order to understand certain bits of code.

## Classes

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

## Inside the prototype

The best way to understand a prototype is to push it and see what quirks pop up. 

Let's say we have the following object:

{lang=coffeescript}
~~~~~~~  
obj = {
  foo: 'bar'
}
~~~~~~~ 

Then we create a reference to it:

{lang=coffeescript}
~~~~~~~  
newObj = obj
~~~~~~~ 

Then we change the value of foo:

{lang=coffeescript}
~~~~~~~  
newObj.foo = 'cats'
~~~~~~~ 

What is the value of `obj.foo`? Is it `foo` or `cats`? It is `cats`. 
When we created `newObj` it was just a reference; so any modifications made to it will be reflected in obj. Cloning an object's prototype and sharing properties across the clones is referred to by the convoluted name "Differential Inheritance"

JS objects can delegate properties to their prototype and the protoype of that can do the same, _it's prototypes all the way up_ to Object.prototype. When you read a property from an object it first tries to read from the property of that object but if that doesn't exist, it then checks the prototype of that object and on and on until it reaches the prototype of _Object_ This means that objects can share properties by sharing prototypes. If you define update methods on the prototype and only use those methods to update the objects, then you can update all the clones by just updating one.

A> If you have ever been confused by clone vs deep clone the above should make it clear. A deep cloned item is an 
A> object with a unique prototype; this prevents JS from continue to look up the inheritance chain.

Let's implement a basic cloning method; just for fun we will place it on the prototype of Object:

{lang=coffeescript}
~~~~~~~ 
Object.prototype.clone = ->
  clone = Object.create(this)

  return clone
~~~~~~~

This utilizes `Object.create` which is a somewhat recent addition to JS.  Object.create creates a new object using the specified prototype. Internally it looks like this:

{lang=coffeescript}
~~~~~~~ 
Object.create = (o) ->
  Func = ->
  Func.prototype = o
  new Func()
~~~~~~~

Then it all comes together like this:

{lang=coffeescript}
~~~~~~~ 
chai = require("chai")
chai.should()

describe "dog", ->
  it "should clone a dog and allow us to change the legs count on all dogs", ->
    Object.prototype.clone = ->
      clone = Object.create(this)

      return clone

    Dog = ->
      name = undefined
      Dog.prototype.legs = 4

    dog1 = new Dog()
    dog1.name = "Jerry"
    dog2 = dog1.clone()
    dog2.name.should.equal "Jerry"
    dog2.legs.should.equal 4
    
    dog1.legs = 8
    dog2.legs.should.equal 8   
~~~~~~~

A> You can find the source for this chapter at https://github.com/k2052/BuildingCoffeeScriptFrameworks.src/tree/chapter-3


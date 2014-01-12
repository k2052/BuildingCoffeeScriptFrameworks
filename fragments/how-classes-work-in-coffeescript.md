---
title: How Classes Work In CoffeeScript
---

If we define a class like this:

    class Cat

CoffeScript will generate the following JS:

    var Cat;
    Cat = (function() {
      function Cat() {}
      return Cat;
    })();

Cat is an anoymous function that returns a Function called Cat. I'll be honest with you, I don't really know what that means yet. Let's add some attributes to our class and see what happens to the resulting generated code.

    class Cat
      legs: 0
      name: null
      @class_attribute: "classy"

      constructor: (name) ->
        @name = name

      talk: () =>
        console.log("Hey, I'm a Cat. Feed me, jerk. I'm just going put these paws on your face. 
                     Yes, the paws I use in the litter box. Shut up, you know you want it.")

Then the following JS is generated:

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

Instance methods/attributes are set on the `prototype` and the prototype gets passed as `this` to any of the instance methods. This reveals that classes are just objects. 
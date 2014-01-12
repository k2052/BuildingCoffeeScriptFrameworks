---
title: Mixins Using CoffeeScript
layout: mochacoffee
---

Mixins are accomplished by "inserting" methods onto a class from another class. 
So before we go any further, let's look at how we would "insert" a function into a class. 

First we need a function:

    something: () ->
      return "something"
{: .language-coffeescript}

Then we need a class:

    class Cat
{: .language-coffeescript}

How then to add the method _something()_ to the class? If we [remember](linke-to-post-on-classes) that classes are just objects, it's easy to see the solution.

    Cat['something'] = something
{: .language-coffeescript}

We now can call `something()` on Cat.

    it "should return something", ->
      Cat.something().should.equal "something"
{: .language-coffeescript.exec}

What if we wanted to add the method to the instance instead? So we could do `cat_instance.something()`. It's rather
straightforward. We just have to remember that instance methods are placed on Cat.prototype and to access the prototype
we need to use `::`

    Cat::[something] = something
{: .language-coffeescript}

Then we can use the method on instances:

    it "should return something", ->
      cat = new Cat()
      cat.should.equal "something"
{: .language-coffeescript.exec}

Constructing a mixin base class from here, is a piece of cake.

Define the class

    class Module
{: .language-coffeescript}

Declare a method for including instance methods

      @include: (obj) ->
{: .language-coffeescript}

Loop through the keys (functions and properties etc) in the obj and set them on this.prototype

        for key, value of obj
          @::[key] = value
{: .language-coffeescript}

Declare a method for including class methods

      @extend: (obj) ->
        for key, value of obj 
          @[key] = value
{: .language-coffeescript}

Add some classes using the mixin class as a base

    class Animal
      talk: () ->
        return "growl!"

      @isInsect: () ->
        return false

    class Cat extends Mixin
      @include Animal
      @extend Animal
{: .language-coffeescript}

    it 'should not be an insect', ->
      Cat.isInsect().should.equal false

    it 'should be able to talk', ->
      cat = new Cat
      cat.talk().should.equal "growl!"
{: .language-coffeescript.exec}

This is great for an inheritance model, but breaks down a bit in other use cases. What if we wanted to generate new classes dynamically that are a mix of two or more classes? Since classes in JS are just objects, the solution is pretty straightforward.

Declare a function for mixing class. Take a base class and a series of classes to mixin.

    mixOf = (base, mixins...) ->
{: .language-coffeescript}

Declare a class called `Mixed` which we will mix the other classes into. It will be returned as the new class from our function.

      class Mixed extends base
{: .language-coffeescript}

Loop through the mixins and set them on the prototype of the Mixed class

      for mixin in mixins by -1 # earlier mixins override later ones
        for name, method of mixin::
          Mixed::[name] = method
{: .language-coffeescript.exec}

Finally return the Mixed class

      return Mixed
{: .language-coffeescript}

Let's declare some classes to Mixin.

    class Otter
      hasFur: () ->
        return true

    class Duck   
      hasBill: () ->
        return true 
{: .language-coffeescript}

Then mix them together into a new class

    class Platypus extends mixOf Otter, Duck

    it 'should have fur and a bill', ->
      platypus = new Platypus
      platypus.hasFur().should.equal true
      platypus.hasBill().should.equal true
{: .language-coffeescript.exec}

In the last few examples we have seen a lot of the `extends` keyword. Obviously, Javascript does not have this built in, so how is it implemented? Understanding how it works will be important for understanding the final bits of our mixin recipe.

The following structure:

    class Animal

    class Otter extends Animal
{: .language-coffeescript}

Results in the following JS:

    (function() {
      var Animal, Otter, _ref,
        __hasProp = {}.hasOwnProperty,
        __extends = function(child, parent) { for (var key in parent) { if (__hasProp.call(parent, key)) child[key] = parent[key]; } function ctor() { this.constructor = child; } ctor.prototype = parent.prototype; child.prototype = new ctor(); child.__super__ = parent.prototype; return child; };

      Animal = (function() {
        function Animal() {}

        return Animal;

      })();

      Otter = (function(_super) {
        __extends(Otter, _super);

        function Otter() {
          _ref = Otter.__super__.constructor.apply(this, arguments);
          return _ref;
        }

        return Otter;

      })(Animal);

    }).call(this);
{: .language-javascript}

The key is the following bit of JS:

    __extends = function(child, parent) { for (var key in parent) { if (__hasProp.call(parent, key)) child[key] = parent[key]; } function ctor() { this.constructor = child; } ctor.prototype = parent.prototype; child.prototype = new ctor(); child.__super__ = parent.prototype; return child; };
{: .language-javascript}

This is very very similar to the extend method we constructed earlier, just a bit more verbose and a lot more JSy. It does the following;

Takes the properties of the parent class and sets them on the child class.

    for (var key in parent) { if (__hasProp.call(parent, key)) child[key] = parent[key]; }
{: .language-javascript}

Then it sets the child's prototype to the parent's prototype

    ctor.prototype = parent.prototype; child.prototype = new ctor(); 
{: .language-javascript}

It then sets the super of the child to the parent's prototype

    child.__super__ = parent.prototype;
{: .language-javascript}

Before we move on we need to understand a bit about scope. We have seen a lot of code like: 

    (function() {

    }).call(this);
{: .language-javascript}

This is a _Immediately-invoked function expression_ or _IIFE_. An IIFE is typically used to maintain scope and make sure `this` points to the correct object. To understand why this is necessary we need to understand a bit more about JS' `this`. `this` in javascript is quite a bit different than the `self` or `this` you might be used to from other languages.

What `this` refers to in JS can change depending on the context in which the function is executed. The best way to see how is through some examples.

Let's say we have a function that returns a value:

    v = 1
    getValue: () ->
      return v

    v = 2

    it 'should return 2', ->
      getValue().should equal 2
{: .language-coffescript.exec}

Now let's say we wanted to preserve the value of v even if we set it. (Even though it's redundant in this case it can 
be extremely useful)

    v = 1
    getValue = do(v) ->
      ->  
        v

    v = 2

    it 'should return 1', ->
      getValue().should equal 1
{: .language-coffescript.exec}

The resulting JS is:

    getValue = (function(v) {
      return function() {
        return v;
      };
    })(v);
{: .language-javascript}

*Note*: The do keyword is a special syntax that just allows us to construct closures without confusing the coffeescript parser. The following code looks like it might work but the resulting JS would move the out of place,

      getValue = (() ->
        -> 
          return v
      )(v)
{: .language-coffescript}

Resulting in the following JS

    getValue = (function(v) {
      return function() {
        return v;
      };
    }, v);
{: .language-javascript}

I apologize for that brief but necessary detour, now where were we? Oh yes, `v` has just been scoped correctly by passing it into an _IIFE_. Where this (no pun intended) gets interesting is when we start modifying and controlling _this_. Let's take a look at a counter example.

    counter = {
      val: 0

      increment: () ->
        this.val += 1
    }
    
    counter.val.should.equal 0
    counter.increment()
    counter.val.should.equal 1
{: .language-coffescript.exec}

We can see how this works by changing the execution context of `increment()`:

    counter = {
      val: 0

      increment: () ->
        this.val += 1
    }
    inc = counter.increment
    inc()
    counter.val.should.equal 0
{: .language-coffescript.exec}

The pointer to increment ("inc") changes the scope of `this` in increment to the global object.
When increment tries to increment this.val it encounters undefined. We can modify the context of inc by using call.

    this.val = 2

    counter = {
      val: 0

      increment: () -> 
        this.val += 1
    }

    inc = counter.increment
    inc.call(this)

    this.val.should.eq 3
    counter.increment()
    counter.val.should eq 1
{: .language-coffescript.exec}

Another way to alter the scope of an object is through the use of the `new` operator. It returns an object with all the properties, functions, values etc scoped to that object.

    Cat = (name) ->
      this.name = name

    cat = new Cat("Doug")
    
    it 'should return the name', ->
      cat.name.should.equal "Doug"
{: .language-coffescript.exec}

We can access the prototype of this object to add methods and properties. For example:

    Cat = (name) ->
      this.name = name

    Cat::talk = () ->
      return "Meow!"

    it 'should talk', ->
      cat.talk().should.equal "Meow!"
{: .language-coffescript.exec}

Now that we have laid the groundwork, it's time to build a more robust include/extending module class.

Define some methods which we will exclude from including or extending:

    moduleKeywords = ['included', 'extended']
{: .language-coffescript}

Define our class:

    class Module
{: .language-coffescript}

Define an include method (for instance properties) that takes an obj (remember classes are just objects in JS)

      @include: (obj) ->
{: .language-coffescript}

Throw an error if we don't get passed an object

        throw new Error('include(obj) requires obj') unless obj
{: .language-coffescript}

Loop through the properties of the object and include them on this class' prototype (instance)

        for key, value of obj when key not in moduleKeywords
          @::[key] = value
{: .language-coffescript}

Use apply to call the `included` callback in the context of this class but only if the method exists.
The `included` callback allows a base class to be alerted when it has been included.

        obj.included?.apply(this)
{: .language-coffescript}

Return this so the method is chainable

        @
{: .language-coffescript}

Define an extend method (for class properties) that takes an obj (remember classes are just objects in JS)

      @extend: (obj) ->

Throw an error if we don't get passed an object

        throw new Error('extend(obj) requires obj') unless obj
{: .language-coffescript}

Loop through the properties of the object and include them in the class

        for key, value of obj when key not in moduleKeywords
          @[key] = value
{: .language-coffescript}

Use apply to call the `extended` callback in the context of this class but only if the method exists.
The `extended` callback allows a base class to be alerted when it has been extended.

        obj.extended?.apply(this)
{: .language-coffescript}

Return this so the method is chainable

        @
{: .language-coffescript}

One final thing, the prototypal nature nature of javascript makes namespacing things dirt simple. 
To have `Module` class inside of a scope we can just do:

    Ryggard = @Ryggard = {}

    # This line is for node.js/require.js stuff It essentially places Ryggard into the globals
    module?.exports = Ryggard
   
   # Set Module on Ryggard
   Ryggard.Module = Module
{: .language-coffescript}

And awesomeness!

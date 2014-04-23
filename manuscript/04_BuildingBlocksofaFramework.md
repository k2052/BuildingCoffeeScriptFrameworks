# Building Blocks of a framework

Now that we fully understand classes and prototypes it's time to bring the knowledge together and build the classes
our framework will rest upon. In this chapter we will build classes for handling Modules (mixins), Events, Routing and Data binding.

## Modules/Mixins

Mixins in JavaScript are merely the insertion of methods from one object into another. Let's start building our modules class by figuring out how to "insert" a function into a class.

### Injecting Methods

First we need a function:

{lang=coffeescript}
~~~~~~~
meow: () ->
  return "meow"
~~~~~~~

Then we need a class:

{lang=coffeescript}
~~~~~~~
class Cat
~~~~~~~

How then to add the method _talk()_ to the class _Cat_? If we remember that CoffeeScript classes are just objects, it's easy to see the solution; we simply need to set the _meow_ attribute of _Cat_ to the function:

{lang=coffeescript}
~~~~~~~
Cat['meow'] = meow
~~~~~~~

We now can call `meow()` on Cat:

{lang=coffeescript}
~~~~~~~
it "should meow", ->
  Cat.meow().should.equal "meow"
~~~~~~~

What if we wanted to add the method to the instance instead? It's rather straightforward; we just have to remember that instance methods are placed on Cat.prototype and to access the prototype
we need to use `::`

{lang=coffeescript}
~~~~~~~
Cat::['meow'] = meow
~~~~~~~

Then we can use the method on instances:

{lang=coffeescript}
~~~~~~~
it "should meow", ->
  cat = new Cat()
  cat.should.equal "something"
~~~~~~~

I> The source for the above specs is available at [https://github.com/k2052/BuildingCoffeeScriptFrameworks.src/blob/chapter-4/spec/insertingMethods_spec.coffee](https://github.com/k2052/BuildingCoffeeScriptFrameworks.src/blob/chapter-4/spec/insertingMethods_spec.coffee)

### Building a Module Class

Constructing a mixin base class from here is a piece of cake. First define the module class:

{lang=coffeescript}
~~~~~~~
class Module
~~~~~~~

Now declare a method for including instance methods:

{lang=coffeescript}
~~~~~~~
  @include: (obj) ->
~~~~~~~

Loop through the keys (functions and properties etc) in the obj and set them on this.prototype:

{lang=coffeescript}
~~~~~~~
    for key, value of obj
      @::[key] = value
~~~~~~~

Now add a method for including class methods (aka extending):

{lang=coffeescript}
~~~~~~~
  @extend: (obj) ->
    for key, value of obj 
      @[key] = value
~~~~~~~

Add then we can use it like this:

{lang=coffeescript}
~~~~~~~
class Animal
  talk: () ->
    return "growl!"

  @isInsect: () ->
    return false

class Cat extends Module
  @include Animal
  @extend Animal

it 'should not be an insect', ->
  Cat.isInsect().should.equal false

it 'should be able to talk', ->
  cat = new Cat
  cat.talk().should.equal "growl!"
~~~~~~~

I> The source for the above spec is available at [https://github.com/k2052/BuildingCoffeeScriptFrameworks.src/blob/chapter-4/spec/moduleClass_spec.coffee](https://github.com/k2052/BuildingCoffeeScriptFrameworks.src/blob/chapter-4/spec/moduleClass_spec.coffee)

This is pretty good but we can do a lot better.

### A Class Mixer

What we just created is great for an inheritance model but breaks down a bit in other use cases. What if we wanted to generate new classes dynamically that are a mix of two or more classes? Since classes in JS are just objects, the solution is pretty straightforward.

First we need to declare a function for mixing classes. It should take a base class and a series of classes to mixin:

{lang=coffeescript}
~~~~~~~
mixOf = (base, mixins...) ->
~~~~~~~

Declare a class called `Mixed` which we will mix the other classes into. It will be returned as the new class from our function:

{lang=coffeescript}
~~~~~~~
  class Mixed
~~~~~~~

Loop through the mixins and set them on the Mixed class:

{lang=coffeescript}
~~~~~~~
  for mixin in mixins by -1
    for name, method of mixin:: # instance methods
      Mixed::[name] = method

    for name, method of mixin # class methods
      Mixed[name] = method  
~~~~~~~

Finally return the Mixed class:

{lang=coffeescript}
~~~~~~~
  return Mixed
~~~~~~~

Let's declare some classes to Mixin:

{lang=coffeescript}
~~~~~~~
class Otter
  hasFur: () ->
    return true

class Duck   
  hasBill: () ->
    return true 
~~~~~~~

Then mix them together into a new class:

{lang=coffeescript}
~~~~~~~
class Platypus extends mixOf Otter, Duck

it 'should have fur and a bill', ->
  platypus = new Platypus
  platypus.hasFur().should.equal true
  platypus.hasBill().should.equal true
~~~~~~~

I> The source for the above specs is available at [https://github.com/k2052/BuildingCoffeeScriptFrameworks.src/blob/chapter-4/spec/classMixer_spec.coffee](https://github.com/k2052/BuildingCoffeeScriptFrameworks.src/blob/chapter-4/spec/classMixer_spec.coffee)

### Inside "extends"

In the last few examples we have seen a lot of the `extends` keyword. JavaScript does not have this built in, so how is it implemented? Understanding how extends works will be important for understanding the final bits of our mixin recipe.

Imagine the following classes:

{lang=coffeescript}
~~~~~~~
class Animal

class Otter extends Animal
~~~~~~~

When complied, the above results in the following JS:

{lang=js}
~~~~~~~
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
~~~~~~~

The key bit is:

{lang=js}
~~~~~~~
__extends = function(child, parent) { for (var key in parent) { if (__hasProp.call(parent, key)) child[key] = parent[key]; } function ctor() { this.constructor = child; } ctor.prototype = parent.prototype; child.prototype = new ctor(); child.__super__ = parent.prototype; return child; };
~~~~~~~

This is very very similar to the extend method we constructed earlier, just a bit more verbose and a lot more JS-y. It does the following;

Takes the properties of the parent class and sets them on the child class:

{lang=js}
~~~~~~~
for (var key in parent) { if (__hasProp.call(parent, key)) child[key] = parent[key]; }
~~~~~~~

Then it sets the child's prototype to the parent's prototype:

{lang=js}
~~~~~~~ 
ctor.prototype = parent.prototype; child.prototype = new ctor(); 
~~~~~~~ 

Finally, it sets the super of the child to the parent's prototype

{lang=js}
~~~~~~~ 
child.__super__ = parent.prototype;
~~~~~~~

### Scopes. Say hello to the IIFE

Before we move on we need to understand a bit about scope. You probably have seen a lot of code like: 

{lang=js}
~~~~~~~ 
(function() {

}).call(this);
~~~~~~~

This is a _Immediately-invoked function expression_ or _IIFE_. An IIFE is typically used to maintain scope and make sure `this` points to the correct object. To understand why this is necessary we need to understand a bit more about JavaScript's idea of _this_. _this_ in JavaScript is quite a bit different than the _self_ or _this_ you might be used to from other languages.

What _this_ refers to in JS can change depending on the context in which the function is executed. The best way to see how is through some examples.

Let's say we have a function that returns a value:

{lang=coffeescript}
~~~~~~~
v = 1
getValue: () ->
  return v

v = 2

it 'should return 2', ->
  getValue().should equal 2
~~~~~~~

I> The source for the above spec is available at [https://github.com/k2052/BuildingCoffeeScriptFrameworks.src/blob/chapter-4/spec/scopes-getValue1_spec.coffee](https://github.com/k2052/BuildingCoffeeScriptFrameworks.src/blob/chapter-4/spec/scopes-getValue1_spec.coffee)

Now let's say we wanted to preserve the value of v even if we set it. (Even though it's redundant in this case it can 
be extremely useful)

{lang=coffeescript}
~~~~~~~
v = 1
getValue = do(v) ->
  ->  
    v

v = 2

it 'should return 1', ->
  getValue().should equal 1
~~~~~~~

I> The source for the above spec is available at [https://github.com/k2052/BuildingCoffeeScriptFrameworks.src/blob/chapter-4/spec/scopes-getValue2_spec.coffee](https://github.com/k2052/BuildingCoffeeScriptFrameworks.src/blob/chapter-4/spec/scopes-getValue2_spec.coffee)

The resulting JS for getValue is:

{lang=js}
~~~~~~~
getValue = (function(v) {
  return function() {
    return v;
  };
})(v);
~~~~~~~

The do keyword is a special syntax that allows us to construct closures without confusing the CoffeeScript parser. The following code looks like it might work but the resulting JS would move the closure out of place:

{lang=coffeescript}
~~~~~~~
getValue = (() ->
  -> 
    return v
)(v)
~~~~~~~

Would result in the following JS:

{lang=js}
~~~~~~~
getValue = (function(v) {
  return function() {
    return v;
  };
}, v); // WAT?
~~~~~~~

I apologize for that brief but necessary detour. Now, where were we? Oh yes, `v` has just been scoped correctly by passing it into an _IIFE_. Where this (no pun intended) gets interesting is when we start modifying and controlling _this_. Let's take a look at a counter example -- no, not that type of counter, a counting counter.

{lang=coffeescript}
~~~~~~~
counter = {
  val: 0

  increment: () ->
    this.val += 1
}

it "should increment the counter", ->
  counter.val.should.equal 0
  counter.increment()
  counter.val.should.equal 1
~~~~~~~

We can see how this works by changing the execution context of `increment()`. We can do this by simply pointing another function to it:

{lang=coffeescript}
~~~~~~~
it "should fail to increment", ->
  inc = counter.increment
  inc()
  counter.val.should.equal 0
~~~~~~~

The pointer to increment (_inc_) changes the scope of _this_ inside of increment to the global object.
Thus, when increment tries to increment _this.val_ it encounters _undefined_. We can modify the context of inc by using call:

{lang=coffeescript}
~~~~~~~
it "should increment val on the global scope", ->
  this.val = 2

  inc = counter.increment
  inc.call(this)

  this.val.should.eq 3
  counter.increment()
  counter.val.should eq 1
~~~~~~~

I> The source for the above specs is available at [https://github.com/k2052/BuildingCoffeeScriptFrameworks.src/blob/chapter-4/spec/scopes-counterIncrement_spec.coffee](https://github.com/k2052/BuildingCoffeeScriptFrameworks.src/blob/chapter-4/spec/scopes-counterIncrement_spec.coffee)

Another way to alter the scope of an object is through the use of the _new_ operator; _new_ returns an object with all the properties, functions, values etc scoped to that object.

{lang=coffeescript}
~~~~~~~
Cat = (name) ->
  this.name = name

cat = new Cat("Doug")

it 'should return the name', ->
  cat.name.should.equal "Doug"
~~~~~~~

Now that we have laid the groundwork it's time to build a more robust module class.

### A Powerful Module Class

First, let's define some methods which we will exclude from including or extending:

{lang=coffeescript}
~~~~~~~
moduleKeywords = ['included', 'extended']
~~~~~~~

Then define our Module class:

{lang=coffeescript}
~~~~~~~
class Module
~~~~~~~

Define an include method (for instance properties) that takes an obj (remember classes are just objects in JS):

{lang=coffeescript}
~~~~~~~
  @include: (obj) ->
~~~~~~~

Throw an error if we don't get passed an object:

{lang=coffeescript}
~~~~~~~
    throw new Error('include(obj) requires obj') unless obj
~~~~~~~

Loop through the properties of the object and include them on this class' prototype (instance):

{lang=coffeescript}
~~~~~~~
    for key, value of obj when key not in moduleKeywords
      @::[key] = value
~~~~~~~

Now we will use apply to call the _included_ callback in the context of this class but only if the method exists:

{lang=coffeescript}
~~~~~~~
    obj.included?.apply(this)
~~~~~~~

T> The _included_ callback allows a base class to be alerted when it has been included.

Finally, we will return _this_ so the method is chainable:

{lang=coffeescript}
~~~~~~~
    @
~~~~~~~

Now we will define an extend method (for class properties) that takes an obj (remember classes are just objects in JS) and extends the class with properties from that obj:

{lang=coffeescript}
~~~~~~~
  @extend: (obj) ->
~~~~~~~

We will need to throw an error if we don't get passed an object:

{lang=coffeescript}
~~~~~~~
    throw new Error('extend(obj) requires obj') unless obj
~~~~~~~

Loop through the properties of the object and include them in the class:

{lang=coffeescript}
~~~~~~~
    for key, value of obj when key not in moduleKeywords
      @[key] = value
~~~~~~~

Now we will apply an extended callback just like we did with _included_:

{lang=coffeescript}
~~~~~~~
    obj.extended?.apply(this)
~~~~~~~

And we will return _this_ so the method is chainable:

{lang=coffeescript}
~~~~~~~
    @
~~~~~~~

Our final Module class looks like this:

{lang=coffeescript}
~~~~~~~
moduleKeywords = ['included', 'extended']
class Module
  @include: (obj) ->
    throw new Error('include(obj) requires obj') unless obj

    for key, value of obj when key not in moduleKeywords
      @::[key] = value

    obj.included?.apply(this)
    @

  @extend: (obj) ->
    throw new Error('extend(obj) requires obj') unless obj

    for key, value of obj when key not in moduleKeywords
      @[key] = value

    obj.extended?.apply(this)
    
    @
~~~~~~~

And we can use it like this:

{lang=coffeescript}
~~~~~~~
class Animal
  talk: () ->
    return "growl!"

  @isInsect: () ->
    return false

class Cat extends Module
  @include Animal
  @extend Animal

it 'should not be an insect', ->
  Cat.isInsect().should.equal false

it 'should be able to talk', ->
  cat = new Cat
  cat.talk().should.equal "growl!"
~~~~~~~

I> The source for the above spec is available at [https://github.com/k2052/BuildingCoffeeScriptFrameworks.src/blob/chapter-4/spec/module_spec.coffee](https://github.com/k2052/BuildingCoffeeScriptFrameworks.src/blob/chapter-4/spec/module_spec.coffee)

### Namespacing

The prototypal nature of JavaScript makes namespacing things dirt simple. 
To place our `Module` class inside of a scope we can just do:

{lang=coffeescript}
~~~~~~~
Ryggard = {}
Ryggard.Module = Module
~~~~~~~

## Events

Any CoffeeScript framework worth its weight needs a system for handling events. Something that looks like this:

{lang=coffeescript}
~~~~~~~
Animal.bind 'animals:talking', => 
  alert('Animals be talking')

Animal.trigger 'animals:talking'
~~~~~~~

It might look complicated but it's actually really straightforward. The callbacky nature of CoffeeScript/JavaScript makes it trivial to implement. At their core, events are simply a Pub/Sub pattern. 

### Pub/Sub

A publish-subscribe pattern can be implemented using a hash of event names mapped to callback methods. Then we only need to make sure we publish through the methods on this class; so the class handles figuring out the callbacks to trigger for an event.

First we need to define a PubSub class:

{lang=coffeescript}
~~~~~~~
class PubSub
  constructor: ->
    @channels = {}
~~~~~~~

This has a channels object to hold the events and their callbacks. Now we need a way to subscribe to events:

{lang=coffeescript}
~~~~~~~
  subscribe: (name, callback, context=undefined) ->
    context or= @
    @channels[name] = [] unless @channels[name]?
    @channels[name].push context: context, callback: callback
    @
~~~~~~~

Our subscribe method takes an event name, a callback, and optional context to execute the callback in.
To publish we simple need to call the callback for the event name:

{lang=coffeescript}
~~~~~~~
  publish: (name, data...) ->
    for sub in @channels[name]
      sub.callback.apply(sub.context, data)
~~~~~~~

Now what if we wanted to unsubscribe? Simple, we just need to remove the channel from _@channels_:

{lang=coffeescript}
~~~~~~~
  unsubscribe: (name, callback) ->
    for sub, i in @channels[name]
      @channels[name].splice(i, 1) if sub.callback == callback
~~~~~~~

To utilize this is is easy:

{lang=coffeescript}
~~~~~~~
describe 'PubSub', ->
  data     = null
  spy      = null
  pubSub   = null
   
  beforeEach ->
    data   = 'cats'
    pubSub = new pubSub
    spy    = sinon.spy() 

  it 'subscribes and publishes', ->
    pubSub.subscribe 'channel', spy
    pubSub.publish   'channel', data
    expect(spy).toHaveBeenCalledWith(data)
   
  it 'handles unsubscribe', ->
    pubSub.subscribe   'channel', spy
    pubSub.unsubscribe 'channel', spy
    pubSub.publish     'channel', data
    expect(spy).not.toHaveBeenCalled()
~~~~~~~

I> The source for the above spec is available at [https://github.com/k2052/BuildingCoffeeScriptFrameworks.src/blob/chapter-4/spec/pubSub_spec.coffee](https://github.com/k2052/BuildingCoffeeScriptFrameworks.src/blob/chapter-4/spec/pubSub_spec.coffee)

### Implementing our Events Handler

Now that we know PubSub let's build a more full fledged events system. First, we need to define an events object to 
hold our Events functionality:

{lang=coffeescript}
~~~~~~~
Events =
~~~~~~~

Now we will need a method that takes an event name as a string and callback to call when that event is triggered:

{lang=coffeescript}
~~~~~~~
  bind: (eventOrEvents, callback) ->
    # Takes the event string and split it into an array of events. 
    # Allows multiple events to passed using spaces as a delimiter 
    events = eventOrEvents.split(' ')

    # Pulls the callbacks from the current class if they exist,
    # if not then default to an empty object
    calls = @hasOwnProperty('_callbacks') and @_callbacks or= {}

    # Loops over the events and sets the callbacks
    for name in events
      calls[name] or= []
      calls[name].push(callback)

    # Returns this to support chaining
    @
~~~~~~~

Now we need a method to trigger events and call the callbacks:

{lang=coffeescript}
~~~~~~~
  trigger: (args...) ->
    # Pull the event name from the first argument
    event = args.shift()

    # Copy the callbacks for this event into a list array. 
    # Return if there are no callbacks for this event
    list = @hasOwnProperty('_callbacks') and @_callbacks?[event]
    return unless list

    # Loop through each of the callbacks for this event 
    # and then call them with apply using the current class as the context
    for callback in list
      if callback.apply(@, args) is false
        break
    true
~~~~~~~

And finally, we will need a method to unbind any an event and its associated callback:

{lang=coffeescript}
~~~~~~~
  unbind: (event, callback) ->
    # If no event name is passed then unbind everything
    unless event
      @_callbacks = {}
      return @

    #  Pull a list of all the callbacks for this event. 
    # Return if there are no callbacks
    list = @_callbacks?[event]
    return @ unless list

    #  If no specific callback is passed then unbind all the callbacks for the 
    #  event and return this
    unless callback
      delete @_callbacks[event]
      return @

    # Loop through the callbacks and when the callback to unbind is found,
    # splice it out.
    for cb, i in list when cb is callback
      list = list.slice()
      list.splice(i, 1)
      @_callbacks[event] = list
      break

    # Return this to support chaining
    @
~~~~~~~

The final events object looks like this:

{lang=coffeescript}
~~~~~~~
Events =
  bind: (eventOrEvents, callback) ->
    # Takes the event string and split it into an array of events. 
    # Allows multiple events to passed using spaces as a delimiter 
    events = eventOrEvents.split(' ')

    # Pulls the callbacks from the current class if they exist, 
    # if not then default to an empty object
    calls = @hasOwnProperty('_callbacks') and @_callbacks or= {}

    # Loops over the events and sets the callbacks
    for name in events
      calls[name] or= []
      calls[name].push(callback)

    # Returns this to support chaining
    @

  trigger: (args...) ->
    # Pull the event name from the first argument
    event = args.shift()

    # Copy the callbacks for this event into a list array. 
    # Return if there are no callbacks for this event
    list = @hasOwnProperty('_callbacks') and @_callbacks?[event]
    return unless list

    # Loop through each of the callbacks for this event 
    # and then call them with apply using the current class as the context
    for callback in list
      if callback.apply(@, args) is false
        break
    true

  unbind: (event, callback) ->
    # If no event name is passed then unbind everything
    unless event
      @_callbacks = {}
      return @

    # Pull a list of all the callbacks for this event.
    # Return if there are no callbacks
    list = @_callbacks?[event]
    return @ unless list

    # If no specific callback is passed then unbind all the callbacks 
    # for the event and return this
    unless callback
      delete @_callbacks[event]
      return @

    # Loop through the callbacks and when the callback to unbind is found, 
    # splice it out.
    for cb, i in list when cb is callback
      list = list.slice()
      list.splice(i, 1)
      @_callbacks[event] = list
      break

    # Return this to support chaining
    @
~~~~~~~

Now we can utilize events like:

{lang=coffeescript}
~~~~~~~
class Animal extends Events

it "can bind/trigger events", ->
  Animal.bind 'animals:talking', spy
  Animal.trigger 'animals:talking'
  expect(spy).to.have.been.called
~~~~~~~

I> The source for the above spec is available at [https://github.com/k2052/BuildingCoffeeScriptFrameworks.src/blob/chapter-4/spec/events_spec.coffee](https://github.com/k2052/BuildingCoffeeScriptFrameworks.src/blob/chapter-4/spec/events_spec.coffee)

## Routing

Routing at its core is a lot simpler than you might think it is. It essentially consists of three pieces;

1. A matcher that matches routes to the current route/url.
2. A storage mechanism for the routes. Usually just an array Regular expression strings mapped to callback functions.
3. Callback functions that are called for each route

A simple implementation of a Router would look like this:

{lang=coffeescript}
~~~~~~~
class Router
  # An array to hold our route objects
  @routes = []
~~~~~~~

A Regex parser that takes strings like _/blog/:year/:month/:day_ and routes them to a function.
The splats (:year :month etc) are turned into params that are then passed in an hash to the function:

{lang=coffeescript}
~~~~~~~
  @buildRegexString: (path) ->
    path.replace(/\//g, "\\/").replace(/:(\w*)/g,"(\\w*)")
~~~~~~~

A function to add a route to routes:   

{lang=coffeescript}
~~~~~~~    
  @add: (path, func) ->
    # A route has;
    # 1. params in the form of splats like :year/:month/:day. 
    #    These are extracted and passed to the callback function as arguments
    # 2. A regular expression to match the current path agains
    # 3. A callback function
    @routes.push {
      params: path.match(/:(\w*)/g)
      regex: new RegExp(@buildRegexString(path))
      callback: func
    }
~~~~~~~ 

We need to find the route that matches the current route (i.e the URL). We can use the captures from the regex to generate the named params, then call the callback function with the params:

{lang=coffeescript}
~~~~~~~    
  # Take a url to process. Default to the current path
  @process: (url = window.location.pathname) ->
    # Loop through the routes
    for route in @routes
      # math the current url against the route's regex
      results = url.match(route.regex)
      
      # Do we have a match?
      if results?
        index = 1
        namedParams = {}

        # Does this route have params defined? if so split them up and
        # push the results onto namedParams
        if route.params?
          for name in route.params
            namedParams[name.slice(1)] = results[index++]

        route.callback(namedParams)
~~~~~~~ 

To utilize this is dead simple:

{lang=coffeescript}
~~~~~~~
Router.add "/blog/:year/:month/:day", (params) ->
  # code for the route goes here
~~~~~~~

Add when the DOM is ready we route the routes by calling Router.process

{lang=coffeescript}
~~~~~~~
$(document).on "ready", ->
  Router.process()
~~~~~~~

An example of how this might be used:

{lang=coffeescript}
~~~~~~~      
describe "Router#process", ->
    api = result = ""
    beforeEach ->
      api = {
        testCallBack: (params)->
          console.log "testCallBack"
      }
      spyOn(api, "testCallBack").andCallFake( (params)->
        result = params
      )
      
      Router.add blog_route, api.testCallBack
      Router.process("/blog/2012/01/01")
    
    it "should find the route and execute the callback", ->
      expect(api.testCallBack).toHaveBeenCalled()
      
    it "should execute the callback with the correct parameters", ->
      expect(result).toEqual({ year: "2012", month: "01", day: "01"})
~~~~~~~

I> The source for the above can be found at [https://github.com/k2052/BuildingCoffeeScriptFrameworks.src/blob/chapter-4/spec/router_spec.coffee](https://github.com/k2052/BuildingCoffeeScriptFrameworks.src/blob/chapter-4/spec/router_spec.coffee)

At this point, our router is missing one crucial thing, a concept of state. 
It would be nice for the router to be aware of the current path and be able to manage current/past paths.
A full implementation is beyond the scope of this chapter but we will get to it when we build our framework's final Router class.

Our full fledged Router class will make use of the browser's history to hold path state. We will utilize the history's events for listening to path changes. It will look this:

{lang=coffeescript}
~~~~~~~
$(window).on 'popstate' ->
  # Match the new path
~~~~~~~

But let's not get ahead of ourselves. First we have some data binding to dive into.

## Data Binding

Data binders appear to be quite complex but this is mostly due to all the convenience features they come with. At their core, data binders are nothing more than some decoration around a PubSub pattern.

A data-binding system can be boiled down to three elements

1. A way of specifying which UI elements are bound to which properties
2. A way to monitor changes both on the UI and on the properties
3. A way to trigger/alert both the UI and the object of any changes

All of these features come with any of the DOM libraries like jQuery. In fact, all we really need to hook up a basic data binder class is to ask jQuery when things are changing and tell it what callbacks to hit when they do.

A simple data binder could be implemented like this:

{lang=coffeescript}
~~~~~~~
class DataBinder
  
  # Take an objectid so the element can be uniquely identified in the DOM.
  constructor: (objectid) ->
    # Use jQuery for the PubSub. But we could simply use the Events class 
    # we constructed earlier.
    @pubSub   = $ {}

    # The attribute to bind to
    attribute = "data-bind-#{objectid}"

    # An Event name
    message = "#{objectid}:change"
    
    # Listen for changes to the element
    $(document).on "change", "[#{attribute}]", (event) =>   
      $elem = $(event.target)
      @pubSub.trigger message, [$elem.attr(attribute), $elem.val()]
    
    # Listen for events
    @pubSub.on message, (event, property, new_val) ->
      for elem in $("[#{attribute}=#{property}]")
        $elem = $(elem)

        if $elem.is("input, textarea, select")
          $elem.val new_val
        else
          $elm.html new_val
~~~~~~~

And we can utilize it like so:

{lang=coffeescript}
~~~~~~~
# We will need some HTML for this too work
# HTML <input type="text" data-bind-cat-1="name" />
class Cat
  binder: null
  pubSub: null
  message: ''
   
  # Every time we set an attribute on cat we need to trigger 
  # the change event on the DataBinder instance
  set: (attr, val) ->
    @[attr] = val
    @pubSub.trigger @message, [attr, val]

  get: (attr) -> 
    @[attr]

  constructor: (id) ->
    @binder  = new DataBinder(id)
    @pubSub  = @binder.pubSub
    @message = "#{id}:change"
     
    @pubSub.on @message, (event, attr, value) =>
      @[attr] = value

cat = new Cat("cat-1")
cat.set("name", "Wiswell")
~~~~~~~

It's that simple!

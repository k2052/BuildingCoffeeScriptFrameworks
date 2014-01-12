--- 
title: Cloning A Prototype: A Brief Introduction To Prototypes In CoffeeScript
---

Say we have the following object:

    obj = {
      foo: 'bar'
    }
{: .language-javascript}

Then we create a reference to it:

    newObj = obj
{: .language-coffeescript}

Then we change the value of foo:

    newObj.foo = 'cats'
{: .language-coffeescript}

What then is the value of `obj.foo`? Is it `foo` or `cats`? It's `cats`. 
When we created `newObj` it was just a reference; so any modifications made to it will be reflected in obj.
Let's run a test just to be certain.

    it "should change the value on the original object", ->
      obj =
        foo: 'bar'

      newObj = obj 
      newObj.foo = 'cats'

      obj.foo.should.equal 'cats'
{: .language-coffeescript executable}

Cloning an object's prototype and sharing properties across the clones is referred to by the convoluted name "Differential Inheritance"

JS objects can delegate properties to their prototype and the protoype of that can do the same, _it's prototypes all the way up_ to Object.prototype.

When you read a property from an object it first tries read from the property object but if that property exists it checks the prototype of that object and on and on until it reaches the prototype of _Object_

This means that objects can share properties by sharing prototypes. If you define update methods on the prototype and only use those methods to update the objects, then you can update all the clones by just updating one.

A> If you have ever been confused by clone vs deep clone the above should make it clear. A deep cloned item is an 
A> object with a unique prototype so it prevents the recursive checking by the system.

Let's implement a basic cloning method:

    Object.prototype.clone = ->
      clone = Object.create(this)

      return clone
{: .language-coffeescript}

This utilizes `Object.create` which is a relatively new feature to JS. It creates a new object using the specified prototype. Internally it looks like this:

    Object.create = (o) ->
      Func = ->
      Func.prototype = o
      new Func()
{: .language-coffeescript}

Now we can utilize our cloning method like this:

    it "should clone a dog and allow us to change the legs count on all dogs", ->
      Dog = 
        name: "" # Not shared
        prototype.legs = 4

      dog1 = new Dog()
      dog1.name = "Jerry"
      dog2 = dog1.clone()
      dog2.name.should.equal undefined
      dog2.legs.should.equal 4
      
      dog1.legs = 8
      dog2.legs.should.equal 8

      dog2.legs = 4
      dog1.legs.should.equal 4
{: .language-coffeescript executable}

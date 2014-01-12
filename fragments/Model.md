The absolute basic functionality of any Model is; Create, Read, Update and Delete or CRUD. 
We can implement most other functionality using events. Syncing, monitoring, updating controllers, binding to views etc all use events to hook in the functionality. Utilizing events even allows us to keep our base Model class thin and we can use event binding to add in commonly needed functionality like AJAX syncing.


Declare a class with support for events

    class Model extends Module
      @extend Events

We will need two containers for the records; 

Holds all the records 

      @records: {}

A an array of attributes for the model e.g @attributes('first_name', 'last_name')
     
      @attributes: []

The attributes of the model.

      @fields: (attributes...) ->

Delete any exisiting records
        @records    = {}

Create an array for the attributes. Supports variable arguments

        @attributes = attributes if attributes.length
        @attributes or= []

Unbind any stray events

        @unbind()
        @ 

Creates a new record. Will take the attributes and pass them into the Model constructor. Then call save on that record

      @create: (atts, options) ->
        record = new @(atts)
        record.save(options)

Adds a record gets called by save on an instance

      @read: (id) ->
        record = @records[id] 
        throw new Error("\"#{@className}\" model could not find a record for the ID \"#{id}\"") unless record
        return record

Finds a record by ID and updates it

      @update: (id, atts, options) ->
        @read(id).update(atts, options)

Find the record by ID and destroy it

      @destroy: (id, options) ->
        @read(id).destroy(options)

GeneratesÏÏÏ unique ID for a record

      @uid: ->
        gUID()

Creates a new Model record

      constructor: (atts) ->
        super
        @load atts if atts
        @uid = @constructor.uid()

      isNew: -> not @exists()

      exists: -> @uid && @uid of @constructor.records

Loads the attributes onto the model instance. Loops through the keys and values and sets the relevant properties on the instance

      load: (atts) ->
        for key, value of atts
          if typeof @[key] is 'function'
            @[key](value)
          else
            @[key] = value
        @

Save this model

      save: (options = {}) ->
        @trigger('beforeSave', options)
        record = if @isNew() then @create(options) else @update(options)
        @trigger('save', options)
        record

Creates the record and then  adds it to @constructor.records

      create: (options) ->
        @trigger('beforeCreate')

        record = new @constructor(@attributes())
        record.uid = @uid
        @constructor.records[@uid] = record

        @trigger('create')
        @trigger('change', 'create')
        record

This is really the true update method. The save method determines whether or not an object is new or just needs updating
Save then triggers the update and create callbacks.

    updateAttributes: (attributes) ->
      @load(attributes)
      @save()

Updates the record

      update: (options) ->
        @trigger('beforeUpdate')

        records = @constructor.records
        records[@uid].load @attributes()

        @trigger('update')
        @trigger('change', 'update')
        records[@uid].clone()

Removes the record from @constructor.records (the collection)

      destroy: (options = {}) ->
        @trigger('beforeDestroy')
        delete @constructor.records[@uid]
        @trigger('destroy')
        @trigger('change', 'destroy')
        @unbind()
        @
      
      unbind: -> @trigger('unbind')

      trigger: (args...) ->
        args.splice(1, 0, @)
        @constructor.trigger(args...)


Utilzing this class is relatively straightforward.

But it breaks down when the we start cloning copies of the instannces. 

    it "clones are dynamic", ->
      asset = Asset.create name: "hotel california"
      clone = Asset.find(asset.uid)
      asset.name = "checkout anytime"
      asset.save()
      expect(clone.name).toEqual "checkout anytime"


This feature is critical if we are going to be passing around instances of our models and need them to stay in sync. It will be near impossible to do any sort of views binding without them.

The simple solution is to make sure we always return a cloned record and make sure the clones always stay up to date. Why clones you may ask? Well the answer lies in how Javacript's prototypal model works. But I'm getting ahead of myself a bit, first we need to know a little about references. Let's start with a very basic example of how JS treats references to objects.

Say we have the following object:

    obj = {
      foo: 'bar'
    }

And we create a reference to it:

    newObj = obj

Then we change the value of foo:

    newObj.foo = 'cats'

What then is the value of `obj.foo`? Is it foo or cats? It's cats. When we created newObj it was just a reference; any modifciations made to it will be reflected in obj. Pretty cool huh?

    it "should change the value on the original object", ->
      obj =
        foo: 'bar'

      newObj = obj 
      newObj.foo = 'cats'

      obj.foo.should.equal 'cats'

Now that we got the aboslute basics under our belt we can move on to cloning prototypes. If we remember from previous chapters, class instances are just prototypes, so cloning them amounts to cloning an objects prototype.

Cloning an object's prototype and sharing properties across the clones is referred to by the heavily convulted name of "Differential Inheritance"

JS objects can delegate properties to their prototype and the protoype of that can do the same, it's prototypes all the way up to Object.prototype.

When you read a property from an object it first tries read from the property object but if that property exist it checks the prototype of that object and on and on until it reaches the prototype of the Object.

This means that objects can share properties by sharing prototypes. If you define update methods on the prototype and only use those methods to update the objects, then you can update all the clones by just updating one.

Note:
  If you have ever been confused by clone vs deep clone the above should make it clear. A deep cloned item is an object with a unique prototype so it prevents the recursive checking by the system.

Let's see a basic example of this

First define a function for cloning a object

    Object.prototype.clone = ->
      clone = Object.create(this)

      return clone

This utlizes Object.create which is a relatively new feature to JS. It creates a new object using the specificed prototype. Internally it looks like this:

    Object.create = (o) ->
      Func = ->
      Func.prototype = o
      new Func()

Let's define an object to clone:


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

Nifty isn't it. To make our previous test for dynamic record passe we just need to change it so all the methods that return a record return a clone.

FINISHED Model.litcoffee here

Now we can utlize it like.

And our dynamic records now work.

Now onto those controllers.











---
title: "MVC Patterns In CoffeeScript Part One: Models"
---

Models can seem complicated but at their core they're usually nothing more than some ordered hashes.
But then again, pretty much all abstractions are at their core just primitive data types.
Most of what we expect from a model besides CRUD can implemented using event callbacks. We don't
need to bloat our models with t Ajax persistence, syncing, binding to views etc. We can accomplished that all using events. For now, let's focus on implementing the CRUD.

Let's declared a model class:

    class Model extends Module
      @extend Events

      # Holds all the records 
      @records: {}
     
      # An array of attributes for the model e.g @attributes('first_name', 'last_name')
      @attributes: []
      
      # We define our fields through this method. e.g 
      # class Human extends Model
      #   @fields 'name'
      @fields: (attributes...) ->
         
        # Delete any stray records
        @records    = {}

        # Create an array for the attributes. Supports variable arguments
        @attributes = attributes if attributes.length
        @attributes or= []
        
        # Unbind any stray events and return this for chaining
        @unbind()
        @ 

      # Creates a new record. Will take the attributes and pass them into the Model constructor. Then call save on that record
      @create: (atts, options) ->
        record = new @(atts)
        record.save(options)
      
      # Reads/finds a record
      @read: (id) ->
        record = @records[id]?.clone()
        throw new Error("\"#{@className}\" model could not find a record for the ID \"#{id}\"") unless record
        return record

      # Finds a record by ID and updates it
      @update: (id, atts, options) ->
        @read(id).update(atts, options)

      # Find the record by ID and destroy it
      @destroy: (id, options) ->
        @read(id).destroy(options)
      
      # Generates a unique ID for this instance
      @uid: ->
        'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace /[xy]/g, (c) ->
          r = Math.random() * 16 | 0
          v = if c is 'x' then r else r & 3 | 8
          v.toString 16
        .toUpperCase()

      constructor: (atts) ->
        super
        @load atts if atts
        @uid = @constructor.uid()

      isNew: -> not @exists()

      exists: -> @uid && @uid of @constructor.records

      # Loads the attributes onto the model instance. Loops through the keys and values and sets the relevant properties on the instance
      load: (atts) ->
        for key, value of atts
          if typeof @[key] is 'function'
            @[key](value)
          else
            @[key] = value
        @

      save: (options = {}) ->
        @trigger('beforeSave', options)
        record = if @isNew() then @create(options) else @update(options)
        @trigger('save', options)
        record

      create: (options) ->
        @trigger('beforeCreate')

        record = new @constructor(@attributes())
        record.uid = @uid
        @constructor.records[@uid] = record

        @trigger('create')
        @trigger('change', 'create')
        record

      # This is really the true update method. The save method determines whether or not an object is new or just needs updating
      # Saves then triggers the update and create callbacks.
      updateAttributes: (attributes) ->
        @load(attributes)
        @save()

      update: (options) ->
        @trigger('beforeUpdate')

        records = @constructor.records
        records[@uid].load @attributes()

        @trigger('update')
        @trigger('change', 'update')
        records[@uid].clone()

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
  {: .language-coffeescript}

Utilizing this class is relatively straightforward.

class Cat extends Model
  @fields "name"

But it breaks down when the we start cloning copies of the instannces. 

    it "clones are dynamic", ->
      asset = Cat.create name: "Doge"
      clone = Cat.read(asset.uid)
      asset.name = "Jerry"
      asset.save()
      expect(clone.name).toEqual "Jerry"

This feature is critical if we are going to be passing around instances of our models and need them to stay in sync. It will be near impossible to do any sort of views binding without them.nce 

The simple solution is to make sure we always return a cloned record and make sure the clones always stay up to date. Why clones? Because they allow us to take advantage the very useful _Differential Inheritance_ nature of JS.
If we start with a clone and then every subsequent clone will look to the original object instance for it's attributes.
Provided of course, we never mess with the prototype of the clone.

Once we modify @read to return a clone everything works as intended
  @read: (id) ->
    record = @records[id]?.clone()
    throw new Error("\"#{@className}\" model could not find a record for the ID \"#{id}\"") unless record
    return record


    it "clones are dynamic", ->
      asset = Cat.create name: "Doge"
      clone = Cat.read(asset.uid)
      asset.name = "Jerry"
      asset.save()
      expect(clone.name).toEqual "Jerry"


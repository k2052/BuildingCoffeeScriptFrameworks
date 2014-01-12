---
title: Building an Events System In CoffeeScript
---

Any CoffeeScript framework worth its weight needs a system for handling events. Something that looks like this:

    Animal.bind 'animals:talking', => 
      alert('Animals be talking')

    Animal.trigger 'animals:talking'
{: .language-coffeescript}

It might look quite complicated, but it's actually really straightforward. The callbacky nature of CS/JS makes it really simple.

First, we need to define events object to hold our Events functionality. This will be able to mixin Events into any class

    Events =

A bind method that takes the event name and callback to bind it to:

    bind: (ev, callback) ->
      # Takes the event string and split it into an array of events. Allows multiple events to passed using spaces as a delimiter 
      evs = ev.split(' ')

      # Pulls the callbacks from the current class if they exist, if not then default to an empty object
      calls = @hasOwnProperty('_callbacks') and @_callbacks or= {}

      # Loops over the evs and sets the callbacks

      for name in evs
        calls[name] or= []
        calls[name].push(callback)

      # Returns this to support chaining
      @
{: .language-coffeescript}

Now we need a method to trigger events and call the callbacks:

    trigger: (args...) ->
      # Pull the event name from the first argument
      ev = args.shift()

      # Copy the callbacks for this event into a list array. Return if there are no callbacks for this event
      list = @hasOwnProperty('_callbacks') and @_callbacks?[ev]
      return unless list

      # Loop through each of the callbacks for this event and then call them with apply using the current class as the context
      for callback in list
        if callback.apply(@, args) is false
          break
      true
{: .language-coffeescript}

Now we need a method to unbind any an event and its callback:

    unbind: (ev, callback) ->
      # If no event name is passed then ubind everything
      unless ev
        @_callbacks = {}
        return @

      #  Pull a list of all the callbacks for this event. Return if there are no callbacks
      list = @_callbacks?[ev]
      return @ unless list


      #  If no specific callback is passed then ubind all the callbacks for the event and return this
      unless callback
        delete @_callbacks[ev]
        return @

      # Loop through the callbacks and when the callback to unbind is found splice it out.
      for cb, i in list when cb is callback
        list = list.slice()
        list.splice(i, 1)
        @_callbacks[ev] = list
        break

      # Return this to support chaining
      @
{: .language-coffeescript}

The final events class look like this:

Now we can utilize events like so:

    class Animal extends Events

    it "can bind/trigger events", ->
      Animal.bind 'animals:talking', spy
      Animal.trigger 'animals:talking'
      expect(spy).to.have.been.called
{: .language-coffeescript executable}

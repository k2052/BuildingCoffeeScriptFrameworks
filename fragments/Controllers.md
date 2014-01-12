Controllers are a bit of a blurry area in JS. What they handle is usually determined more by convention than anything else. For our purposes, we will use them much like we would controllers on the server side, where methods correspond to routes or actions in the application. For example:

A users controller:

    class Users extends Controller
      
      # A method for retrieving a User
      read: () ->
      
      # A method for adding a new user
      create: () -> 

This should be very familiar to anyone that has worked with a server side MVC framework before,

So there are two primary functions the controller class will have to server;

1. Providing event delegation for UI events. This will route events like a click on button "New User" to the relevant action. 
2. Provide routing to route URLs like `/#/users/all` to a relevant action.

The controller class needs to extend Module and provide Events

    class Controller extends Module
      @include Events

Splits the event into an event name and a selector
It will make our API look like this:
{"eventType selector", "functionName"}
Say something like:

events:
  "click .new-user": addUser

      eventSplitter: /^(\S+)\s*(.*)$/

The default tag to associate with this controller

      tag: 'div'

      constructor: (options) ->

Set any options on a properties on the class

        for key, value of @options
          @[key] = value

Create a dom el and the jQueryify it

        @el  = document.createElement(@tag) unless @el
        @el  = $(@el)
        @$el = @el

Add a class and atttributes if there are any

        @el.addClass(@className) if @className
        @el.attr(@attributes) if @attributes

Add events and elemenents from the class vars (remember @constructor points to the class)

        @events = @constructor.events unless @events
        @elements = @constructor.elements unless @elements

Delegate the events and refresh the dom elements

        do @delegateEvents if @events
        do @refreshElements if @elements
        super

Delegates events using jQuery's delegate. If you havre done some dynamic binding of events using $.live() 
then delegate is a lot like it. With live, the event delegation is added to the document and jQuery makes
use of event propogation to catch events for the element. This has some obivous perfomance drawbacks, jQuery has to 
wait until the event bubbles all the way up to the document, which can take a while; and only then does it trigger the event
. delegate binds the event to a specific element so jQuery only has to wait for the event to bubble up the element of
our controller; which we have already created. 

Binding live is of course neccessary because our controllers and views will be rendering fully after the events have been
binded. 

      delegateEvents: ->
        for key, method of @events
          method = @proxy(@[method]) unless typeof(method) is 'function'

          match      = key.match(@eventSplitter)
          eventName  = match[1]
          selector   = match[2]

          if selector is ''
            @el.bind(eventName, method)
          else
            @el.delegate(selector, eventName, method)

      refreshElements: ->
         @[value] = @el.find(key) for key, value of @elements

      destroy: =>
        @trigger 'release'
        @el.remove()
        @unbind()



    it "can add events", ->
      Tasks.include
        events:
          click: "wasClicked"

        wasClicked: $.proxy spy

      tasks = new Tasks el: element
      element.click()
      expect(spy).to.have.been.called


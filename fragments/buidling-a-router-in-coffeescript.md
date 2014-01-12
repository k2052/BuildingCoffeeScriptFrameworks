---
title: Building a Router In CoffeeScript
---

Routing at its core is a lot simpler than you might think it is. It essentially consists of three pieces;

1. A matcher that matches routes to the current route/url.
2. A storage mechanism for the routes. Usually just an array of Regular expression strings mapped to a lambda.
3. Callback functions that are called for each route

A simple implementation of a Router would look like this:

The class

    class Router

An array to hold the routes
      @routes = []

A Regex parser that takes strings like _/blog/:year/:month/:day_ and routes them to a function.
The splats (:year :month etc) are turned into params that are then passed in an hash to the function:

    @buildRegexString: (path) ->
      path.replace(/\//g, "\\/").replace(/:(\w*)/g,"(\\w*)")

A function to add a route to routes:   
    
      @add: (path, func) ->
        @routes.push {
          params: path.match(/:(\w*)/g)
          regex: new RegExp(@buildRegexString(path))
          callback: func
        }

We need to find relevant route that matches the current route (i.e the URL). We can use the captures from the regex to generate the named params, then call the callback function with the params:

    @process: (url = window.location.pathname) ->
      for route in @routes
        results = url.match(route.regex)
        if results?
          index = 1
          namedParams = {}
          if route.params?
            for name in route.params
              namedParams[name.slice(1)] = results[index++]
          route.callback(namedParams)
          return

To utilize this is dead simple:

    Router.add "/blog/:year/:month/:day", (params) ->
      # code for the route goes here

Add when the DOM is ready we route the routes by calling Router.process

    $(document).on "ready", ->
      Router.process()

An example of how this might be used:
      
    describe "#process", ->
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
 
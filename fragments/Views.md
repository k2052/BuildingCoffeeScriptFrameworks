In the JS world there are a lot of different ways of doing views.

You can use a plain string (pretty easy if one is using coffeescript):

    "<div>#{user.name}</div>"

You can use custom template engines that are a lot like server side templating:

    <% if @projects.length: %>
      <% for project in @projects: %>
        <a href="<%= project.url %>"><%= project.name %></a>
        <p><%= project.description %></p>
      <% end %>
    <% else: %>
      No projects
    <% end %>

You can use simple logi-less substitution engines like Mustache:

    {{#items}}
      {{#first}}
        <li><strong>{{name}}</strong></li>
      {{/first}}
      {{#link}}
        <li><a href="{{url}}">{{name}}</a></li>
      {{/link}}
    {{/items}}

Or you can combine logic-less templates with a view prepocessor using something like Handlebars.js:

    <div class="post">
      <h1>By {{fullName author}}</h1>
      <div class="body">{{body}}</div>

      <h1>Comments</h1>

      {{#each comments}}
      <h2>By {{fullName author}}</h2>
      <div class="body">{{body}}</div>
      {{/each}}
    </div>

    var context = {
      author: {firstName: "Alan", lastName: "Johnson"},
      body: "I Love Handlebars",
      comments: [{
        author: {firstName: "Yehuda", lastName: "Katz"},
        body: "Me too!"
      }]
    };

    Handlebars.registerHelper('fullName', function(person) {
      return person.firstName + " " + person.lastName;
    });


Most templating solutions treat views as dumb, in eras past this made a lot of sense, views were dumb, once rendered they often made little if any changes. But our web apps are no longer static, they undergo increasingly dynamic and complex changes based on user interaction. It's strange then that our templating solutions continue to treate markup like server side templating solutions do. 

The render and forget model starts to break down with more complex apps. Our web applications are starting to act and feel like their desktop counterparts, so our templating solutions need to start treating views like desktop frameworks.

In Cocoa a view is not dumb. It is considered where UI code is handled, it's not just a container for Markup. How do we implement cocoa like views in the browser?

There are a few projects out there and most if not all can fall into two categories. Component based and custom view language based.

Component based solutions compartmentalize a view into a component consisting of logic, styling and markup. Frameworks like Angular.js and Ember.js fall into this category. They look like this:

    App.GravatarImageComponent = Ember.Component.extend({
      size: 200,
      email: '',

      gravatarUrl: function() {
        var email = this.get('email'),
            size = this.get('size');

        return 'http://www.gravatar.com/avatar/' + hex_md5(email) + '?s=' + size;
      }.property('email', 'size')
    });

    <ul class="example-gravatar">
      <li>{{gravatar-image email="tomster@emberjs.com" size="200"}}</li>
      <li>{{gravatar-image size="200"}}</li>
    </ul>

    <img {{bind-attr src=gravatarUrl}}>
    <div class="email-input">
      {{input type="email" value=email placeholder="Enter your Gravatar e-mail"}}
    </div>

The other category is usually accomplished with 1. An entirely different language specifically for templating 2. Or by utlizing a framework on top of a DSL friendly language that compiles to javascript.

An example of the former is Seranade.js

    ul#comments
      - collection @comments
        li @title 

We will be focusing on the latter and coffeescript. Our implementation will be heavily inspired from space-pen and will look a little like the following when we our done:

    class Spacecraft extends View
      @content: ->
        @div =>
          @h1 "Spacecraft"
          @ol =>
            @li "Apollo"
            @li "Soyuz"
            @li "Space Shuttle"

Note: For other options you may want to checkout http://ckirkendall.github.io/enfocus-site/#

View.litcoffee HERE

Once we have that utlizeing them ise rather straightforward

Example test HERE

Controller side views are nothing more than a string inserted onto the @el
We can simply add a render method and place a call to it in our constructor. 












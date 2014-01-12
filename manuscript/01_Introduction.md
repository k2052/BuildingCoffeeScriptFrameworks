# Introduction

Web development these days is full of abstractions. Where once we built static pages whose only motion was the occasional misused GIF and inappropriately styled form, we now build full fledge applications, that twirl, swirl, and blaze with a thousand shiny flat icons... but you already know this. 

The web today demands so much of its keepers, we have abstraction after abstraction to keep up with;
frameworks, template engines, packaging systems, compile to JS languages, and even entirely new platforms that just happen to use the web; like Dart and EmScripten. It is daunting just too keep up with all the new doodads much less learn how they work.

This book is about the innards of just one abstraction, that of the CoffeeScript Framework.

Webapp frameworks vary greatly in design but they can be broken down into three main categories;

1. *Component/Module based* These are frameworks designed for large applications. They have an opinionated way of structuring your application but when you follow it you get a lot of magic for free. Rather than simply carry over patterns from other types of app development, they often have invented entirely new ones. Ember.js is the most well known representative of this category.
2. *Platforms* Platforms typically combine a compile-to-js language with a framework of library. Sometimes they even abstract away CSS or HTML. Stuff like [Dart](LinkToDart) and [Objective-J](LinkToObjective-j) fall int this category.
3. *Backbone.js derived* Backbone.js is what started it all. Being first, meant that Backbone borrowed heavily from familiar patterns like MVC. A characteristic of these frameworks is a focus on simple re-usable patterns. They often shine best in small to medium apps. Spine.js, INSERT a couple others here,    and the framework we will building falls into this category.

I will focus on the Backbone variety in this book. We will learn how to implement MVC like patterns and eventually turn those patterns into a full and functional framework.
                 

# Introduction

The web today demands much of its keepers, we have abstraction after abstraction to keep up with;
frameworks, template engines, packaging systems, compile to JS languages, and even entirely new platforms (think TypeScript) that just happen to use the web. It is daunting just too keep up with all the new doodads much less learn how they work.

## The three types of CoffeeScript/JS Frameworks

Modern Web frameworks fall into three main categories;

### 1. *Component/Module based* 

Frameworks designed for large applications. They have an opinionated way of structuring your application but when you follow it you get a lot of stuff for free. Rather than simply carry over patterns from other types of app development, they often have invented entirely new ones. [Ember.js](http://emberjs.com/) is the most well known representative of this category.

### 2. *Platforms* 

Platforms typically combine a compile-to-js language with a framework or library. Sometimes they even abstract away HTML or even CSS. [Dart](https://www.dartlang.org/), [Cappuccino](http://www.cappuccino-project.org/), [ClojureScript](https://github.com/clojure/clojurescript) and [Elm](http://elm-lang.org) fall into this category. 

### 3. *Design Pattern Based/Minimalist* 

These frameworks are built with varying combinations of ModelView*. Some have controllers some use presenters. Some simply opt for placing most of their logic in routes and a controller. What they all have in common is their focus on design patterns and structure. They give you a good starting point and then get out of your way. Backbone.js was the first (and remains the most popular) of this kind but many many have sprung since then, including the one we will build in this book [Ryggrad](https://github.com/ryggrad/Ryggrad)


## How to use this book

### Code

You'll find all the code for this book on Github. Each completed chapter has a corresponding branch. You clone the the repo, switch to the chapters branch and then the code should run with no hiccups.

For example;

    $ git clone https://github.com/k2052/BuildingCoffeeScriptFrameworks.src
    $ cd BuildingCoffeeScriptFrameworks.src

Then checkout the branch:

    $ git checkout chapter-2

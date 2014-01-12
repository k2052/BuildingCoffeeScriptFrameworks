# Building CoffeeScript Frameworks

It might seem a bit odd to begin the book with a section named after the book but I have good reason.
When you build a building you first start by making blueprints, building scaffolding, setting a foundation etc so that you know where, when and how to build it. Where you code and how you code is just as important as what you code.

We are going to learn how to structure, organize, and test our framework first. Only once we have some solid foundations will we get started building.

Before we go any further let's create a folder for our framework and then cd into it:

    $ mkdir Ryggrad && cd Ryggrad

## Setting Up A Dev Environment

### Ubuntu 

Node.js is avaiable in the standard repo but it's outdated so we will install the version
from Chris Lea's repo.


Make sure you update apt and install the prerequisites: 

    $ sudo apt-get update
    $ sudo apt-get install -y python-software-properties python g++ make

Then add the repo:

    $ sudo add-apt-repository -y ppa:chris-lea/node.js
    $ sudo apt-get update

And finally install node (npm will be installed with node):

    $ sudo apt-get install nodejs

Now install Grunt (we will install CoffeeScript later with our grunt tasks):

    $ npm install -g grunt-cli

### OS X

Install Homebrew:

    $ ruby -e "$(curl -fsSL https://raw.github.com/Homebrew/homebrew/go/install)"

Update Homebrew:

    $ brew update

Add brew to your path:

    export PATH="/usr/local/bin:$PATH"

And finally install node (npm will be installed with node):

    $ brew install node

Now install Grunt (we will install CoffeeScript later with our grunt tasks):

    $ npm install -g grunt-cli

## Introducing Bower

The first tool in our arsenal is [Bower](LinkToBower). Bower is a simple package manager for web assets. Stuff like;
javaScript libraries, css frameworks, icon fonts etc. It's written in node.js and is npm package so to install it we simply need to run:

    $ npm install -g bower

Then to install a package using bower we simple need to run:

    $ bower install [package name]

Let's install our first package (*Note*: run this in your project's folder):

    $ bower install jquery

This will install jQuery and place it in bower_components in root of our project. Since we intend for this to be a package installable via bower let's go ahead and generate a bower.json file by running:

    $ bower init

You'll get something like the following:

    -----------------------------------------
    Update available: 1.2.8 (current: 1.2.6)
    Run npm update -g bower to update
    -----------------------------------------

    [?] name: Ryggrad
    [?] version: 0.0.1
    [?] description: "A CoffeeScript FrameWork"
    [?] main file: 
    [?] keywords: 
    [?] authors: 
    [?] license: MIT
    [?] homepage: 
    [?] set currently installed components as dependencies? Yes
    [?] add commonly ignored files to ignore list? No
    [?] would you like to mark this package as private which prevents it from being [?] would you like to mark this package as private which prevents it from being accidentally published to the registry? Yes

    {
      name: 'Ryggrad',
      version: '0.0.1',
      description: '"A CoffeeScript FrameWork"',
      license: 'MIT',
      private: true,
      dependencies:{
        jquery: '~2.0.3'
      }
    }

    [?] Looks good? Yes
{: .language-sh}

That does it for bower, for the moment.

## Assembling Our framework Using Grunt

## Testing Our Framework

## Overview

We should now have a project with a structure like the following:

- *bower_components/*: Contains assets installed via bower.
- *bower.json*: A bower manifest
- *node_modules*: Contains npm managed dependencies. *Note:* you should not git commit this repo
- *Gruntfile.coffee*: Con
- *package.json*: A npm manifest
- *ryggrad.debug.js*: The compiled framework, not minified or compressed, suitable for debugging.
- *ryggrad.js*: The compiled framework. This is in the root of the project so that it can be easily used in a bower asset pipeline.
- *spec/*: Contains our specs.
- *src/*: Contains the source CoffeeScript files of the framework
- *test/* A folder with our test web pages meant to be run by a browse (can be headless).




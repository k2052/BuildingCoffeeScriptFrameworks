---
title: Testing Using Mocha Chai and Sinon
---

There are so many options out there for testing, for good reasons; testing is both incredibly important and something you have to do a lot. When you do something a lot you want it to work just so and when it's important you want it to just work! Everyone has different styles and different things will make it click for them, so it's no wonder
than when ever an itch needs to scratched chances are someone else already had it and scratched it.

I don't want to push my way of testing on you. You'll find your own formula soon enough. I just want share my own little recipe and what works for me. When you see something work, there is a chance that it'll connect and you'll want to work that way.

Mocha, Chai and Sinon are born what one might call the post-mega-testing framwork era. There was ave  time when packang JS was really annoying and distributing it was even more annoying. The first testing frameworks where birthed in this era, they needed to have everything and kitchen sink because every separate bit meant one more hassle. A modular testing framework meant one more shell cscript one more configruation file, two more places for things to go wrong. 

In an arena where things just need to work (nothing is worse nor more ironic than your tests failing because of a failing test framework) monolithic testing frameworks were a necessary evil.

Mocha and their ilk are born of the modern era and are thus modular and broken up. Mocha is just test framework you can match it up with whatever you like. Chai is just the assertion library it can be matched up with whatever else you need. Sinon is just mocking and stubs you can match... You get the point.

No matter what testing frameworks you use there are two pieces of magic that make them all work; 1. PhantomJS and 2. Grunt. PhantomJS is a headless browser used to run tests in a, you guessed it, browser. Grunt is a task runner which is used to automate your testing so you don't have to remember long and verbose CLI arguments to pass PhantomJS, and to make sure you don't forget to run your tests you lazy developer.

## Getting Started

When Mocha is ran against a test suite (in the browser) it generates a series of failing/passing messages. These are appended to the DOM and can easily be parsed. So, if you are clever you can write a script to run PhantomJS against a test page and then you can extract the results output them to the console. Unsurprisingly someone has already written scripts to do this in the form of Grunt tasks. We just need a test page to run the task against.

We will place our tests in *spec/* and our test pages will be *test/*

The first thing we need to do is add the dependencies to our project. Since we are testing in browser we are going to use bower to manage them. If we were testing backend code only we could forgo PhantomJS the browser and bower altogether and simply utilize npm and mocha via node.js. 

Install them:

    $ bower install sinon chai chai-jquery mocha sinon-chai

Now we need to add some Grunt tasks to compile our tests and dependencies and add them to a page with Mocha.

Let's define an array (in our Gruntfile.coffee) with all our test dependencies:

    test_dependecies:
      [
        'bower_components/mocha/mocha.js',
        'bower_components/chai/chai.js',
        'bower_components/sinon-chai/lib/sinon-chai.js',
        'bower_components/sinon/lib/sinon-1.7.3.js'
      ]
{: .language-coffeescript}

And one for all our specs:

    spec: [
      'spec/*_spec.coffee'
    ]
{: .language-coffeescript}

It's probably best to scope these to something that semantically makes sense:

    resources:
      test_dependecies:
        [
          'bower_components/mocha/mocha.js',
          'bower_components/chai/chai.js',
          'bower_components/sinon-chai/lib/sinon-chai.js',
          'bower_components/sinon/lib/sinon-1.7.3.js'
        ]

      spec: [
        'spec/*_spec.coffee'
      ]
{: .language-coffeescript}

Then we can utilize the coffee task to compress and output our tests:

    coffee:
      test:
        files:
          'test/js/lib.js': '<%= resources.test_dependencies %>'
          'test/js/spec.js':   ['<%= resources.spec %>']
{: .language-coffeescript}

This compiles all our dependencies to _test/js/lib.js_ and all our specs into _test/js/spec.js_.
Now we only need to add a test.html page and a task to run it.
Our test page will look like this:

    <!DOCTYPE html>
    <html>
    <head>
        <meta http-equiv="Content-type" content="text/html; charset=utf-8">
        <title>Tests</title>
        <link rel="stylesheet" href="../bower_components/mocha/mocha.css" type="text/css" charset="utf-8" />
    </head>
    <body>
        <!-- Required for browser reporter -->
        <div id="mocha"></div>

        <!-- mocha and other dependencies-->
        <script src="js/lib.js" type="text/javascript" charset="utf-8"></script>
        <script type="text/javascript" charset="utf-8">
          // This will be overridden by mocha-helper if you run with grunt
          mocha.setup('bdd');
        </script>

        <script src="../../phantomjs/bridge.js" type="text/javascript" charset="utf-8"></script>
        <script type="text/javascript" charset="utf-8">
          // Setup chai
          var expect = chai.expect;
          chai.should();
        </script>

        <!-- Spec files -->
        <script src="js/spec.js" type="text/javascript" charset="utf-8"></script>

        <!-- run mocha -->
        <script type="text/javascript" charset="utf-8">
          mocha.run();
        </script>
    </body>
    </html>
{: .language-html}

*Note*: You'll also need to add a task to compile the thing you're testing and link to from the html file.
I typically link directly to the debug file that is built in the build task. For example,

I'll have a Grunt task like:

    coffee:
      src:
        files:
          '<%= meta.file %>.debug.js': '<%= resources.src %>'
{: .language-coffeescript}

And then link to it in my test.html:

    <script src="../ryggrad.debug.js" type="text/javascript" charset="utf-8"></script>
{: .language-html}


Once we have this we only need to configure Mocha and load the Mocha task:

    mocha:
      test:
        src: [ 'test/test.html' ],
        options:
          # Select a Mocha reporter
          # http://visionmedia.github.com/mocha/#reporters
          reporter: 'Spec',

          # Indicates whether 'mocha.run()' should be executed in
          # 'bridge.js'. If you include `mocha.run()` in your html spec,
          # check if environment is PhantomJS. See example/test/test2.html
          run: true,

          # Override the timeout of the test (default is 5000)
          timeout: 10000
{: .language-coffeescript}

Then load the task:

  grunt.loadNpmTasks 'grunt-mocha'

Add register a spec task that compiles our specs and then runs the mocha task:

  grunt.registerTask 'spec', ['coffee:test', 'mocha:test']
{: .language-coffeescript}

Then you can run it using:

    $ grunt spec
{: .language-sh}

See this repo for a simple example and my Ryggrad repo for a full example.

---
title:  Assembling Things With Grunt
---

There was once a dark time in JS world. Assembling things was hard back then. We built our JS projects using
giant monstrosities of such enormous complexity and size that only wisest wizards among us could operate them. These monsters [^Monsters]: took XMl files to compile our xml files to compile our JSON to a task file that was finally ran by a horde of json-to-xml parsers that then ran a shell script. It was hard, complex and rarely worked smoothly. Building with these tools, was simply put, grunt work.

Times have changed. The days have gotten brighter and the nights shorter. The darkness is held back for a time at least, by a plethora of new and simple tools that now populate our land. One tool in particular, rose up to save us, a tool called grunt.

Grunt is a task runner designed to take the grunt work out of assembling a JS project. It has tasks for everything from minification to unit testing. It will make building even the most complicated of JS projects simple

## Installation

Installing grunt is simple:

    $ npm install grunt-cli -g

Installing _grunt-cli_ will make a grunt wrapper command available globally. When you run run `$ grunt` _grunt-cli_ will look for the local/project scoped version of Grunt and then run it. How do we install a local version of Grunt to our project? Simple, we use a package.json file

## package.json

If you have used npm to manage your JS projects then you are familiar with a the package.json [^package.json-guide] format. It holds meta data about your project and any dependencies it has. A basic grunt project would look like this:

    {
      "name": "my-project-name",
      "version": "0.1.0",
      "devDependencies": {
        "grunt": "~0.4.2",
        "grunt-contrib-jshint": "~0.6.3",
        "grunt-contrib-nodeunit": "~0.2.0",
        "grunt-contrib-uglify": "~0.2.2"
      }
    }
{: .language-json}

If have an existing Grunt project you can use npm to append the grunt dependency to package.json:

    $ npm install grunt --save-dev

Once you have your package.json you can install grunt by running:

    $ npm install .

We can then run grunt against our project:

    $ grunt

For now, running this will do nothing. How do we make it do something? We need to add a Gruntfile

## Gruntfile

Grunt is configured with a Cruntfile.coffee or Gruntfile.js file in the root of a project.

A basic Gruntfile looks like:

    module.exports = (grunt) ->
      
      # Project configuration.
      grunt.initConfig
        pkg: grunt.file.readJSON("package.json")
        uglify:
          options:
            banner: "/*! <%= pkg.name %> <%= grunt.template.today(\"yyyy-mm-dd\") %> */\n"

          build:
            src: "src/<%= pkg.name %>.js"
            dest: "build/<%= pkg.name %>.min.js"

      
      # Load the plugin that provides the "uglify" task.
      grunt.loadNpmTasks "grunt-contrib-uglify"
      
      # Default task(s).
      grunt.registerTask "default", ["uglify"]

But what if we want to do something more interesting with our Gruntfile?

## The Anatomy of a Grunt Project

Let's say you have a project and it looks like this;

- _src/_ Which contains a bunch of CoffeeScript source files
- _spec_ Which contains some specs that need to be ran in a browser

The first thing we need to do is define some vars that hold the paths to those files

    resources:
      src: [
        'src/*.coffee'
      ]
      spec: [
        'spec/*_spec.coffee'
      ]
{: .language-coffeescript}

Paths can both be patterns or specific files. If we wanted to control source order we could do something like:

    resources:
      src: [
        'src/file1.coffee',
        'src/file2.coffee'
      ]
{: .language-coffeescript}

    coffee:
      options:
        join: true
      src:
        files:
          '<%= pkg.name %>.debug.js': '<%= resources.src %>'
      test:
        files:
          'test/js/<%= pkg.name %>.js': '<%= resources.src %>'
          'test/spec/spec.js': ['<%= resources.spec %>']
{: .language-coffeescript}

This uses the coffee task to output a build file to the root called pkg.name.debug.js (pkg.name will be replaced
with pkg.name) and our specs to _root/test/spec_. Where does the _coffee_ task come from? We loud tasks by doing the following:

    grunt.loadNpmTasks 'grunt-contrib-coffee'
{: .language-coffeescript}

This loads the tasks into our Gruntfile and makes them available. Let's load some for minification, watching files and running our tests:

    grunt.loadNpmTasks 'grunt-contrib-uglify'
    grunt.loadNpmTasks 'grunt-contrib-watch'
    grunt.loadNpmTasks 'grunt-mocha'
{: .language-coffeescript}

Then configure them

    uglify:
      options:
        compress: false
        banner: '<%= meta.banner %>'
      endpoint:
        files: '<%=meta.file%>.js': '<%= meta.file %>.debug.js'

    watch:
      src:
        files: '<%= resources.src %>'
        tasks: ['coffee:src', 'uglify']
    
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

Now one last thing, how do we run them? We have to register them with grunt and then we call them using the correct scope. For example, our coffee:src task would be ran using grunt coffee:src. Here is how we register them:

    grunt.registerTask 'default', ['coffee:src', 'uglify']
    grunt.registerTask 'spec', ['coffee:test', 'mocha:test']
{: .language-coffeescript}

Our final Gruntfile looks a bit like this:

    module.exports = (grunt) ->
      grunt.initConfig
        pkg: grunt.file.readJSON "package.json"

        meta:
          file: 'ryggrad'
          endpoint: 'package',
          banner: '/* <%= pkg.name %> v<%= pkg.version %> - <%= grunt.template.today("yyyy/m/d") %>\n' +
                  ' <%= pkg.homepage %>\n' +
                  ' Copyright (c) <%= grunt.template.today("yyyy") %> <%= pkg.author.name %>' +
                  ' - Licensed under <%= pkg.license %> */\n'

        resources:
          src: [
            'src/*.coffee',
          ]
          spec: [
            'spec/*_spec.coffee'
          ]


        coffee:
          options:
            join: true
          src:
            files:
              '<%= pkg.name %>.debug.js': '<%= resources.src %>'
          test:
            files:
              'test/js/<%= pkg.name %>.js': '<%= resources.src %>'
              'test/spec/spec.js': ['<%= resources.spec %>']

        uglify:
          options:
            compress: false
            banner: '<%= meta.banner %>'
          endpoint:
            files: '<%=meta.file%>.js': '<%= meta.file %>.debug.js'

        watch:
          src:
            files: '<%= resources.src %>'
            tasks: ['coffee:src', 'uglify']
        
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

      grunt.loadNpmTasks 'grunt-contrib-coffee'
      grunt.loadNpmTasks 'grunt-contrib-uglify'
      grunt.loadNpmTasks 'grunt-contrib-watch'
      grunt.loadNpmTasks 'grunt-mocha'

      grunt.registerTask 'default', ['coffee:src', 'uglify']
      grunt.registerTask 'spec', ['coffee:test', 'mocha:test']
{: .language-coffeescript}

That is it!

Take a look at my CoffeeScript framework Ryggrad for a real world example of this workflow in use:
[https://github.com/k2052/ryggrad](https://github.com/k2052/ryggrad)

[Monsters]: Monsters like Ant and Apache build.
[package.json-guide]: An excellent interactive guide on the package.json format is available at http://package.json.nodejitsu.com/

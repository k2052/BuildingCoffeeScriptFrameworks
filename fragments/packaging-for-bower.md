---
title: Packaging Assets For Bower
---

Bower is a very simple package management system for frontend stuff (js, css, images etc). It leaves all the details
of utilizing the packages up to the rest of your build stack. Bower is essentially just a registry of git repos that contain web assets. This simplicity provides a great deal of flexibitiy and allows bower to be utilized in the assets pipelines for all sorts of stacks. 

Packaging assets for bower is straightforward. You'll need the following;

1. Final versions of the assets i.e a compressed css and and compressed JS file.
2. A manifest file called bower.json. Should be placed in the root of your git repo
3. A git repo for bower to pull things from. You'll need to make sure you use [semantic versioning tags](http://semver.org).

The manifest file (`bower.json`) consists of the following:

- `name` (required)[string]: The name of the package
- `version` (required)[string]: The version of the package
- `description` [string]: A description
- `homepage` [string]: A homepage
- `keywords` [array]: Keywords to make it easy to find you package via `bower search`
- `author` [string]: An author string with email formatted like: `K-2052 <k@2052.me>`
- `repository` [string]: The git repo where the package is located
- `main` [string|array]: The primary endpoints of your package.
- `ignore` [array]: An array of paths that you don't want Bower to copy
- `dependencies` [hash]: Production dependencies
- `devDependencies` [hash]: Development dependencies.

Then to register your package all you need to do is:

    $ bower register <my-package-name> <git-endpoint>

Remember to tag each new release semantically; when bower installs a package with a specified version it will look for the git tag for that version. 

That is it! If you need to look at an example package checkout https://github.com/k2052/ryggrad
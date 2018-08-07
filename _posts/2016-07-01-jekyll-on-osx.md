---
title: "Jekyll on OSX"
categories:
  - osx
  - jekyll
tags:
  - osx
  - jekyll
summary:
  - image: osx.png
---
While Jekyll and its dependencies can be easily installed in Linux, Mac OSX and Windows need some more work to get a local Jekyll running.<more>

**Prerequisites**

* Mac OSX Yosemite or higher
* Xcode: Available from the Mac App Store

**Xcode**

Xcode is needed for Jekyll to run on OSX. Just install Xcode from the Mac App Store and start it once (agree to the terms and conditions). Once that is done, just take sure everything is installed by opening the `Terminal` (open Spotlight and type `terminal`) and enter

```shell
$ xcode-select --install
```

**Install Jekyll**

After installing Xcode you have all needed dependencies for running Jekyll. Just open up the `Terminal` (open Spotlight and type `terminal`) and enter

```shell
$ sudo gem install jekyll
```

This will install Jekyll and all of its Ruby dependencies.

After that you can use Jekyll locally on your OSX system.

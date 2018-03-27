---
title: "Environment Isolation"
date: 2018-03-05
categories:
  - python
  - tutorial
tags:
  - python
  - development
  - environment
  - tutorial
  - beginner
  - tox
  - pyenv
keywords:
  - python
  - pyenv
  - tox
  - virtualenv
  - pyenv-virtualenv
  - tutorial
  - beginner
  - development
  - environment
thumbnailImagePosition: left
thumbnailImage: ./images/first.jpg
coverImage: ../../../images/first.jpg
coverCaption: Photo Credit https://unsplash.com/@claudelrheault
showMeta: false
draft: true
---



## A Note on Environment Isolation

So some people might be asking, why would I want to install python? Isn't it already on my machine? Well, yes in most linux (and MacOS) distributions there is indeed a python already installed on the machine. So why bother installing another? Well, for starters the version of python installed might not be the version you want, in fact it almost certainly isn't the newest version.  I'll refer to this installation of python as the system's python. You should view the system's python installation as "off-limits." AKA Don't touch it!

The system's python is for the system's use. So unless you're building something for your Operating Systems' distribution, you should really stay away from it. There are several key benefits from leaving it alone:

1. You won't accidentally break your system - Many system tools (like yum) are expecting specific versions of packages to exist, and if you erase them, these tools suddenly break. You can get away with installing and playing with your system python for a while, but in my experience it always comes back to bite you in the butt.

2. You don't want to have to use `sudo` to install things - If you're installing a python package at the system-level, it totally makes sense that you would have to `sudo` your `pip install` commands. That means you're going to have to be operating as the super user more often than you would like. Operating as the superuser should be avoided wherever possible to reduce the possibility of irreparable  damage to your operating system

3. You might want a different python - The system python comes as is. You don't get to recompile it, or pick its implementation. You get a Cpython by default. There are lots of ways to customize the python installation and even implementation (see pypy, ironpython, or cython)! Well to do any of those, you're going to need to install this somewhere other than your system python.

Typically, beginners can understand why we would want our own python and don't to share the system python. However, it is often a struggle to understand why you would want isolate environments for each of your projects. TODO: Turn this into its own post.

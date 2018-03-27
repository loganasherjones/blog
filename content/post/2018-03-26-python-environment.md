---
title: "Setting up your Python environment"
date: 2018-03-26
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
thumbnailImage: ./images/greenfield.jpg
coverImage: ../../../images/greenfield.jpg
coverCaption: Photo Credit https://unsplash.com/@claudelrheault
showMeta: false
---

Whether you are a beginner or seasoned software developer, this is a great getting started guide for standing up a complete, "professional" python development environment in Linux or MacOS.
<!--more-->

<!-- toc -->

# Introduction

I've been writing Python code for the last 4 years. During that time, I've had to install many different python versions on many different Operating Systems. I've struggled with dependency management and have learned lessons along the way that I'd like to share here. By the end of this article, you'll be able to:

* Install any verison of python you want
* Have isolated environments for each of your projects
* Be able to run tests in multiple python languages

The tools I am using currently are:

* [Pyenv][1] - to manage python installations
* [Pyenv-virtualenv][2] - to manage python versions
* [tox][3] - for testing in mulitple python versions

{{< alert info >}}
In the future, I will be transitioning to [pipenv](https://docs.pipenv.org/) but that will be its own blog post. Pipenv is definitively the better package manager, but there are some integration parts that need to be ironed out before I feel comfortable moving over. I believe once pip 2.0.0 comes out, I will likely switch over. Until then, here's what I've got!
{{< /alert >}}

# Installing Python

We will be demonstrating how to install python on Ubuntu, but the general process is the same on most Linux distributions. Before we can actually install Python, we have to install some of the base dependencies. These dependencies are used to build Python itself. Without them, we simply cannot install Python. So, open up your favorite package manager, and its time to install some dependencies:

{{< tabbed-codeblock "Install Dependencies" >}}

<!-- tab Ubuntu -->
sudo apt install build-essential libssl-dev libbz2-dev libsqlite3-dev libreadline-dev
<!-- endtab -->

<!-- tab MacOS -->
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew install xz pkg-config sphinx-doc gdbm openssl readline sqlite tcl-tk
<!-- endtab -->

<!-- tab CentOS -->
sudo yum groupinstall "Development Tools"
sudo yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel
<!-- endtab -->

<!-- tab Alpine -->
sudo apk add bzip2-dev \
		coreutils \
		dpkg-dev dpkg \
		expat-dev \
		gcc \
		gdbm-dev \
		libc-dev \
		libffi-dev \
		linux-headers \
		make \
		ncurses-dev \
		libressl \
		libressl-dev \
		pax-utils \
		readline-dev \
		sqlite-dev \
		tcl-dev \
		tk \
		tk-dev \
		xz-dev \
		zlib-dev
<!-- endtab -->

{{< /tabbed-codeblock >}}

Next, we will be installing `pyenv` which will require git. So we're going to install [git][4] as well.

{{< tabbed-codeblock "Install Git" >}}

<!-- tab Ubuntu -->
sudo apt install git
<!-- endtab -->

<!-- tab MacOS -->
brew install git
<!-- endtab -->

<!-- tab CentOS -->
sudo yum install git
<!-- endtab -->

<!-- tab Alpine -->
sudo apk add git
<!-- endtab -->

{{< /tabbed-codeblock >}}

With git installed, we are going to install [Pyenv][1]. Pyenv helps us manage multiple versions of python simultaneously. Having multiple python versions on your machine is very valuable. Even if the application you're going to build/work on is only targeted for a single python installation, it is likely that you'll want to update to the latest version. Alternatively, if you're creating a package for use by other developers, you might want to test integration with older python versions. Pyenv makes managing these installations much easier.

{{< codeblock "Install Pyenv" >}}
git clone https://github.com/pyenv/pyenv.git ~/.pyenv
{{< /codeblock >}}

This will install pyenv into your home directory. To activate pyenv and make it usable in your environment you'll need to put the following in your shell's profile. If you're using Zsh, you'll place this in `~/.zshenv` for Ubuntu/Fedora users, the following will be placed in `~/.bashrc`

{{< codeblock "~/.bashrc" bash >}}
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
if command -v pyenv 1>/dev/null 2>&1; then
  eval "$(pyenv init -)"
fi
{{< /codeblock >}}

Now either source this file, or restart your terminal session
{{< codeblock "Source" Shell>}}
exec $SHELL
{{< /codeblock >}}

Now that we have pyenv installed along with the python dependencies, installing will be easy! There are many different python installations that you could want. You can see all the options available by running:

{{< codeblock "List All Pythons" >}}
pyenv install -l
{{< /codeblock >}}

{{< alert info >}}
If the version of python you want was not listed by the above command, try cloning the newest version of the git repo. Altenratively, you could install [pyenv-update](https://github.com/pyenv/pyenv-update) and run `pyenv update`
{{< /alert >}}

Now we can install any python version we want. For this post, I'll be showing a small package which attempts to support Python 2 and Python 3. We can install each:

{{< codeblock "Install Python" >}}
pyenv install 3.6.4
pyenv install 2.7.14
{{< /codeblock >}}

This may take a bit, but at the end you should have a python installation. Feel free to install whatever versions you want including [pypy][5], [ironpython][6] or any other python version. The rest of the tutorial will assume you want at least two different versions.

# A Quick Look at Pyenv

The [pyenv documentation](https://github.com/pyenv/pyenv#how-it-works) is a great resource to understand how pyenv works, but I will try to summarize the bits I think are important here and demo some of the main commands you will want to use. In short, pyenv works by modifying your `PATH` dynamically. This has a couple of weird consequences, but they are worth living with. First, let's inspect what version of Python we are running:

{{< alert info >}}
All of the following output is based on ubuntu 16.04 LTS, so if you see differences, don't panic!
{{< /alert >}}

{{< codeblock "Check Python Version" Shell>}}
logan@blog:~$ python -V
Python 2.7.12
{{< /codeblock >}}

Hmm, we installed python 2.7.14 and 3.6.4 above, so why is this 2.7.12? Well, it just so happens that Ubuntu ships with its own python. So let's try to figure out which python we're refrencing when we run `python`.

{{< codeblock "Check Python Installation" >}}
logan@blog:~$ which python
/home/logan/.pyenv/shims/python
{{< /codeblock >}}

If you were expecting `/usr/bin/python` or something else, don't worry so was I the first time I ran this command. This shims directory is how pyenv is able to edit your python at runtime. It injects this into your `PATH` and then dynamically changes the linking.

Great but, where is the python installation on disk? Pyenv allows you to answer that with the following commmand:

{{< codeblock "Check Python Installation with pyenv" >}}
logan@blog:~$ pyenv which python
/usr/bin/python
{{< /codeblock >}}

Here we can see that we are using the system python. Preferably, we never want to touch the system's python. There is an easy way to do this. We simply need to tell pyenv that we want to use a different python.

{{< codeblock "Set global python version" >}}
logan@blog:~$ pyenv global 3.6.4
{{< /codeblock >}}

This sets your global python as 3.6.4. So now when you ask for the version:

{{< codeblock "Check Python Version" >}}
logan@blog:~$ python -V
Python 3.6.4
{{< /codeblock >}}

But what if we want to run python 2.7.14? Let's try it:

{{< codeblock "Run Python2.7" >}}
logan@blog:~$ python2.7 -V
pyenv: python2.7: command not found

The `python2.7` command exists in these Python versions:
  2.7.14
{{< /codeblock >}}

Pyenv successfully found your executable, but it cannot run it because that python environment is not active. We could of course swap the environment by simply running `pyenv global 2.7.14` but this is a bit of a pain when you want to switch back and forth. I like to have both python verisons on my path at the same time so I can quickly test different versions when necessary. Luckily, pyenv allows you to do this:

{{< codeblock "Set Multiple Global Python Versions" >}}
logan@blog:~$ pyenv global 3.6.4 2.7.14
logan@blog:~$ python -V
Python 3.6.4
logan@blog:~$ python3.6 -V
Python 3.6.4
logan@blog:~$ python2.7 -V
Python 2.7.14
{{< /codeblock >}}

We can see that by default, `python` will resolve to 3.6.4, because we placed it first on the list. However, each individual python is callable at the same time.

# Setting up a Virtual Environment

Now that we know how pyenv works, let's try to apply it on a project. I have a simple [toy package up on GitHub][7] that we can clone. Let's clone our toy package.

{{< codeblock "Clone the toy project" >}}
git clone https://github.com/loganasherjones/toy_package.git
cd toy_package
{{< /codeblock >}}

In theory, we could just go ahead and install all of our requirements with `pip install` but that would muddy up our global Python 3.6.4 version. We really don't want that. That's where the concept of [virtual environments](https://docs.python.org/3/tutorial/venv.html) come into play. You should have at least one virtual environment per project. Luckily, there is a python package named [virtualenv](https://virtualenv.pypa.io/en/stable/) that can help with this problem. Even more lucky is that there is a plugin to pyenv that allows us to use this tool. Let's install [pyenv-virtualenv](https://github.com/pyenv/pyenv-virtualenv)

{{< codeblock "Install pyenv-virtualenv" >}}
git clone https://github.com/pyenv/pyenv-virtualenv.git $(pyenv root)/plugins/pyenv-virtualenv
{{< /codeblock >}}

Similar to pyenv itself, we now need to activate pyenv-virtualenv Add the following to your `~/.bashrc` (or equivalent):

{{< codeblock "~/.bashrc" bash >}}
if command -v pyenv virtualenv 1>/dev/null 2>&1; then
    eval "$(pyenv virtualenv-init -)"
fi
{{< /codeblock >}}

Restart your shell or source your `~/.bashrc`

{{< codeblock "Restart your shell" >}}
exec $SHELL
{{< /codeblock >}}

Now we can create a virtual environment for this toy package:

{{< codeblock "Create your first virtual environment" >}}
pyenv virtualenv 3.6.4 toy_package
{{< /codeblock >}}

This created a new virtual environment named "toy_package". We specified that this virtual environment should use python 3.6.4. Virtual environments are very important concepts. They help to keep our python environments clean. This is important because it is very easy to accidentally install different versions of the same package which can cause all kinds of nasty bugs and waste countless hours (don't ask me how I know). To avoid them, we will activate our new virtual environment. Think of this virtual environment as a personal playground for your project. You can do whatever you want to this playground and always be a few commands away from a fresh start.

{{< codeblock "Active your virtual environment" >}}
pyenv local toy_package
{{< /codeblock >}}

This tells pyenv that we want to use the python associated with this virtual environment. Not only is it using a separate `python`, it is also using a separate `pip`! We can prove that via:

{{< codeblock "Install pyenv-virtualenv" >}}
logan@blog:~$ pyenv which pip
/home/logan/.pyenv/versions/toy_package/bin/pip
{{< /codeblock >}}

Here you can see that there is a separate virtual environment in `~/.pyenv/versions/toy_package` with its own pip installation and everything. Now we can install our dependencies safely!

{{< codeblock "Install the dependencies" >}}
pip install -r requirements.txt
{{< /codeblock >}}

This installs all our python dependencies. Python has a rich environment of over 250,000 packages so its quite common to see a `requirements.txt` file in the root of our repository. Sometimes you'll see them broken down into `requirements.txt` and `requirements_dev.txt`. Their purpose is to list all the package dependencies our project needs to do local development. One of the packages our project lists is pytest. This allows us to run tests on our application. Let's run some and make sure they work!

{{< codeblock "Running a test" >}}
logan@blog:~$ pytest
==================== test session starts ===========================
platform linux -- Python 3.6.4, pytest-3.5.0, py-1.5.3, pluggy-0.6.0
rootdir: /home/logan/workspace/toy_package, inifile:
collected 1 item

tests/package_test.py .                                       [100%]

================== 1 passed in 0.05 seconds ========================
{{< /codeblock >}}

Great! Our tests passed. If you are only interested in running on a single python version, then you can stop here. However, if you want to test on multiple python versions then you'll need to use a different tool called [tox][3]

# Testing against Multiple Python Versions

Testing in multiple python versions is a common occurrence for package maintainers. If you are interested in writing a package or just interested in maximizing portability, you should really be using [tox][3]. Tox is a tool for testing. It can test with differing virtual environments and even switch python versions.

Tox requires a configuration file to tell it what tests you would like to run. This file is the `tox.ini` file and should be found at the same level the `setup.py` file is found. The toy package already has a `tox.ini`, let's take a look at it:


{{< codeblock "tox.ini" "ini" >}}
[tox]
envlist = py27,py36
[testenv]
deps = -rrequirements.txt
commands = pytest
{{< /codeblock >}}

Basically, this file tells us we want two environments: `py27` and `py36`. For each test environment we are then going to install the dependencies by installing what is in `requirements.txt`. This is specified by `deps = -rrequirements.txt`. Finally, in each environment we are going to run the command `pytest`. Let's initiate tox

{{< codeblock "shell_commands" "bash" >}}
logan@blog:~$ tox
GLOB sdist-make: /home/logan/workspace/toy_package/setup.py
py27 create: /home/logan/workspace/toy_package/.tox/py27
ERROR: Error creating virtualenv. Note that some special characters (e.g. ':' and unicode symbols) in paths are not supported by virtualenv.
Error details: InvocationError('Failed to get version_info' for python2.7: b"pyenv: python2.7: command not found\\n\\nThe `python2.7\' command exists in these Python versions:\\n  2.7.14\\n\\n"',)
ERROR: Error creating virtualenv. Note that some special characters (e.g. ':' and unicode symbols) in paths are not supported by virtualenv.
Error details: InvocationError('Failed to get version_info' for python3.6: b"pyenv: python3.6: command not found\\n\\nThe `python3.6\' command exists in these Python versions:\\n  3.6.4\\n\\n"',)
py36 create: /home/logan/workspace/toy_package/.tox/py36
ERROR: 	py27: Error creating virtualenv. Note that some special characters (e.g. ':' and unicode symbols) in paths are not supported by virtualenv. Error details: InvocationError('Failed to get version_info' for python2.7: b"pyenv: python2.7: command not found\\n\\nThe `python2.7\' command exists in these Python versions:\\n  2.7.14\\n\\n"',)
ERROR: 	py36: Error creating virtualenv. Note that some special characters (e.g. ':' and unicode symbols) in paths are not supported by virtualenv. Error details: InvocationError('Failed to get version_info' for python3.6: b"pyenv: python3.6: command not found\\n\\nThe `python3.6\' command exists in these Python versions:\\n  3.6.4\\n\\n"',)
{{< /codeblock >}}

Yikes! Lots of errors. But the error message is actually coming from not being able to resolve the differing python environments. Tox is attempting to create special virtual environments for each of the test environments. Of course it needs python to be able to do that. So let's make sure python3.6 and 2.7 are available. The way we did this before was by using `pyenv global` to set up multiple versions. It turns out pyenv allows us to do the same thing with the `local` command:

{{< codeblock "Activate Local Environments" "bash" >}}
pyenv local toy_package 3.6.4 2.7.14
{{< /codeblock >}}

Now that each of the python versions is on the path and available, we can run tox again:

{{< codeblock "shell_commands" "bash" >}}
logan@blog:~$ tox
{{< /codeblock >}}

This time, we have a new failure, but it is because of a bug in our code. I'll leave solving this bug up to you! If you can't figure it out you could checkout the answer in the `solution` branch:

{{< codeblock "shell_commands" "bash" >}}
logan@blog:~$ git checkout solution
logan@blog:~$ git diff master
{{< /codeblock >}}

# Conclusion

This is just how I choose to do my python development. You can always modify, and I encourage you to look at other tools if you'd like. I've found that this setup works quite well for me and lets me support easily swapping between projects quickly. I've barely scratched the surface with lots of these tools and there is still lots more to learn. At the very least I hope you can now:

1. Install python easily - `pyenv install X.X.X`
2. Setup multiple virutal environments easily - `pyenv virtualenv myenv`
3. Activate multiple python versions - `pyenv local myenv 3.6.4 2.7.14`
4. Run tests against multiple versions - `tox`

Thanks for reading! If you have questions or topic you'd like to see me blog about, hit me up on twitter [@loganasherjones](https://twitter.com/LoganAsherJones)


[1]: https://github.com/pyenv/pyenv
[2]: https://github.com/pyenv/pyenv-virtualenv
[3]: https://tox.readthedocs.io/en/latest/
[4]: https://git-scm.com/
[5]: https://pypy.org/
[6]: http://ironpython.net/
[7]: https://github.com/loganasherjones/toy_package

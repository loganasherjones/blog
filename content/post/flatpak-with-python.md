---
title: "Using Flatpak with Python"
date: 2018-05-17
categories:
  - python
  - beeware
  - linux
tags:
  - python
  - development
  - beeware
  - briefcase
  - packaging
  - flatpak
  - linux
  - distribution
keywords:
  - python
  - development
  - beeware
  - briefcase
  - packaging
  - flatpak
  - linux
  - distribution
thumbnailImagePosition: right
thumbnailImage: ./images/second.jpg
coverImage: ../../../images/second.jpg
coverCaption: Photo Credit https://www.katiehowellphotography.com/
showMeta: false
draft: false
---

A guide to get you started writing python applications for all Linux distributions with flatpak.
<!--more-->

I just got back from PyCon 2018 in Cleveland. It was a whirlwind tour of Python and I met some
great people along the way. One of the projects that really peaked my interest was
[BeeWare](https://pybee.org/) which is a suite of tools that I plan on going over in a different
blog post. The long-and-short of it is that BeeWare is a set of tools that helps you build native
applications with Python. Sweet!

As a first timer at PyCon, I decided to try the [PyCon
Sprints](https://us.pycon.org/2018/community/sprints/).
The BeeWare team was very welcoming to people of all skill levels so I was very interested in
trying to contribute something the single day I was there. I chose to work on the
[Briefcase](https://github.com/pybee/briefcase/) which is the tool that actually packages the
applications you write in [Toga](https://github.com/pybee/toga/). The linux support needed to be
bumped up, so I picked up [the linux support issue on GitHub](https://github.com/pybee/briefcase/issues/2).
This led me to start investigating an interesting tool called [Flatpak](https://flatpak.org/).
Flatpak helps developers publish on every Linux distribution. This sounded like a perfect
product for Briefcase so I started trying to build my first mini python package with flatpak. It
was not as easy as I hoped, so I'm going to document what I did here so anyone can follow along.

# Installing Flatpak

The first step in this journey is to install Flatpak. The flatpak website has some great
[installation instructions](https://flatpak.org/setup/) for every Linux distribution. I'll be
going through this for my platform (Ubuntu) but you can follow the steps for yourself. It's a very
simple process

#### 1. Install Flatpak itself:
{{< codeblock "Install Flatpak" bash >}}
sudo add-apt-repository ppa:alexlarsson/flatpak
sudo apt update
sudo apt install flatpak
{{< /codeblock >}}

#### 2. Install the Software Flatpak plugin
This allows you to install apps without the command-line:
{{< codeblock "Install Flatpak Plugin" bash >}}
sudo apt install gnome-software-plugin-flatpak
{{< /codeblock >}}

#### 3. Add the flathub repository
This is where all the flatpak artifacts and runtimes are stored, you will need it.
{{< codeblock "Add flathub repository" bash >}}
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
{{< /codeblock >}}

#### 4. Restart your computer.

{{< codeblock "Restart your computer" bash >}}
sudo reboot
{{< /codeblock >}}

#### 5. Install flatpak-builder
{{< codeblock "Install flatpak-builder" bash >}}
sudo apt install flatpak-builder
{{< /codeblock >}}

# Hello World With Flatpak

The flatpak tutorial for [building your first Flatpak](http://docs.flatpak.org/en/latest/first-build.html)
does a great job laying out the basic commands, but I'll cover them again here.

#### 1. Install an SDK
The SDK will have everything we will need to build Python later. Flatpak also has a
[List of available runtimes](http://docs.flatpak.org/en/latest/available-runtimes.html) if this
one doesn't work for you.

{{< codeblock "Install the freedesktop SDK Runtime" bash >}}
flatpak install flathub org.freedesktop.Platform//1.6 org.freedesktop.Sdk//1.6
{{< /codeblock >}}


#### 2. Create an app
This will be a simple hello world:

{{< codeblock "hello.sh" bash >}}
#!/bin/sh
echo "Hello world, from a sandbox"
{{< /codeblock >}}

#### 3. Add a manifest

{{< codeblock "org.flatpak.Hello.json" json >}}
{
    "app-id": "org.flatpak.Hello",
    "runtime": "org.freedesktop.Platform",
    "runtime-version": "1.6",
    "sdk": "org.freedesktop.Sdk",
    "command": "hello.sh",
    "modules": [
        {
            "name": "hello",
            "buildsystem": "simple",
            "build-commands": [
                "install -D hello.sh /app/bin/hello.sh"
            ],
            "sources": [
                {
                    "type": "file",
                    "path": "hello.sh"
                }
            ]
        }
    ]
}
{{< /codeblock >}}

The manifest is the key to understanding flatpak. So I'm going to give a brief overview of the
key-value pairs laid out here before moving on. This is where all the magic happens.

* `app-id` - A unique application identifier, it takes the form of a three-part identifier.
* `runtime` - This is provided by flatpak and is a base runtime on which there are many build
dependencies, in theory you could build your own, but generally you shouldn't.
* `runtime-version` - The version of the `org.freedesktop.Platform` runtime
* `sdk` - The name of the SDK to use on top of this runtime
* `command` - This will determine the entrypoint to our application
* `modules` - This lists all the steps to add all of our dependencies to this runtime.
  * `name` - The name of the module, which is required for each module.
  * `buildsystem` - The way to build this particular module, `"simple"` just means that we will run specific commands to build the application
  * `build-commands` - A list of commands to execute
  * `sources` - A list of all the sources for this module, in our case, only a single `file` called `hello.sh` (which we created earlier)

A complete list of all options for the manifest can be found in the [flatkpak documentation](http://docs.flatpak.org/en/latest/flatpak-builder-command-reference.html)

#### 4. Build the application

{{< codeblock "Build your first flatpak app" bash >}}
flatpak-builder build-dir org.flatpak.Hello.json
{{< /codeblock >}}

#### 5. Run the application

{{< codeblock "Run your first flatpak app" bash >}}
flatpak-builder --run build-dir org.flatpak.Hello.json hello.sh
{{< /codeblock >}}

Once you've done that, you should see the message printed out from our shell script! Hooray
you've created your first application!

# What's next?

Unfortunately, that's where the flatpak installation instructions start to falter. This is
obviously the most simple thing you can do, and works correctly, but now the real questions start.
I wanted to run my python script. In order to do that, I need to install Python in my sandbox. The
"sandbox" is flatpak's isolated environment for applications. If you're familiar with docker you
can think of the sandbox as the container layer.

Installing Python can be achieved by modifying the `org.flatpak.Hello.json`. We need to add
another module. We will call it `cpython`:

{{< codeblock "org.flatpak.Hello.json" json >}}
    . . .
    "modules": [
        {
            "name": "cpython",
            "sources": [
                {
                    "type": "archive",
                    "url": "https://www.python.org/ftp/python/3.6.5/Python-3.6.5.tar.xz",
                    "sha256": "f434053ba1b5c8a5cc597e966ead3c5143012af827fd3f0697d21450bb8d87a6"
                }
            ]
        },
        . . .
    ]
}
{{< /codeblock >}}

If you build this application:

{{< codeblock "Build python in your sandbox" bash >}}
flatpak-builder build-dir org.flatpak.Hello.json --force-clean
{{< /codeblock >}}

You'll notice that you're actually building Python from source. While super powerful, this was
very confusing for me at first. As it turns out, flatpak recognizes that the source you specified
is a compressed file (which you told it with `archive`), but what's more surprising was that
flatpak recognized that the extracted components actually contained a GNU Build System inside it
(i.e. a `configure` and `Makefile`) and so it took it upon itself to run the installation. For the
python application, we don't actually need any special flags, but you can tweak the configure and
make flags with the `config-opts` and `make-args` build options.

Flatpak successfully installed python 3.6.5 in `/app`. You can test this with the following:

{{< codeblock "Run python in your sandbox" bash >}}
flatpak-builder --run build-dir org.flatpak.Hello.json /app/bin/python3
{{< /codeblock >}}

This should drop you into a python3 shell. Go ahead and look at `sys.path`!

Now that python is in our application environment, let's write a script and execute it from that
environment.

{{< codeblock "hello.py" python >}}
def main():
    print("Hello, world from python sandbox.")

if __name__ == "__main__":
    main()
{{< /codeblock >}}

In theory you could make this file executable and point it to `/app/bin/python`, but I don't think
that is very portable. So, I usually create a runner script and use that as my entrypoint for
flatpak.  Something like the following:

{{< codeblock "runner.sh" bash >}}
#!/bin/sh
python3 /app/hello.py
{{< /codeblock >}}

Now let's update our manifest to let flatpak know about these new files:

{{< codeblock "org.flatpak.Hello.json" json >}}
    "command": "runner.sh",
    "modules": [
        {
            "name": "cpython",
            "sources": [
                {
                    "type": "archive",
                    "url": "https://www.python.org/ftp/python/3.6.5/Python-3.6.5.tar.xz",
                    "sha256": "f434053ba1b5c8a5cc597e966ead3c5143012af827fd3f0697d21450bb8d87a6"
                }
            ]
        },
        {
            "name": "runner",
            "buildsystem": "simple",
            "build-commands": [
                "install -D runner.sh /app/bin/runner.sh",
                "install -D hello.py /app/hello.py"
            ],
            "sources": [
                {
                    "type": "file",
                    "path": "hello.py"
                },
                {
                    "type": "file",
                    "path": "runner.sh"
                }
            ]
        }
    ]
{{< /codeblock >}}

Here you can see we've added our `hello.py` file to `/app/hello.py` and `runner.sh` to `/app/bin/runner.sh`.  We also need to update our `command` in the main manifest to `runner.sh`.

Now if we re-build and run:

{{< codeblock "Run Hello World from Python" bash >}}
flatpak-builder --force-clean build-dir org.flatpak.Hello.json
flatpak-builder --run build-dir org.flatpak.Hello.json runner.sh
{{< /codeblock >}}

You should see `Hello, World from python sandbox.` You might also notice that we've never rebuilt
python. That's because flatpak is smart enough to cache our states as we go along, so our
build times are only slow the first time. Then we can iterate quickly!

# Packaging Dependencies

While Python is "batteries included" there are still lots of useful packages. Getting packages
into our flatpak sandbox can be done, but we're going to need to introduce a couple of new
options to our manifest.

There are several ways to get a package that you want into your sandbox. You could download the
package locally and then just references in your `sources` section with the `file` type. This
really isn't very portable as other people will not be able to build your application without
downloading the same files.

Another way that we could do it would be to add the source as an archive the same way we did with
Python itself. If we did this, we would have to make sure to know the sha256 hash, which is good
from a security perspective, but can be painful to update, but at least people don't have to
manually download a file before building.

The way I would prefer is to actually just use `pip` to install my package. To do this, we'll
introduce a new module that we'll call `pip-install`.

{{< codeblock "org.flatpak.Hello.json" json >}}
    "modules": [
        . . .
        {
            "name": "pip-install",
            "buildsystem": "simple",
            "build-options": {
              "build-args": [
                "--share=network"
              ]
            },
            "build-commands": [
                "pip3 install requests"
            ]
        }
        . . .
    ]
{{< /codeblock >}}

Here we've introduced two new concepts in the manifest, the `build-options` and `build-args`
options. Flatpak attempts to sandbox your application every step of the way. It explicitly makes
it harder to access resources on your host machine like your filesystem or the network. So if
we want to access the network during the build step, we have to tell it to allow use (hence
`--share=network`).

Before we rebuild the application, we'll want to update our `hello.py` to use our new package

{{< codeblock "hello.py" python >}}
import requests

def main():
    print(requests.get('https://google.com').text)


if __name__ == "__main__":
    main()
{{< /codeblock >}}


This code will not be able to execute successfully yet. If you try to build and run this
program now, (go ahead I'll wait). You'll end up with an error like this:

{{< codeblock "Network error" python >}}
Traceback (most recent call last):
  File "/app/lib/python3.6/site-packages/urllib3/connection.py", line 141, in _new_conn
    (self.host, self.port), self.timeout, **extra_kw)
  File "/app/lib/python3.6/site-packages/urllib3/util/connection.py", line 60, in create_connection
    for res in socket.getaddrinfo(host, port, family, socket.SOCK_STREAM):
  File "/app/lib/python3.6/socket.py", line 745, in getaddrinfo
    for res in _socket.getaddrinfo(host, port, family, type, proto, flags):
socket.gaierror: [Errno -3] Temporary failure in name resolution_

{{< /codeblock >}}

This is because, just like our build steps, our application is running in a sandbox. It cannot
talk to the network unless we allow it. Our previous `--share=network` only applied to the
`pip-install` module. If we want our application to have access to the network, we need to set
a new option in the manifest (noticing a pattern?)

{{< codeblock "org.flatpak.Hello.json" json >}}
    "command": "runner.sh",
    "finish-args": [
      "--share=network"
    ],
    "modules": [
      . . .
    ]
{{< /codeblock >}}

Here we've added the `finish-args` option. In it, we specify that we want to share the host
network. This will now allow us to perform a network request google and see that wonderful HTML,
CSS and JavaScript of Google's homepage.

{{< codeblock "Run Hello Google from Python" bash >}}
flatpak-builder --force-clean build-dir org.flatpak.Hello.json
flatpak-builder --run build-dir org.flatpak.Hello.json runner.sh
{{< /codeblock >}}

# Packaging Your Package

Now that you know how to install python, run a python script, and install dependencies, you're
ready to apply the flatpak build process to your favorite Python Package and deploy it via
flatpak. This is the easiest way I've found to do that.  Let's say that I have a python package
with a directory structure like the following:

{{< codeblock "hello package" bash >}}
hello/
    | setup.py
    + hello/
        | - __init__.py
        | - __main__.py
{{< /codeblock >}}

To achive a directory structure like this, we'll first have to create a `setup.py` file:

{{< codeblock "setup.py" python >}}
from setuptools import setup, find_packages

setup(
    name='hello',
    version='0.0.1',
    description='A package that hits google',
    author='Logan Asher Jones',
    author_email='loganasherjones@gmail.com',
    packages=find_packages(),
    classifiers=[
        'Development Status :: 1 - Planning',
    ],
    install_requires=[
        'requests',
    ],
)

{{< /codeblock >}}

Then move our old `hello.py` to `hello/__main__.py`

{{< codeblock "setup package directory" bash >}}
mkdir -p hello
touch hello/__init__.py
cp hello.py hello/__main__.py
{{< /codeblock >}}

A very simple hello package with dependencies listed in the `setup.py`. We are now going to add
this to our sandbox and install all our dependencies. Time for a new module! Replace the old
`pip-install` module with this new `app-install` module.

{{< codeblock "org.flatpak.Hello.json" json >}}
    "modules": [
      . . .
      {
        "name": "app-install",
        "buildsystem": "simple",
        "build-options": {
          "build-args": [
            "--share=network"
          ]
        },
        "build-commands": [
          "install -D setup.py /app/package/setup.py",
          "cp -r hello/ /app/package/hello",
          "pip3 install /app/package"
        ],
        "sources": [
          {
            "type": "file",
            "path": "setup.py"
          },
          {
            "type": "dir",
            "path": "hello/",
            "dest": "hello/"
          }
        ]
      }
      . . .
    ]
{{< /codeblock >}}

Hopefully this all makes sense. We've introduced a new source type (which is currently undocumented)
called `dir`. This allows us to copy the `hello` directory in its entirety into the sandbox, then
we simply copy the setup.py into the same directory and point pip at it! With a quick modification
to our `runner.sh` script, we'll have a package installed and running:

{{< codeblock "runner.sh" bash >}}
    #!/bin/sh
    export PYTHONPATH="/app/packages/hello"

    python3 -m hello
{{< /codeblock >}}

We will build and run it one more time:

{{< codeblock "Run your first python package" bash >}}
flatpak-builder --force-clean build-dir org.flatpak.Hello.json
flatpak-builder --run build-dir org.flatpak.Hello.json runner.sh
{{< /codeblock >}}

# Conclusion

Hopefully this was helpful for you getting started in flatpak. It is a very interesting technology
and I'm looking forward to see where it goes from here. In the meantime, I plan on taking what I've
learned and applying it to the briefcase project, so you can just run `python setup.py linux` on
your next Native application and get a nice linux distribution.

Everything that I did can be found in my [GitHub Repository for flatpak](https://github.com/loganasherjones/flatpak-python)

As always you can reach me on twitter [@loganasherjones](https://twitter.com/LoganAsherJones) if
you have questions. Otherwise post a comment on Disqus. Thanks so much for reading!

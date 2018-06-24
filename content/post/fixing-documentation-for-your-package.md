---
title: "Correcting Documentation for a Deployed Python Package"
date: 2018-06-24
categories:
  - python
  - package
  - documentation
tags:
  - python
  - development
  - packaging
  - package
  - deploy
keywords:
- python
- development
- packaging
- package
- deploy
thumbnailImagePosition: right
thumbnailImage: ./images/four.jpg
coverImage: ../../../images/four.jpg
coverCaption: Photo Credit https://www.katiehowellphotography.com/
showMeta: false
draft: false
---

A clever way to release new documentation without releasing a new package that might confuse
your user base.

<!--more-->

# The Problem

After my newest release of [yapconf](https://github.com/loganasherjones/yapconf) it was brought to
my attention that the documentation didn't look very good on PyPi. You can
[see for yourself.](https://pypi.org/project/yapconf/0.3.2/) This made me very sad, as I had worked
quite hard on the documentation.

It looked like I was missing some dashes underneath a header. Alas, I did not want to cut a brand
new release just to make PyPi look better. After all, it was rendered correctly on GitHub and my
readthedocs page also looked pretty good. Still, this irked me, so I
[took my lamenting to twitter](https://twitter.com/LoganAsherJones/status/1008873842259963907).

Luckily, the Python community is absolutely amazing. I had the pleasure of meeting
[Brett Cannon,](https://snarky.ca/) one of the Python core developers (and a key-note speaker at
[PyCon 2018](https://us.pycon.org/2018/about/)). Luckily for me Brett had a solution that seemed to
solve my problems.

# Preventing the Problem

First, there was a library that could have helped me. [@btskinn](https://twitter.com/btskinn) on
twitter pointed me to a library called [restview](https://pypi.org/project/restview/) which was
exactly what I needed. Using it couldn't be simpler:

{{< codeblock "Install and run restview" Shell >}}
pip install restview
restview --long-description
{{< /codeblock >}}

This let me see exactly what was wrong with my text. I got something like this in my browser:

{{< alert danger >}}
\<string\>:146: (WARNING/2) Title underline too short.

0.3.1 (2018-06-07)

\-\-\-\-\-\-
{{< /alert >}}

If I had used restview from the beginning, I wouldn't have been in this pickle to begin with.
I would have known there was a problem before I released, and fixed it.

# Solving the Problem

Sadly, I did not know what the problem was before I released, and I imagine I could miss it again.
So I edited my `HISTORY.rst` and now it looked correct. No more warnings anywhere. I can now use
a versioning technique described in [PEP-0440](https://www.python.org/dev/peps/pep-0440/) called
[post relases](https://www.python.org/dev/peps/pep-0440/#post-releases).

The concept is very simple. You can add `.postN` to the end of your semantic version for your
package. This will indicate to end users that this change has no actual bug fixes and instead are
just fixes related to minor errors in, for example, a `HISTORY.rst`

I just updated my version number `0.3.2` => `0.3.2.post1` and ran `make release` and boom!
[The documentation is fixed](https://pypi.org/project/yapconf/).


# The conclusion

I learned some valuable lessons:

1. Twitter can be quite helpful.
2. The Python Community is awesome.
3. restview is a very helpful library.
4. The post release version is meant for this problem exactly.

Thanks for reading! If you have a better way to do it, feel free to comment or hit me up on
twitter [@loganasherjones](https://twitter.com/LoganAsherJones)

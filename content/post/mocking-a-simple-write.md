---
title: "Mocking a Simple Write"
date: 2018-06-11
categories:
  - python
  - testing
  - mock
tags:
  - python
  - development
  - testing
  - mock
  - write
keywords:
  - python
  - development
  - testing
  - mock
  - write
thumbnailImagePosition: left
thumbnailImage: ./images/third.jpg
coverImage: ../../../images/third.jpg
coverCaption: Photo Credit https://www.katiehowellphotography.com/
showMeta: false
draft: false
---

A short tutorial showing you a clever way to mock out some writes during your unit testing.

<!--more-->

# The Problem

I had a [small feature](https://github.com/loganasherjones/yapconf/issues/78) in my open-source
project [yapconf](https://github.com/loganasherjones/yapconf) that I wanted to solve.
Basically, I wanted to confirm that what I wrote to `STDOUT` was what I expected to write. So I
turned to the [Mock](https://docs.python.org/3/library/unittest.mock.html) library for python.

Here is the basic code I wanted to test (some parts are omitted for brevity)

{{< codeblock "yapconf/__init__.py" python >}}
def _dump(data):
    if isinstance(data, Box):
        data = data.to_dict()

    yaml.dump(data, sys.stdout, **kwargs)

{{< /codeblock >}}

Pretty simple code, we are just converting the Box over to a dictionary before dumping. You can
see that we should be streaming the result out to `sys.stdout`

# The solution

I wanted to test this code was actually going to write out what I thought it was going to write
out. After all, that's why I was writing this new feature. So let's create a unit test:

{{< codeblock "dump_test.py" python >}}
@pytest.fixture
def ascii_data():
  return {
    'db': {'name': 'db_name', 'port': 123},
    'foo': 'bar',
    'items': [1, 2, 3],
  }

def test_dump_box(ascii_data):

    # Expected string to be written. We use os.linesep
    # so that this test works cross-platform.
    expected = (
        'db:{n}'
        '  name: db_name{n}'
        '  port: 123{n}'
        'foo: bar{n}'
        'items:{n}'
        '- 1{n}'
        '- 2{n}'
        '- 3{n}'
    ).format(n=os.linesep)

    # Create a data structure so mock_write can modify it.
    data = {'writes': ''}

    # A simple mock
    def mock_write(s):
        data['writes'] += s


    # Use patch to mock out the .write call with our simple version.
    with patch('sys.stdout.write', mock_write):
        yapconf.dump(Box(ascii_data))

    assert data['writes'] == expected

{{< /codeblock >}}

# The explanation

The code above uses a [pytest fixture](https://docs.pytest.org/en/latest/fixture.html). If you
haven't heard of either, a highly suggest you go check them out. Either way, we are taking in
`ascii_data` into our test function `test_dump_box`.

At the beginning of the function, we construct the string we expect to receive. In this case, we
are dumping yaml, so we would expect to see a yaml formatted string. We also use the `.format`
to make sure that our line separators will work cross-platform.

Next, we create a `data` object that is initialized at the function scope. We create a new
function called `mock_write` that will modify this `data` variable. That way we get a pseudo-global
variable that we can play with.

Next, we use `patch` to actually mock out the `sys.stdout.write` call. The next argument to
`patch` is the new method we are sending it to.

Finally, we call our `dump` and `assert` that the value is correct.

# The conclusion

Thanks for reading! If you have a better way to do it, feel free to comment or hit me up on
twitter [@loganasherjones](https://twitter.com/LoganAsherJones)

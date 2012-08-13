## A Whistlestop Tour of Python Requests

Return to our regularly scheduled technical blogging, I'm going to give a
quick overview of one of the software libraries I know best: 
[Kenneth Reitz](http://kennethreitz.com/)'s
[Requests](http://docs.python-requests.org/en/latest/index.html) library for
the Python programming language. Since I started making minor (and I mean
[really minor](https://github.com/kennethreitz/requests/commit/f72c13ffdad47ff684d102b6d9fb6122db899891))
contributions to the library, I've become increasingly familiar with its use
and utility, and I now firmly believe that it is the best third-party library
available for Python.

The documentation is in general good: the bad parts tend to be the parts I
wrote. Nevertheless, it's worth having an informal introduction to the library
somewhere on the internet. I'm actually pretty sure other sources exist, but
this is one of the very few things I can write about with even a modicum of
legitimate authority, so damnit I'm going to write it anyway.

Be warned: Requests provides a great many features that I'm not going to go
into in this post. The library has developed from being a useful wrapper over
some of Python's less [Pythonic](http://www.python.org/dev/peps/pep-0020/)
libraries (oh, hello there [urllib2](http://docs.python.org/library/urllib2)
and [httplib](http://docs.python.org/library/httplib.html)) into the most
intuitive method of interacting with HTTP and HTTPS available in any
programming language I have ever seen. The intention of this blog post is to
quickly highlight the utility of Requests, not its depth. As always, the
best place to look is the [official docs](http://docs.python-requests.org/en/latest/index.html).

### What is Requests?

**tl;dr:** Requests is a Python library intended to make it as easy as
possible to work with [HTTP](http://en.wikipedia.org/wiki/HTTP).

More fully, Requests is intended to provide a simple and intuitive API that
allows the creation of complex workflows from Python code. This library is,
in my opinion, the holy grail of
[API-Oriented Design](http://lukasa.co.uk/2012/07/API_Oriented_Design/). For
the maintainer, any addition to Requests that makes the API any less clear is
a bad change.

This means that Requests has the ability to scale naturally to the complexity
of your problem. If all you want to do is grab some
[JSON](http://en.wikipedia.org/wiki/JSON) from a single web URL, Requests will
make that a one-line program. If you want to manage a complicated workflow
using [OAuth](http://en.wikipedia.org/wiki/OAuth) to grab data from multiple
sources, Requests provides you with all the tools you need to handle the data
exchange.

Any Python developer who has to interact with the web as a client will
eventually find their way to Requests, and when they do they'll have stumbled
onto the single best Python library around.

### Let's Start!

Ok, let's get going. Firstly, you want to create a sandbox in which you can
play. It is rarely a good idea to pollute your system Python installation with
any package installation, so you should probably create a useful
[`virtualenv`](http://www.virtualenv.org/en/latest/index.html). If there was
any package to install into your site packages, Requests would be it, but it
then becomes something you need to maintain. As a result, I recommend you
create a `virtualenv` for following through this little tour. Do this like
so:

    $ virtualenv venv
    $ source venv/bin/activate

You should get a little `(venv)` at the start of your prompt. This lets you
know that you're using the `virtualenv`. When that's done, install Requests
into the `virtualenv`:

    $ pip install requests

Requests is present. Grab yourself an interactive Python shell (`python` at
the command prompt). Now you can begin to see some of the glory of Requests.
Let's see what we need to do to get Google's homepage.

    >>> import requests
    >>> r = requests.get('http://www.google.com/')

And we're done.

Seriously, that's it. The variable `r` now contains a `Response` object, which
contains everything associated with the response from the web server. We can
take a look at what we got.

    >>> print r.status_code
    200
    >>> print r.reason
    OK
    >>> print r.text
    <!doctype html><html>...
    ...snip...
    ...</html>

As you can see, you can find out the HTTP status code returned by the server,
and also the human-readable response code. In this case, because the URL I
used points to a real resource, the server has returned a 200 OK response.

We can make other forms of HTTP requests too. POST, DELETE, PUT, OPTIONS,
HEAD and PATCH can all be made using the exact same syntax: `requests.post()`,
`requests.delete()`, `requests.put()`, `requests.options()`,
`requests.head()`, `requests.patch()`.

Knowing that the resource I wanted has been returned, I can access it using
the `text` method of the `Response` object.

#### Getting Data From a Response

The `text` method is one of a few methods of getting the content of a
`Response`. To demonstrate the others, the most effective way is to use a
resource that returns JSON, for which Requests has built-in support.

A good resource of this type is Twitter. Let's find the top 20 trending topics
for each hour of today. To do that, run:

    >>> r = requests.get('http://api.twitter.com/1/trends/daily.json')
    >>> print r.status_code
    200

In Requests you can get the content of a `Response` in four ways. For now,
we'll focus on three. The first is to get the content as a stream of bytes,
which in Python is represented by a standard string (Python 2.x) or a Python
bytestring (Python 3.x). This is the `content` method:

    >>> print r.content
    ...snip...

The next option is the preferred method for non-JSON requests, which is to use
the `text` method. This is because
[encoding text is hard](http://www.joelonsoftware.com/articles/Unicode.html).
It's highly likely that your web server has returned data that is not encoded
in ASCII. This means the text needs to be decoded first. Requests will look at
the headers of the response and use them to determine the encoding used. We
can see this clearly in our response from Twitter. First, take a look at the
response headers.

    >>> print r.headers['Content-Type']
    application/json; charset=utf-8

Requests looks for that Content-Type header, in particular for the charset
field. If that is present, Requests assumes that the encoding specified is the
one being used. We can see that:

    >>> print r.encoding
    utf-8

If the header isn't present (and
[a specific other header isn't there either](http://docs.python-requests.org/en/latest/user/advanced/#encodings)),
Requests uses [chardet](http://pypi.python.org/pypi/chardet) to try to guess
the encoding. If it turns out that Requests gets it wrong, you can also *set*
the `encoding` property to the correct value.

Regardless, when you call `Response.text`, you will get the content of the
response *decoded using that codec*. This happens behind the scenes, so that
in 99.99% of cases everything will be seamless.

    >>> print r.text
    ...unicode snip...

The final method works well when you have JSON, and that is to use the `json`
method. This method returns a Python object (basically a dict) that contains
the JSON from the response, exactly like you'd get if you called
[`json.loads()`](http://docs.python.org/library/json.html) on the content of
the response. This dict can then be immediately interacted with like any other
Python dictionary.

    >>> print r.json
    ...dict snip...

#### Cookies and Headers and Such!

Requests makes it really easy to get at the headers and cookies sent in a
Response. The headers are stored in objects that behave almost exactly like
Python dictionaries. This means you can print the entire thing, like this:

    >>> print r.headers
    ...dict snip...

Both the headers and cookies support access like Python dictionaries. This
means you can do this:

    >>> print r.headers['cache-control']
    max-age=300, must-revalidate
    >>> print r.cookies['_twitter_sess']
    ...long alphanumeric string here...

You can also send custom headers and cookies by using Python dictionaries. For
instance, you can do this:

    >>> headers = {'User-Agent': 'not-a-real-browser (like Gecko)'}
    >>> cookies = {'Tasty': 'Chocolate_chip_cookies'}
    >>> t = requests.get('http://httpbin.org/get', headers=headers, cookies=cookies)

Of course, cookies aren't really of much use when you only send them on one
request. The whole point is that cookies enable you to maintain state, and
Requests' API appears to be stateless. Obviously, Requests makes it easy to
maintain state.

#### State in Requests

To maintain state, Requests provides you with the `Session` class. Once you
create an instance of the class, you can make all of you requests using
methods on the `Session` instance. This looks like this:

    >>> s = requests.Session()
    >>> s.get('http://www.google.com/')
    >>> s.close()

There are two main reasons to use sessions. The first is to maintain state
over a series of connections. If you make subsequent GETs or POSTs or
what-have-you, Requests will continue to pass the cookies around, which is
exactly what you want to have happen.

The next reason is that the `Session` class allows you to set some
parameters that will be used for all subsequent requests. This includes
headers and cookies, as well as some of the other options I won't be going
into this time around.

As you can see in the code example above, sessions need to be instantiated
and cleaned up. It's good form, then, to use sessions as context managers,
like so:

    >>> with requests.Session(headers=headers) as s:
    ...    s.get('http://www.google.com/')
    ...    s.get('http://www.example.com/')
    >>> do_some_other_stuff()

This saves you having to call `Session.close()` on your session, and ensures
that you don't leak sockets. It's also very Pythonic, and that's always a good
thing.

#### Query Parameters

A common feature of web APIs and URLs in general is the use of query
parameters. These are the portions of the URL that follow the leading '?'
character. Requests makes it easy to add these into the URL without having
to form it correctly yourself:

    >>> params = {'hi': 'there'}
    >>> r = requests.get('http://httpbin.org/get', params=params}
    >>> print r.url
    http://httpbin.org/get?hi=there

These parameters are placed into the URL and correctly encoded. Passing
parameters in this way ensures that there will not be difficulties passing
them to APIs upstream.

### So much more!

This was the most brief look at the features available in Requests. There are
so many other features: proxies, automatic HTTP authentication, automatic
redirect handlers, multipart file upload, SSL certificate verification and
more. If I can find a good example, I'll walk through some of the more
advanced features of the library in the future.

In the meantime, I thoroughly suggest that if you have any need to
programmatically access web services, whether an API or in some web-scraping,
you try out Python's Requests library. It's truly an exemplar in the world of
programming libraries.

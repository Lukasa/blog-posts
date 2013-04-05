## Requests' Two APIs

[Kenneth Reitz](http://kennethreitz.org/)'s excellent
[Requests](http://docs.python-requests.org/en/latest/) library has been
praised, rightfully, for its excellent API. In fact, its API is so good that
it's been praised in a
[literary context](http://thediagram.com/12_6/rev_reitz.html), as well as by
almost every programmer who has come across it. There is no question that this
API is one of the best you can find in the Python world. It has even inspired
[other programmers' API design](https://github.com/sigmavirus24/github3.py).

What a lot of people seem not to know, however, is that Requests actually has
_two_ APIs. The first API is the highly-praised procedural and object-oriented
API that drives making and interacting with HTTP traffic. The second API
appeared when Kenneth undertook the
[giant refactor for v1.0](http://kennethreitz.org/exposures/announcing-requests-v1-0-0).
This is Requests' barely heralded inheritance-based API.

### Requests' Inheritance API

To discuss the second, _super secret_ Requests API we need to draw one very
import distinction. This distinction is between properties of a single HTTP
request (things like data, headers and cookies) and things that are properties
of the connection (things like the kinds of SSL you can use, or what IP address
you send data from).

Requests by and large exposes the first kind of property on its 'standard' API.
You can set the headers, and the body data, and the cookies, all through the
arguments to the Request object and as properties of the Session object. This
makes sense: this is the kind of thing that is really specific to a single
request or session of requests.

But what about the second kind of thing? Where do you set that? Some of the
most-used stuff bubbles its way up to the main API so that it's easily
accessible, but most of it is hardcoded.

I can hear some of you now. "_Kenneth hardcoded these options? How awful!_"
In the Requests versions v0.14.2 and earlier, I'd have probably agreed with
you. They aren't options people want often, but it would be good to have access
to them.

The problem is: how do you give access to these options without cluttering the
main API so much your documentation looks
[like this](http://docs.python.org/2/library/json.html#json.dump)?

#### Configuration Through Inheritance

If you look carefully at the Requests code, you'll notice that there are two
major objects involved in each request. The first is the Requests Session
object (`requests.Session`). The second is the Requests default
[Transport Adapter](http://kennethreitz.org/exposures/the-future-of-python-http),
the HTTPAdapter (`requests.adapters.HTTPAdapter`). These two objects
maintain all of the configuration in Requests.

The Session object is almost entirely concerned with the first kind of
property. You can set default headers and default cookies, and that all goes
really well. But what if you want to handle the second kind? The answer is
surprisingly simple: subclass the HTTPAdapter.

The HTTPAdapter manages all of the connection-level behaviour, and doesn't
expose much in the way of options on its constructor. The real way to affect
this behaviour is to subclass it and create your own.

#### Examples

I've shown this configuration
[once before on this blog](//lukasa.co.uk/2013/01/Choosing_SSL_Version_In_Requests/),
but I want to provide a few examples of what I mean. The first is one I've
shown before: selecting an SSL version.

    from requests.adapters import HTTPAdapter
    from requests.packages.urllib3.poolmanager import PoolManager

    class SSLAdapter(HTTPAdapter):
        '''An HTTPS transport adapter that uses an arbitrary SSL version.'''
        def __init__(self, ssl_version=None, **kwargs):
            self.ssl_version = ssl_version
            super(SSLAdapter, self).__init__(**kwargs)

        def init_poolmanaager(self, connections, maxsize):
            self.poolmanager = PoolManager(num_pools=connections,
                                           maxsize=maxsize,
                                           ssl_version=self.ssl_version)

You can unconditionally sign requests:

    from requests.adapters import HTTPAdapter
    from requests.packages.urllib3.poolmanager

    class SigningAdapter(HTTPAdapter):
        """This adapter signs a request using some unknown method."""
        def send(self, request, **kwargs):
            signature = signing_function(request.body)
            request.headers['Signature'] = signature

            super(SigningAdapter, self).send(request, **kwargs)

You can even support
[totally different protocols](//lukasa.co.uk/2012/12/Writing_A_Transport_Adapter/)
(though having done that myself, I don't entirely recommend it, but it _can_
be done).

#### Why This Works

The reason this works is because Requests allows you to configure the backend,
using `Session.mount()`. Before you could do this, anything that was not
contained in the Session object was totally inaccessible to users. This meant
if you wanted to change the sending logic, you had to subclass the Session and
get your hands dirty in the 2000-line `send` function.

Now, you get to subclass the clearly laid out `HTTPAdapter` and have to
rewrite as little as possible. More people need to know about this and use it,
because it's the best way to really take control of Requests.

Requests has never been more modular or powerful, and it's API is just as clean
as it was when it was born. Take advantage of this secondary API, and you'll
find that you can configure Requests just as easily as you can drive it.

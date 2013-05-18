## Caching In Python Requests

I think I've made it clear in the past that I think
[Requests](http://python-requests.org/) is awesome. At this stage it's become
a mature, feature-filled library that is more than capable of replacing urllib2
and friends in almost every situation you might be interested in. There are
very few things that urllib2 can do that Requests can't do, and Requests is
almost always capable of doing it better.

However, one of things urllib2 can do that Requests _can't_ do (out of the box)
is caching. This is a shame, since HTTP caching has effectively made the modern
internet.

Writing your own HTTP cache isn't really very hard:
[RFC 2616 is pretty clear about how it works](http://pretty-rfc.herokuapp.com/RFC2616#caching)
and there isn't actually that much functionality. You could make pretty major
gains just by supporting the Cache-Control header, and realistically that
doesn't take much work at all.

That's a hassle, though. Requests gives you so much: why should you have to do
this yourself? What you really need is a Requests plugin that makes HTTP
caching 'just work'.

You'll never guess what I've been doing.

### httpcache: Caching For Requests

You want caching? You're using Requests? Here's how you get caching with
minimal work. Crack out your command prompt and install my brand new module,
[httpcache](http://httpcache.readthedocs.org/en/latest/):

    $ pip install httpcache

Once you've installed it, you get caching like this:

    import requests
    from httpcache import CachingHTTPAdapter

    s = requests.Session()
    s.mount('http://', CachingHTTPAdapter())
    s.mount('http://', CachingHTTPAdapter())

Done.

Really, I mean done. Everything just works. All your HTTP traffic passes
through my caching adapter which handles all the busy work. You can just sit
there and reap the benefits of decreased bandwidth usage and shorter latency.

### What Do You Get?

Using httpcache gives you lots of things. Here are the highlights:

- Tight integration with Requests. Plugs in and just works.
- Cache-Control headers are understood, in all their complicated glory.
- Expires headers are understood, in all their HTTP/1.0 retroness.
- Responses to non-idempotent messages aren't cached.
- Non-idempotent messages invalidate cached responses, as per RFC 2616.
- Performs validation caching: If-Modified-Since headers and HTTP 304
  responses.

All of this, and you don't have to do a thing. The life of a Python programmer
is pretty awesome sometimes.

### I Know What's Best For You

Like Requests, httpcache has some very strong opinions about 'the right thing'.
For that reason, you don't get much in the way of configuration options. In
fact, you get exactly one: the cache capacity. You know, so you don't run out
of memory and fall over.

### Contributions And Further Work

I haven't used httpcache industrially yet, which means there might be some bugs
that I don't know about. For that reason, it's got a very early version number:
v0.1.2 at the time of writing. I would love for you to use it, and I would love
for you to report any bugs you find. You could even write some code to fix them
if that floats your boat!

Fork [the Github repo](https://github.com/Lukasa/httpcache) and get going.
Those bugs won't find themselves!

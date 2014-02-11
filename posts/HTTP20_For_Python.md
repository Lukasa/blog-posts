## HTTP/2.0 For Python

The HTTPbis have spoken, HTTP/2.0 is happening. Major websites are beginning to
adopt it (hello there Twitter!), and the spec is beginning to get nailed down.
If you're unfamiliar with HTTP/2.0 and all the fun things it brings you, Ilya
Grigorik has been doing an excellent job evangelising for it. Take a look at
[this talk](http://youtu.be/ZxfEcqJ4MOM) if you want a deep dive into HTTP/2.0.
If you want a shorter discussion,
[this article](http://queue.acm.org/detail.cfm?id=2555617) is also a great
reference.

HTTP/2.0 brings a number of advantages: it saves time setting up and tearing
down TCP connections, it compresses HTTP headers, it provides flow control and
connection multiplexing. For certain applications in certain use-cases, these
can represent a huge advantage over HTTP/1.1.

It's time for Python to get support for it. With that in mind, I've
spent the last few weeks working on my brand new project. I give you:
[`hyper`](http://hyper.rtfd.org/).

### Hyper

Hyper is an attempt to build a pure-Python HTTP/2.0 stack that perfectly
matches the `http.client` API from Python's standard library. This will allow
`hyper` to be a drop-in replacement for `http.client`, providing all the
HTTP/2.0 goodness to all Python programs without the need for potentially
painful rewrites.

How does it work? Like this:

    from hyper import HTTP20Connection

    conn = HTTP20Connection('twitter.com:443')
    conn.request('GET', '/')
    resp = conn.getresponse()

    print(resp.read())

That's all it takes. The same API as `http.client`, but with all the fun of
HTTP/2.0. For more examples, see [the documentation](http://hyper.rtfd.org/).
If you're really excited, just go right ahead and download it using `pip`.
Alternatively, take a look at the code
[on GitHub](https://github.com/lukasa/hyper): it's available under the MIT
license, so everyone should be able to grab it and play with it.

### Features

At the moment, `hyper` is in a very early alpha (version 0.0.1). I've proved
that the mainline use-case works fine, but I'm sure there are plenty of bugs
lying around, and there are several features I'd like to implement that I
haven't.

Nevertheless, the core feature set is already in place:

- Can make HTTP/2.0 requests.
- Obeys the HTTP/2.0 flow-control mechanisms.
- Verifies TLS certificates.
- Streaming uploads.
- Streaming downloads.

There are plenty of features planned, with the headline one being transparent
HTTP/2.0 and HTTP/1.1 interworking. Currently, `hyper` will only work against
servers that fully support HTTP/2.0. I'm planning to implement an abstraction
layer that should prevent the user from needing to know whether the server
they're contacting supports HTTP/2.0: `hyper` will transparently fall back to
HTTP/1.1.

The only feature I have no intention of implementing support for is one of
HTTP/2.0's headline features: Server Push. I'll discuss this omission later.

### Warning

Please note that `hyper` is in a very early alpha. I do not recommend you use
it in production: things _will_ go wrong. However, I'd like as many of you as
possible to take it out for a spin and report any bugs you find on the
[GitHub page](https://github.com/Lukasa/hyper). The more bugs you find the
better `hyper` will get.

### Restrictions

Currently, `hyper` requires Python 3.3 or higher. This unfortunate restriction
is caused by the fact that earlier versions of the standard library's `ssl`
module are totally braindead and do not support the TLS
Next-Protocol-Negotiation extension, which HTTP/2.0 requires. Until I can find
a good way around this limitation, `hyper` will remain limited to at least
Python 3.3. If you don't like this (I certainly don't), I encourage you to
take your complaint up with the Python core development team.

Additionally, you currently need to find out whether your target server
supports HTTP/2.0 or not before you point `hyper` at it, or it'll raise an
`AssertionError`. This is a necessary guard in this early release, but this
will be improved in future releases.

Sadly, `hyper` is also currently inherently single-threaded. This means it has
some _unusual_ behaviours, but also that you need to implement your own
threading mechanism on top of it if you want to use it in multi-threaded code.
A suggested implementation is sketched out in the documentation, and I hope to
provide example code in the near future. More long-term, `hyper` will become
thread-safe and you'll be able to simply have global connections.

#### The Omission of Server Push

I should briefly touch on the omission of Server Push from `hyper`. I've been
on record in the past as saying that Server Push is fundamentally incompatible
with the way most Python HTTP code is written. All the big Python HTTP client
libraries are built primarily to be used from synchronous code: they expect to
send a request and then block until the response comes back. They absolutely do
_not_ expect to receive multiple responses to the same request, and provide no
API for receiving these responses.

I don't believe it's possible to write a pleasant API that correctly handles
Server Push without using either an event loop or having `hyper` spawn its own
background threads. Until `asyncio` becomes a standard, I don't think `hyper`
should force you to do the first, and it's obnoxious for a library to spawn its
own threads. This means that, for the moment, `hyper` does not support Server
Push. I do not believe this to be a big loss for 90% of the potential users of
`hyper`, so I judge it to be an acceptable limitation. If you strongly
disagree, please get in touch so we can talk about why you see this issue
differently to me.

### Development

`hyper` is in early alpha and under active development
[on GitHub](https://github.com/Lukasa/hyper). I'd love it if people took the
time to download and try out `hyper`. I'm particularly interested to see what
kinds of bugs people hit: you'll definitely hit them, and a detailed bug report
would be really phenomenal.

All contributions are welcomed. Even if you try out `hyper` and find no bugs,
drop me a line on [Twitter](https://twitter.com/Lukasaoz) and let me know that
you used it: at this early stage I need all the feedback I can get.

Finally, I'd like to specially call out
[Sriram Ganesan](https://github.com/elricL), who graciously volunteered some of
his time to work on `hyper` when it didn't work at all. He provided some useful
infrastructure when I was too busy to do it myself, and he made my work
immeasurably easier: thanks so much Sriram!

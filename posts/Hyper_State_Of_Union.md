## Hyper: The State of the Union

It's been a few months since I last spoke about
[`hyper`](http://hyper.rtfd.org/). The reason I've been so quiet is primarily
because not much progress has been made on the project. I've been swamped with
other projects (most notably [Project Calico](http://www.projectcalico.org/)),
and that's limited my ability to make much forward progress.

However, in the last few weeks there's been a bit more activity on `hyper`, in
large part due to the excellent work of
[Tetsuya Morimoto](https://github.com/t2y), who has provided a couple of
desperately needed bug fixes. If you don't want to read the rest of this post
and just want the goodness, you can download v0.2.0 of `hyper` from PyPI
*right now*. Go get it!

In addition to those bug fixes, however, he provided the impetus for the most
exciting new 'feature' of `hyper`, which isn't actually a feature of `hyper` at
all.

### Big News!

The highlight is this: **as of today you can get the awesome
[HTTPie](http://httpie.org/) to issue requests over HTTP/2, using `hyper`**.
This development takes advantage of HTTPie's very cool plugin architecture to
allow for all HTTPS requests to be sent over HTTP/2 instead.

Here's an example:

    lukasa@mbp:hyper/ % http https://nghttp2.org/httpbin/get
    HTTP/2.0 200
    access-control-allow-credentials: true
    access-control-allow-origin: *
    content-length: 271
    content-type: application/json
    date: Sat, 07 Feb 2015 12:57:57 GMT
    server: nghttpx nghttp2/0.7.4-DEV
    strict-transport-security: max-age=31536000
    via: 1.1 nghttpx

    {
        "args": {},
        "headers": {
            "Accept": "*/*",
            "Accept-Encoding": "gzip, deflate",
            "Connection": "close",
            "Host": "httpbin",
            "User-Agent": "HTTPie/1.0.0-dev",
            "Via": "2.0 nghttpx"
        },
        "origin": "81.156.148.41",
        "url": "https://httpbin/get"
    }

Notice the HTTP/2.0 return code, and the fact that this request went to a copy
of [httpbin](http://httpbin.org/) that runs behind the nghttp2 HTTP/2 server.

This is hugely exciting. The above very simple web requests represents a great
bit of collaborative open-source effort, involving the following tools:

- hyper
- requests
- HTTPie
- httpbin
- nghttp2

I'd like to thank the contributors and maintainers of all those tools for this
enormously exciting moment. `hyper` required relatively little work to slot in
to HTTPie, which is a testament both to HTTPie and to `hyper`.

If you want to try this out, you can follow the instructions on
[this page](https://github.com/jakubroztocil/httpie-http2) to get going. Note
that you should definitely play around with this in a virtual environment, as
it'll break any HTTPS-encrypted HTTP/1.1 connections you want to make. Best not
to break your normal HTTPie install!

### More Big News

The other big news is that `hyper` now supports Python 2.7.9! Thanks to the
work done backporting the `ssl` module to Python 2.7
([PEP 466](https://www.python.org/dev/peps/pep-0466/)), the Python 2.7.9 `ssl`
module is good enough to do HTTP/2. This means that Python 2.7.9 is no longer
blocked behind the development-hell-ridden pyOpenSSL module.

So now those of you stuck on Python 2.7 can use HTTP/2 as well! The above
HTTPie news applies just as well on Python 2.7.9 as on Python 3.4.

### Exciting Times!

The summary is that `hyper` is in great shape right now, with lots of exciting
work being done. The next step is to make sure we can transparently fall back
to standard HTTPS to improve our HTTPie integration.

If you want to start working on this, then get in touch! I'd love your help.

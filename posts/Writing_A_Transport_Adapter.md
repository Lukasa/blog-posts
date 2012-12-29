## Writing A Transport Adapter

[Last post](/2012/12/Not_All_Opinions_Are_Equal/) I briefly mentioned that
[Python Requests](http://docs.python-requests.org/en/latest/) went v1.0. This
involved a huge code refactor and a few changes in the API.

One of these changes was the inclusion of something Kenneth (and others) have
been thinking about for a while as part of the
[future of Python HTTP](http://kennethreitz.org/the-future-of-python-http.html)
'project'. This something is the 'Transport Adapter'. From Kenneth's blog post:

> Transport adapters will provide a mechanism to define interaction methods
> for an "HTTP" service. They will allow you to fully mock a web service to fit
> your needs.

Well, v1.0 brought the inclusion of Transport Adapters into Requests. This is a
shiny new feature, and I wanted to take some time to explore it. The best way
to do that is to go ahead and write an adapter! As well as providing a useful
bit of experience for me, I would be able to document the process and thereby
provide a bit of a tutorial for anyone who wants to follow in my footsteps.

But what to write an adapter for? I wanted to see how far Kenneth's goal of
Transport Adapters could be stretched, so I decided to implement a transport
adapter not just for a specific web service, but for a whole new protocol: FTP.

Come with me, as I show you how to write an adapter through the lens of my own.

_For those who just want to see my work, you can see the finished product_
_[here](https://github.com/Lukasa/requests-ftp)_.

### Getting Started

The first thing to do is to work out how you implement such a thing! A quick
look at the Requests code should tell you the answer. A Transport Adapter
should be a subclass of `requests.adapters.BaseAdapter`, and should implement
three methods: `__init__()`, `send()` and `close()`. Let's take a closer look
at these.

First, `__init__()`. This will be called only once, when your adapter is
instantiated. This means that any state that should be common to all requests
belongs here. As an example, Requests' included HTTP Transport Adapter
creates a `urllib3` `ConnectionPool` object here. In my case, all I wanted to
do was allow multiple FTP verbs, so I created an ad-hoc function table keyed
off the names of the verbs. Pretty hacky, but fast.

Second, `close()`. This is also only called once, at the exact opposite
moment to `__init__()`. Any state you set up in `init()` should be torn down
here. Again, for Requests' built-in adapters, this means tearing down their
`ConnectionPool`s. For me, nothing.

Finally, `send()`. This is where the meat of your functionality will occur, so
let's take some time over it.

### Send

Glancing through the Requests code reveals that the `send()` function must, at
minimum, accept a single `PreparedRequest` object and a list of `kwargs`, and
return a `Response` object. This is where I began to run into trouble.

By default, the `PreparedRequest` object is _very_ HTTP oriented. This is by
design, and any transport adapter you write is likely to be happy with that.
However, for me it was mildly problematic. In particular, the `PreparedRequest`
contains almost none of the information on the standard `Request` object. It
carries a `url`, a `method`, its `headers`, its `body` and its `hooks`. That's
it.

You get a little more detail from the `kwargs`. You get the values of the
following Requests fields: `stream`, `timeout`, `verify`, `cert` and `proxies`.
Mostly this stuff is of minimal value, and I ended up ignoring it (though I may
go back and use `timeout`.)

If you're using HTTP, I suspect you can mostly get by with this. Byte-level
manipulation of the body is possible (and encouraged), so you can permute the
Request into whatever form is expected by your web service.

For FTP, this was harder. Mostly I wasn't interested in the data here, except
when running an FTP STOR. Here, I wanted to piggyback on Requests' current
file upload syntax. However, by the time I get hold of the `Prepared Request`,
the file has been encoded as multipart form-data. My current solution is to
use the Python CGI module to decode the data again. This is shoddy, and far and
away the worst part of the code I've written.

Returning to your code, this function has all of the responsibility for both
sending the data out and parsing it back into a useful form for Requests. You
may find, as I did, that you actually want to split this out into several
functions or methods.

When building a response, it's particularly instructive to take a look at the
function `build_response` in `requests.adapters`. This gives you an idea of the
kinds of fields Requests expects to be filled in. I ended up basing my
equivalent function very heavily on this.

### Surprises

There are a few surprises to be found. The first is that `response.content` and
`response.text` only work if you provide a file-like object as `r.raw`. I ended
up using BytesIO for this, which is a pain. If you're using `urllib3` this
should be easy for you, but it's worth knowing.

Also bear in mind that you may have to jump through some hoops to get very
Requests-like behaviour. The special case of Basic Auth as a naked tuple is
handled above the `PreparedRequest` layer, so if you want to piggyback on it
(like I did) be prepared to mess about with headers to get it to work.

### Plugging it in

Once you've got a transport adapter, all you need to do is plug it in. To do
this, use the `mount` method of the `Session` object:

    s = requests.Session()
    s.mount('http://randomservice', YourAdapter())

In my case, this was actually

    s = requests.Session()
    s.mount('ftp://', FTPAdapter())

Any subsequent Request through that session will use your transport adapter if
the URL prefix matches!

### Going Deep

For my stuff, though, I wanted to go a bit further. In particular, since I was
adding new verbs (LIST, STOR, RETR and NLST), I wanted to make the utility
methods available: the equivalents of `Session.get()` and `Session.post()`. To
do this required that I monkeypatch the `Session` object.

I didn't want to do this on import, though: the end user should have the choice
about whether they actually want me to mangle the `Session` object or not. For
this reason, I export a function: `monkeypatch_session()`. This function adds
four utility functions to the `Session` object and overrides the constructor so
that the `FTPAdapter` is automatically mounted.

### All Together Now!

When you put that all together, you get my shiny new
[library](https://github.com/Lukasa/requests-ftp). You can get this from the
cheeseshop: just run `pip install requests-ftp`. Once you've got it, you can
go straight into using it! For example, you can list the files at the root of
the directory and then get one:

    import requests
    import requests_ftp as ftp
    ftp.monkeypatch_session()

    with requests.Session() as s:
        r = s.list('ftp://127.0.0.1/', auth=('username', 'password'))
        file = r.content.split('\n')[0]
        r = s.retr('ftp://127.0.0.1/' + file, auth=('username', 'password'))
        file = open('temp.txt', 'wb')
        file.write(r.content)

### Caveats

There are lots of things my code doesn't do. A list can be found on the
[readme](https://github.com/Lukasa/requests-ftp/blob/master/README.rst). If you
feel like you want to use it anyway, feel free! I'd also welcome contributions
to resolve some of its more significant problems (like the fact it isn't
tested).

### Summing Up

Writing a transport adapter for HTTP is very easy, especially as the in-built
Requests ones have very clean and comprehensible code. Writing them for new
protocols is harder.

This is because Requests is laser-focused on HTTP, as it should be. However,
you can take my word for it: it is definitely _possible_ to implement _some_
other protocols as Transport Adapters in Requests. Whether you want to is
entirely up to you, of course.

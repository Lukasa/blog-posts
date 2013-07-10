## Python Requests And Proxies

One of [Requests](http://docs.python-requests.org/en/latest/)' most popular
features is its simple proxying support. HTTP as a protocol has very
well-defined semantics for dealing with proxies, and this has lead to
widespread deployment of HTTP proxies.

The vast majority of these proxies are 'transparent': that is, they sit on the
message path and quietly capture HTTP messages before forwarding them on. These
proxies are not a problem for people interacting with HTTP exactly because of
their transparency: you don't need to know anything about them to get your
messages through.

Many proxies however are non-transparent. The most prevalent use of this kind
of HTTP proxy is at the border between a controlled LAN and the wider internet.
In particular, companies and state institutions (e.g. schools) deploy HTTP
proxies _very_ widely. These proxies require explicit configuration on HTTP
clients because all HTTP traffic must pass through them.

The widespread nature of this kind of deployment means that Requests is
essentially obligated to support routing HTTP requests through proxies. Today
I'm going briefly to talk about how this is done, and some particular problems
we've had with the implementation.

### The Good: The API

From the perspective of the Requests user, the configuration of proxies is the
perfect combination of simple and powerful. You simply build a dictionary,
mapping URL schemes to the URL to the proxy. A proxy dictionary could look
like this:

    proxies = {'http' : 'http://10.0.0.1:8080',
               'https': 'https://10.0.0.1:4444'}

This dictionary would then get passed into the standard Requests call:

    r = requests.get('http://www.google.com/', proxies=proxies)

Voila! Your HTTP messages are now being routed through the proxy at `10.0.0.1`.
If you were using a `Session` object then you'd just configure the proxy
dictionary on the `Session`:

    s = requests.Session()
    s.proxies = proxies

No big deal, right?

### Also Good: The Requests Internals

Happily, inside Requests everything also looks pretty good. The `proxies`
parameter isn't used until it reaches
[the Transport Adapter](//lukasa.co.uk/2012/12/Writing_A_Transport_Adapter/) at
the bottom of the Requests stack. Here, it is used for three things. The first
two are simple: it can affect the URL that Requests passes to urllib3 and we
can potentially add a `Proxy-Authorization` header (in an ugly hack I'm not
entirely proud of writing). The third thing, however, is the most complex: it
affects what connection pool we use.

This is the the bit that matters most. We take great advantage of the urllib3
connection pools, and obviously all requests that pass through a proxy should
use the same connection pool: after all, they're all going to the same place.
The urllib3 connection pool used for proxies is basically the same as the
standard kind, but it'll put on a few extra headers and does a bit less sanity
checking. No big deal. Another win for code sanity!

### The Bad: HTTPS

So far so good, right? Unfortunately, this is where I tell you that the
idealised view of proxies provided above is only half the story. You see, with
the above steps, HTTP over proxies works like a charm. In fact, Requests has
had functioning proxy support over HTTP for a very long time, and it has almost
never broken. It's one of the stablest parts of the library.

However, proxying and HTTPS is a totally different story. To explain why I'm
going to walk you through a little bit of proxying in Requests.

To do that, we're going to use a tool that I consider to be a vital weapon in
the arsenal of the network programmer: [mitmproxy](http://mitmproxy.org/). The
list of sweet features in mitmproxy is as long as my arm, so I'll just direct
you to their website. In this case, we're going to abuse it as a cheap, easy to
run proxy.

We crack it out, and then get to work. First, let's pass a simple HTTP request
through it:

    >>> proxies = {'http': 'http://127.0.0.1:8080'}
    >>> r = requests.get('http://www.google.com/')

In the mitmproxy window we can see the request and response come through, no
big deal:

    GET http://www.google.com/
        <- 200 text/html 10.58kB

Awesome, so we know it works. Now, let's try to pass an HTTPS request through
it:

    >>> proxies = {'https': 'https://127.0.0.1:8080'}
    >>> r = requests.get('https://www.google.com/')
    requests.exceptions.SSLError: [Errno 1] _ssl.c:503: error:140770FC:SSL routines:SSL23_GET_SERVER_HELLO:unknown protocol

Uh-oh. What the hell happened there?

The short answer is that everything went to hell in a hand-basket, and to
understand why you need to understand what happens when you try to proxy a
HTTPS request.

#### Proxying HTTPS

The thing about HTTPS is that it relies on secure connections created using
public key cryptography. The keys for the connection are established using
cryptographically signed certificates, which are handed out by certificate
authorities (by 'handed out' I mean 'exorbitantly charged for'). In principle
these authorities (also called 'CA's) should verify the person applying for the
certificate owns the domain in question: in practice if a government comes to
them and asks really nicely, they'll usually hand over a new set of keys.

When your computer establishes an SSL connection, it begins by performing the
SSL handshake, which involves handing certificates over. This certificate is
only valid for a single domain, and _nothing else_. If your User-Agent verifies
SSL certificates (like Requests does by default), your connection will fail if
the machine you're connecting to hands over a certificate that isn't correct
for the domain.

This poses a problem for proxying HTTPS traffic. To send the message on the
proxy needs to know where it's going, but it can't find out without performing
the SSL handshake. It can't do _that_ because it doesn't have the right
certificate for the connection, so the User-Agent will terminate the
connection attempt. (Those _au fait_ with SSL/TLS will note I've simplified a
lot here, but we don't have time for the full discussion.)

The solution has been to use the HTTP CONNECT verb. The CONNECT verb
essentially turns HTTP into a tunnel over which you can send raw TCP data. This
is obviously ludicrously inefficient (TCP over HTTP over TCP), but means the
proxy can pass your handshake (and then the subsequent encrypted messages)
along without needing to be able to read them.

So what's the problem?

#### Yeah, We Don't Do That

Requests does _not_ support the CONNECT verb. At all. This is because our
underlying HTTP connection library, urllib3, also doesn't support it. There has
been an [open Pull Request](https://github.com/shazow/urllib3/pull/139) on
urllib3 for some time, but it has been essentially abandoned by its original
author and there's not been a sufficient push to get the
[rebased version](https://github.com/shazow/urllib3/pull/170) up to standards.
This is, in my opinion, the single biggest problem Requests has as a library at
the moment.

### How Do I Get HTTPS Working?

It depends. If you want HTTPS proxying without the proxy being able to read it,
I'm afraid you can't use Requests right now. This is unfortunate, but until we
can get the other Pull Request to move forward that's just where we are.

However, if you want to be able to connect to HTTPS URLs and don't care if the
proxy can read it (more fool you!), you can set up your proxies argument like
this:

    proxies = {'https': 'http://127.0.0.1:8080'}

This establishes an HTTP connection to your proxy, which should then establish
an HTTPS connection upstream. This isn't secure, but should work if you
desperately need it. Otherwise, you'll just have to wait until this gets sorted
out. Or take the initiative and sort it out yourself!

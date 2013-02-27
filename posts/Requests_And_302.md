## Requests and the HTTP 302 Status Code

I wanted briefly to touch on the behaviour of the
[Python Requests](http://python-requests.org/) library when it receives an HTTP
302 message. This has come up a couple of times on GitHub, and has usually been
considered a bug, so it's worth briefly stepping in and explaining what
Requests does and why it does it.

First, HTTP 302 is defined in
[RFC 2616](http://pretty-rfc.herokuapp.com/RFC2616#status.302). You can find
the full definition there, but I'll reproduce some of it here, with the
relevant passage highlighted (emphasis mine):

> **302 Found**
>
> The requested resource resides temporarily under a different URI. Since the
> redirection might be altered on occasion, the client SHOULD continue to use
> the Request-URI for future requests.
>
> The temporary URI SHOULD be given by the Location field in the response.
>
> **If the 302 status code is received in response to a request other than GET
> or HEAD, the user agent MUST NOT automatically redirect the request unless it
> can be confirmed by the user**, since this might change the conditions under
> which the request was issued.

OK, fine. The RFC is very clear on this point, right? It would be failing to
comply with the RFC if Requests did not prompt for user input before
redirecting after a 302.

Actually, it's not that simple. The reason is that almost all browsers do NOT
implement this behaviour and never have. Instead, they treat the 302 as an HTTP
303 See Other response: that is, they automatically and without prompting issue
a GET on the redirected-to URI.

This behaviour is _so prevalent_ that RFC 2616 contains a note (emphasis mine):

> Note: RFC 1945 and 2068 specify that the client is not allowed to change the
> method on the redirected request. However, most existing user agent
> implementations treat 302 as if it were a 303 response, performing a GET on
> the Location field-value regardless of the original request method. The
> status codes 303 and 307 have been added for servers that wish to make
> unambiguously clear which kind of reaction is expected by the client.

Hmm.

### What To Do?

So then, what should Requests do? We can't actually require the user stop and
say OK, so we would need to obtain permission beforehand. We do this by taking
the `allow_redirects` parameter on all the Request verbs. This parameter
defaults to `True`.

The way I see it, Requests has three options:

1. Follow the specification, and only redirect if `allow_redirects` is `True`.
   Additionally, use the same verb on the Location field.
2. Do what browsers do. Only redirect is `allow_redirects` is `True`. Issue a
   GET request on the Location field.
3. One of the above, but ensure that we got permission by using some different
   field, or ternary logic in `allow_redirects`. (False, True, and
   Yes-I-Really-Mean-It?)

There is no right answer here. If we do 1, we get the warm fuzzy feeling that
says 'I am doing What The Spec Says', while in the background everything fails
because web servers expect browsers and not Requests. If we do 3, we have some
horribly complicated API to ensure that we ask permission, even though we
sort-of asked permission anyway in the `allow_redirects` field.

Doing 2 is more subtle. For 99% of people, Requests works properly. Web servers
that genuinely care about what we do shouldn't be issuing 302s because they
must assume that they might get hit by a web browser, which will do The Wrong
Thing (TM). However, 1 person in 100 believes the RFCs to be sacred, and will
file bugs against Requests, insisting that we are doing The Wrong Thing (TM)
and that the world will collapse and that we'll go to hell and that Batman will
come and get us.

Ignore those guys.

### The Bottom Line

Here's the reality. Ignore
[Postel's Law](http://en.wikipedia.org/wiki/Postel%27s_law), ignore the
RFC-fascists and ignore the doubters. Instead, follow the following law:

It doesn't matter if you comply with the spec, if you don't work **out of the
box** with the majority of implementations **you are wrong**.

### For Requests

In the case of Requests, we have the following behaviour for all redirects: if
browsers automatically redirect, so do we. If browsers don't, we don't. If you
want anything different, you have to set `allow_redirects` to `False` and
handle it yourself. Sorry I'm not sorry.

## Requests 2.0

Every now and then the Requests project gets bored of fixing bugs and decides
to break a whole ton of your code. But it doesn't look good when we put it
like that, so instead we call it a 'major release' and sell it as being full of
shiny new features. Unfortunately it turns out that people complain if we break
their code and don't provide a nice way to find out what we broke.

So we provide
[a changelist](https://github.com/kennethreitz/requests/blob/master/HISTORY.rst)
with every release. The changelist is aimed at providing an easy to scan list
of the changes so that busy downstream authors can quickly identify the things
that will cause them pain and fix them. That's great, but often people want a
slightly more detailed explanation and description of what we did and why we
did it.

So this post can be viewed as a lengthier explanation of what has changed in
Requests to bring us up to 2.0. I'll tackle each change in order of their
presence in the changelist, and link to the relevant issues in Github for
people who want to see what fool convinced Kenneth it was a good idea.

Let's do it!

### Header Dictionary Keys are always Native Strings

*[Issue 1338](https://github.com/kennethreitz/requests/pull/1338)*

Previously, Requests would always encode any header keys you gave it to
bytestrings on both Python 2 and Python 3. This was in principle fine. In
practice, we had a couple of problems

- It broke overriding headers that are otherwise automatically set by Requests,
  such as Content-Type.
- It could cause unpleasant `UnicodeDecodeError`s if you had unicode header
  values on Python 2.
- It didn't work well with how `urllib3` expects its inputs.

So we now coerce to the native string type on each platform. Note that if you
provide non-native string keys (unicode on Python 2, bytestring on Python 3),
we will assume the encoding is UTF-8 when we convert the type. Be warned.

### Proxy URLs must now have an explicit scheme

*Merged in [Issue 1497](https://github.com/kennethreitz/requests/pull/1497),
originally raised in
[Issue 1182](https://github.com/kennethreitz/requests/issues/1182)*

You used to be able to provide the proxy dictionary with proxies that didn't
have a scheme, like this:

    {'http': '192.0.0.1:8888', 'https': '192.0.0.2:8888'}

This was useful for convenience, but it turned out to be a secret source of
bugs. In the absence of a scheme, Requests would assume you wanted to use the
scheme of the key, so that the above dictionary was interpreted as:

    {'http': 'http://192.0.0.1:8888', 'https': 'https://192.0.0.2:8888'}

It turns out that this is often not what people wanted. Rather than continue to
guess, as of 2.0 Requests will throw a `MissingScheme` exception if such a
proxy URL is used. This includes any proxies source from environment
variables.

### RequestException is now a subclass of IOError

*[Issue 1532](https://github.com/kennethreitz/requests/issues/1532)*

This is fairly simple. The
[Python docs](http://docs.python.org/2/library/exceptions.html#exceptions.RuntimeError)
are pretty clear on this point:

> Raised when an error is detected that doesn't fall in any of the other
> categories.

Conceptually, `RequestsException` should not be a subclass of `RuntimeError`,
it should be a subclass of `IOError`. So now it is.

### Added new method to PreparedRequest objects

*[Issue 1476](https://github.com/kennethreitz/requests/pull/1476)*

We do a lot of internal copying of `PreparedRequest` objects, so there was a
fair amount of redundant code in the library. We added the `PreparedRequest.copy()`
method to clean that up, and it appeared to be sufficiently useful that it's
now part of the public API.

### Allow preparing of Requests with Session context

*Proposed in [Issue 1445](https://github.com/kennethreitz/requests/issues/1445),
implemented in [Issue 1507](https://github.com/kennethreitz/requests/pull/1507)*

This involved adding a new method to `Session` objects:
`Session.prepare_request()`. This method takes a `Request` object and turns it
into a `PreparedRequest`, while adding data specific to a single `Session`,
e.g. any relevant cookie data. This has been a fairly highly requested feature
since Kenneth added the `PreparedRequest` functionality in 1.0.

The new primary `PreparedRequest` workflow is:

    r = Request()

    # Do stuff with the Request object.

    s = Session()
    p = s.prepare_request(r)

    # Then, later:
    s.send(p)

This provides all the many benefits of Requests sessions for your
`PreparedRequest`s.

### Extended the HTTPAdapter subclass interface

We have a `HTTPAdapter.add_headers()` method for adding HTTP headers to any
request being sent through a Transport Adapter. As part of the extended work on
proxies, we've added a new method, `HTTPAdapter.proxy_headers()`, that does
the equivalent thing for requests being sent through proxies. This is
particularly useful for requests that use the CONNECT verb to tunnel HTTPS data
through proxies, as it enables them to specify headers that should be sent to
the proxy, not the downstream target.

It's expected that most users will never worry about this function, but it is a
useful extension to the subclassing interface of the `HTTPAdapter`.

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

Well, Requests just [released its version 2.0](https://twitter.com/kennethreitz/status/382582748705488896),
and that's a major release right there. To help make your life easier, I've
whipped up this post: a lengthier explanation of what has changed in Requests
to bring us up to 2.0. I'll tackle each change in order of their presence in
the changelist, and link to the relevant issues in Github for people who want
to see what fool convinced Kenneth it was a good idea.

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

### Timeouts Are Better

*Fixed downstream from us in urllib3.*

Timeouts have been a source of pain for a lot of people for quite some time.
They tend to behave in unintuitive ways, and we ended up adding notes to the
documentation to attempt to fight this problem.

However, thanks to some sweet work done in urllib3, you now get better control
over timeouts.

When `stream=True`, the timeout value now applies only to the connection
attempt, not to any of the actual data download. When `stream=false`, we apply
the timeout value to the connection process, and then to the data download.

To be clear, that means that this:

    >>> r = requests.get(url, timeout=5, stream=False)

Could take up to 10 seconds to execute: 5 seconds will be the maximum wait for
connection, and 5 seconds will be the maximum wait for a read to return.

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

*Implemented as part of the proxy improvements mentioned later.*

We have a `HTTPAdapter.add_headers()` method for adding HTTP headers to any
request being sent through a Transport Adapter. As part of the extended work on
proxies, we've added a new method, `HTTPAdapter.proxy_headers()`, that does
the equivalent thing for requests being sent through proxies. This is
particularly useful for requests that use the CONNECT verb to tunnel HTTPS data
through proxies, as it enables them to specify headers that should be sent to
the proxy, not the downstream target.

It's expected that most users will never worry about this function, but it is a
useful extension to the subclassing interface of the `HTTPAdapter`.

### Better Handling of Chunked Encoding Errors

*Identified by many issues, but the catalyst was [Issue 1397](https://github.com/kennethreitz/requests/issues/1397),
and implemented in [Issue 1498](https://github.com/kennethreitz/requests/pull/1498).*

It turns out that a distressingly large number of websites report that they
will be using chunked encoding (by setting `Transfer-Encoding: chunked` in the
HTTP headers), but then send all the data as one blob. I've actually touched on
this [in a previous post](//lukasa.co.uk/2013/04/User_Agent_Strings/).

Anyway, when that happens we used to throw an ugly `httplib.IncompleteRead`
exception. We now catch that, and instead throw the much nicer
`requests.ChunkedEncodingError` instead. Far better.

### Invalid Percent-Escape Sequences Now Better Handled

*Proposed in [Issue 1510](https://github.com/kennethreitz/requests/issues/1510),
resolved by [Issue 1514](https://github.com/kennethreitz/requests/issues/1514).*

This is fairly simple. If Requests encountered a URL that contained an invalid
percent-escape sequence, such as the clearly invalid `http://%zz/`, we used to
throw a `ValueError` moaning about an invalid literal for base 16. That, while
true, was unhelpful. We now throw a `requests.InvalidURL` exception instead.

### Correct Some Reason Phrases

*Proposed and fixed by [Issue 1456](https://github.com/kennethreitz/requests/pull/1456).*

We had an invalid reason phrase for the HTTP 208 response code. The correct
phrase is `Already Reported`, but we were using `IM Used`. We fixed that up,
and added the HTTP 226 status code whose reason phrase actually is `IM Used`.

### Vastly Improved Proxy Support

*Proposed many many times, I wrote
[a whole post](//lukasa.co.uk/2013/07/Python_Requests_And_Proxies/) about it,
and fixed by [Issue 1515](https://github.com/kennethreitz/requests/pull/1515).*

HTTPS proxies used to be totally broken: you could just never assume they
worked. Thanks to some phenomenal work on `urllib3` by a number of awesome
people, we can now announce support for the HTTP CONNECT verb, and as a result
support for HTTPS and proxies.

This is a huge positive for us, and I'm delighted it made it in. Special thanks
go to [Stanislav Vitkovskiy](https://github.com/stanvit),
[Marc Schlaich](https://github.com/schlamar),
[Thomas Weißschuh](https://github.com/t-8ch) and
[Andrey Petrov](https://github.com/shazow) for their great work getting this in
place.

### Miscellaneous Bug Fixes

We also fixed a number of bugs. In no particular order, they are:

- Cookies are now correctly sent on responses to 401 messages, and any 401s
  received that set cookies now have those cookies persisted.
- We only select chunked encoding only when we legitimately don't know how
  large a file is, instead of when we have a zero length file.
- Mixed case schemes are now supported throughout Requests, including when
  mounting Transport Adapters.
- We have a much more robust infrastructure for streaming downloads, which
  should now actually run to completion.
- We now collect environment proxies from more locations, such as the Windows
  registry.
- We have a few minor assorted cookies fixes: nothing dramatic.
- We no longer reuse `PreparedRequest` objects on redirects.
- Auth settings in `.netrc` files no longer override explicit auth values:
  instead it's the other way around.
- Cookies that specify port numbers in their host field are now correctly
  parsed.
- You can perform streaming uploads with `BytesIO` objects now.

### Summary

Requests 2.0 is an awesome release. In particular, the proxy and timeout
improvements are a massive win. 2.0 has involved a lot of work from a ton of
contributors, and coincides with Requests passing 5 million downloads. This is
definitely another major milestone. So thanks for all your continuing support!
On behalf of the Requests project, I want to say that you're excellent, and we
love you all.

I think Requests is getting better all the time, and hopefully you do too. I
encourage you to download the new version and get using it. If you encounter
any problems, raise an issue and let us know.

Enjoy yourself!

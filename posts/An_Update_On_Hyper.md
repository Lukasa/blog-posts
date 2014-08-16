## An Update On Hyper

About half an hour ago I pushed the latest release of
[`hyper`](http://hyper.readthedocs.org/en/latest/), v0.1.0. This marks the
first release of hyper since early March! This delay is not because I've been
resting on my laurels. In fact, a quick check of the `git` repository shows
that there were almost as many commits between v0.0.4 and v0.1.0 as there were
_in total_ up to v0.0.4.

Why the delay? What have I been working on? Why wasn't I releasing
incrementally? This post acts as a long form explanation to bring you up to
speed with what's going on in `hyper`. If you're just interested in the
highlights, feel free to consult
[the changelog](https://github.com/Lukasa/hyper/releases/tag/v0.1.0) directly.

### Regressions

First, the bad stuff. This release of `hyper` abandons support for Python 3.3,
instead supporting only Python 3.4. Why did I do this?

Simple: Python 3.3's `ssl` module is not good enough to support HTTP/2. I need
features that it does not have: specifically, I need to limit TLS negotiation
to TLS v1.2 only. When Python 3.4 came out with that support we immediately
moved to add it, expecting that `PyOpenSSL` would be able to fill the gap for
Python 3.3.

Unfortunately, `PyOpenSSL` was missing NPN support. At the end of March I
opened a
[pull request on `PyOpenSSL`](https://github.com/pyca/pyopenssl/pull/86) that
added NPN support. I also opened two others needed for `hyper`, one adding
support for ALPN and one adding support for the socket `recv_into` method. I
then blocked my next release of `hyper` on `PyOpenSSL` merging my NPN pull
request and shipping a new release.

As of now this has not happened. `PyOpenSSL` has fallen into a form of
development hell where there does not appear to be sufficient resources to fix
[a major underlying problem](https://github.com/pyca/pyopenssl/issues/15) that
blocks `PyOpenSSL`'s next release. Until this gets resolved there are unlikely
to be future releases of `PyOpenSSL`.

A second problem stemming from this is that I no longer get to be excited about
one of `hyper`'s planned marquee features for this release. The truly excellent
[Alek Storm](https://github.com/alekstorm) added support for both CPython 2.7
and PyPy to `hyper` in March. This support remains in the library, but without
`PyOpenSSL` is unusable.

`hyper` continues to have its test suite run on PyPy, CPython 2.7, and CPython
3.3, so when `PyOpenSSL` is eventually updated it should be a trivial job to
re-add support for those Python distributions. I'll make a big deal of it when
it finally happens.

If you want more details, check out
[issue #37](https://github.com/Lukasa/hyper/issues/37).

### Major New Features

Onto the good stuff! Each feature has a link to its tracking issue (where
applicable) directly after the header if you want more details or to see
the code that added it.

#### HTTP/2-14 and HPACK-9

The development version of `hyper` has been keeping pace with the HPACK and
HTTP/2 draft versions as they appeared. Version 0.1.0 contains support for
HTTP/2-14 and HPACK-9. These versions are the current interop drafts, and also
represent the Working Group Last Call documents. This means they'll be kept
unchanged for a while so that distributions can get interop experience.

This feature alone makes it a perfect time to download `hyper` and give it a
try!

#### Server Push

[*Issue #40*](https://github.com/Lukasa/hyper/pull/40)

Back when I wrote my
[original blog post](//lukasa.co.uk/2014/02/HTTP20_For_Python/) announcing
`hyper` I had the following to say about Server Push:

> I don't believe it's possible to write a pleasant API that correctly handles
> Server Push without using either an event loop or having hyper spawn its own
> background threads. Until asyncio becomes a standard, I don't think hyper
> should force you to do the first, and it's obnoxious for a library to spawn
> its own threads. This means that, for the moment, hyper does not support
> Server Push.

I'm delighted to say that this is no longer true! Once again, Alek Storm did
the heavy lifting here and deserves huge congratulations. The upshot is that
`hyper` now supports Server Push and has it turned on by default. Check out
[the docs](http://hyper.readthedocs.org/en/latest/advanced.html#server-push)
for usage examples.

#### Buffered Socket To Avoid Overhead

[*Issue #56*](https://github.com/Lukasa/hyper/issues/56)

One of the bits of feedback I got in response to
[my investigation of hyper's performance](//lukasa.co.uk/2014/05/An_Investigation_Into_Hyper/)
was that I could avoid the overhead of making many syscalls by buffering socket
data in userspace.

I've now gone ahead and done that: `hyper` maintains a 64kB buffer in memory
for each connection to ameliorate some of the performance costs. It also avoids
copying data for much longer, saving some overheads involved in unnecessarily
copying memory around. Both of these changes should cause performance
improvements.

For those interested in borrowing the implementation, the buffered socket can
be found
[here](https://github.com/Lukasa/hyper/blob/development/hyper/http20/bufsocket.py).
You are welcome to use it in your own projects under the same license as
`hyper`.

#### Nghttp2: If You Can't Beat 'Em, Join 'Em

[*Issue #60*](https://github.com/Lukasa/hyper/issues/60)

`hyper` by default uses its built-in pure-Python HPACK encoder and decoder,
which is reasonably quick and fairly efficient. However, the awesome
[nghttp2](https://nghttp2.org/) project contains a C implementation of HPACK
that achieves astonishing compression faster than `hyper` does, while using
less memory.

If you're concerned about performance, you can now use nghttp2's encoder and
decoder with `hyper`! Simple install nghttp2 with the Python bindings, and
`hyper` will prefer nghttp2's encoder and decoder to its own.

This allows the best of both worlds: zero dependency installation, with
optional performance improvements for those who want them.

In future, I'll probably approach nghttp2's maintainers to suggest that they
replace the Python bindings with a CFFI-based Python module to allow PyPy users
to also use nghttp2.

#### Trailers

*Discussed in part in [Issue #71](https://github.com/Lukasa/hyper/issues/71).*

I've also taken my first step into adding features that are not supported in
vanilla `httplib`, by adding support for HTTP trailers. These are headers sent
_after_ a chunk-encoded body, containing metadata that was not known before the
request was sent. This is often things like `Content-Length`.

`httplib` does not support trailers transparently, but `hyper` does. See
[this section of the docs](http://hyper.readthedocs.org/en/latest/api.html#hyper.HTTP20Response.gettrailer)
for the relevant methods.

I'm aware that many people will not be able to use this method for fear of
needing to switch back to vanilla `httplib`, which does not have it. I plan
to start work on the abstraction layer soon, which should help ameliorate these
concerns.

### Bugfixes and Minor Features

We also added a number of smaller features. These are listed below, click on
their issue links to see more information.

- `HTTP20Response` objects are context managers.
  [*Issue #24*](https://github.com/Lukasa/hyper/issues/24)
- Pluggable window managers are now correctly informed about the document size.
  [*Issue #26*](https://github.com/Lukasa/hyper/issues/26)
- Header blocks don't get corrupted if read in a different order to the one in
  which they were sent.
  [*Issue #39*](https://github.com/Lukasa/hyper/issues/39)
- Default window manager is now smarter about sending WINDOWUPDATE frames.
  *[Issue #41](https://github.com/Lukasa/hyper/issues/41) and
  [Issue #52](https://github.com/Lukasa/hyper/issues/24)*
- Fixed inverted window sizes.
  [*Issue #27*](https://github.com/Lukasa/hyper/issues/27)
- Correctly reply to PING frames.
  [*Issue #48*](https://github.com/Lukasa/hyper/issues/48)
- Added a universal wheel.
  [*Issue #46*](https://github.com/Lukasa/hyper/issues/46)
- HPACK encoder correctly encodes header lists with duplicate headers.
  [*Issue #50*](https://github.com/Lukasa/hyper/issues/50)

### Progress Is Being Made

This is still an exciting time to be working on `hyper`. We've got lots of open
issues and plans for the future. This means there's really no better time for
you to download it and try it out. Give it a go!

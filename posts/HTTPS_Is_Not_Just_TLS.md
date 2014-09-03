## HTTPS Is Not Just TLS

Right now the CPython core development team are having an extremely lengthy
conversation about
[whether to enable certificate verification for HTTPS clients](https://mail.python.org/pipermail/python-dev/2014-August/136034.html).
When it comes to making this the new default behaviour in Python 3.5 the
response is an enthusiastic "Yes!" (just as it should be). However, when it
comes to backporting that behaviour to a patch release of Python 2.7 the
response from the core committers is distinctly more lukewarm.

Stating this up front, I'm strongly in favour of backporting the fix to Python
2.7. Yes, it will break people's code, but guess what: that code was *already*
broken! The user's just didn't know it yet.

Here's the thing: HTTPS has semantic meaning. It's not just "HTTP with TLS".
The desire to do that is being addressed by the HTTPBis Working Group in the
[Opportunistic Security Internet Draft](https://tools.ietf.org/html/draft-ietf-httpbis-http2-encryption-00).
HTTPS carries significantly more baggage than just purely encryption.

The covenant of HTTPS is this: when you ask for a website with a URL that has
the scheme `HTTPS` you are asking to contact someone who is authoritative for
that resource. You are essentially cutting all intermediaries out of your
connection in an attempt to get exactly the resource you asked for.

This means that validating the certificate isn't incidental to HTTPS, it is
*vital* to HTTPS. If you present a certificate that is not valid for the domain
that I'm trying to contact, then I should absolutely stop contacting you. I
should fail hard. I should throw my toys out of the pram and totally refuse to
collect them again.

What I should *not* do is shrug my shoulders and go ahead anyway. Anyone
relying on the fact that `httplib` or `http.client` or `urllib2` all do exactly
that is either being lied to by their client or, just as bad, they are
mis-using HTTPS.

Yes, there are organisations in the world that provide resources via HTTP over
TLS on port 443 without valid certificates. Those organisations are wasting
their CPU cycles on encrypting data that is *clearly* of no importance to them.
Worse, they are implying a level of privileged communication to both ends of
the connection that neither end has. Not verifying the certificate doesn't just
put the client at risk: it puts the server at risk as well. The server can't
tell you didn't verify the certificate, and so it can't tell whether you got
MITM'd or not.

Yes, there are problems with certificates and certificate authorities that make
a lot of people unhappy with HTTPS. Tough. Want to use self-signed certs? Fine,
but either add the cert to your trust store or don't call what you're doing
HTTPS: it isn't.

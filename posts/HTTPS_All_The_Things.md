## HTTPS All The Things (Especially This Thing)

I think we can all agree that [HTTPS](http://en.wikipedia.org/wiki/HTTP_Secure)
is a good thing. In fact, as a bit of a privacy-nut, I'd go further: I think
all encryption is a good thing. We increasingly live in a world where our lives
play out on the internet, and a key part of keeping some part of our lives
private is using encryption.

For this reason, I've wanted to add HTTPS to my blog for a very long time. The
blog doesn't actually _need_ it: I don't have any form of
[CMS](http://en.wikipedia.org/wiki/Content_management_system) for the blog,
instead managing all my content via a collection of scripts. This meant there
was no risk of any sensitive information getting passed over the open internet:
the scripts were secured by [SSH](http://en.wikipedia.org/wiki/Secure_Shell).

Nevertheless, there are a few good reasons to use HTTPS. After all, we really
want to get to a place where more websites have a green lock symbol in the
browser bar than don't. After all, the more websites that have SSL enabled,
the less weird encrypted browser traffic is to your average snooper. They
expect to see encrypted traffic, so that's what they get.

Additionally, I want you to be able to trust that, when you visit my website,
the only person who can be doing evil stuff to you is me. There are some ways
in which this isn't true (Hey, Heroku: please don't be evil, OK?), but by and
large you can be sure that the only people who can mess with this website are
me and Heroku. This is infinitely better than in the HTTP-only case, where any
wandering person can hop in the middle of your web traffic and inject all kinds
of malicious crap into my carefully-crafted HTML.

HTTPS isn't perfect. Indeed, the whole notion of
[certificate authorities](http://en.wikipedia.org/wiki/Certificate_authority)
is a pretty sketchy one. [Armin Ronacher](http://lucumr.pocoo.org/about/) has
lectured me about the evils of CAs in the past, and if you ask him really
nicely ([@mitsuhiko](https://twitter.com/mitsuhiko) on Twitter), I'm sure he'll
explain their evils to you as well.

Nevertheless, I take the pragmatic approach: HTTPS is better than nothing, and
I wanted to make it available.

### Guess What?

Well now I have. As of right now, you can visit this website at either
[https://lukasa.co.uk/](https://lukasa.co.uk/) or
[https://www.lukasa.co.uk/](https://www.lukasa.co.uk/) and check out the snazzy
green padlock in your browser bar. As of now I suggest you update your
bookmarks (you do have me bookmarked, right?) to point to the HTTPS link. All
future internal links on my blog will point to the protocol-relative version of
my site, to ensure that those of you who care about security continue to reap
its benefits.

### What Took You So Long?

Honestly? Heroku charges $20 a month to use SSL on your website. Right now I
was paying the princely sum of $0 a month to host this blog, so it was tough to
bite the bullet and do this. The only silver lining is that
[StartSSL](https://www.startssl.com/) kindly provided me a free SSL certificate
for this domain. And they'd do it for you too! In fact, it's kind of their
whole business model. Very cool.

This hasn't changed, so what is special about now? The real reason I added
HTTPS to this website _now_ is an interesting one, but I haven't finished with
it yet. When I have, I'll write another blog article to let you know. In the
meantime, you'll just have to live in suspense!

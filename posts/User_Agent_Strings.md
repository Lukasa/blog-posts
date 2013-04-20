## User-Agent Strings (or, Don't Make Me Come After You)

A very long time ago (read: ten years ago), we were in-between the so-called
First and Second Browser Wars. Internet Explorer had killed Netscape
Navigator by taking advantage of their desktop monopoly and Scrooge McDuck-like
financial reserves to install a free copy of Internet Explorer on every single
computer in the world (basically). Internet Explorer 6 was the dominant
browser, and Netscape as a company was over.

Netscape, before their demise, had embarked on a project to totally rewrite
their web browser. Their new code was open-sourced and given to the Mozilla
foundation. In hindsight, this was a stunningly successful move, with the
ever-awesome Mozilla Foundation going from strength to strength now, nearly ten
years after its foundation.

The second browser war was initially a festering cold war between the reborn
Netscape Navigator (now entitled Mozilla Firefox) and the dormant Internet
Explorer 6 (eventually updated to IE 7 after a 6 year development freeze).
Later, other parties like Google Chrome joined the party. Oh, and Safari and
Opera were kinda floating around in this war too, but honestly they're not that
important to the story I'm trying to tell.

Anyway, long story still kinda long, as part of these two browser wars,
browsers felt the need to compete with each other on features. However, to use
these features you needed to get web developers to build web sites that used
them. The problem is that your new feature would only work on _your_ browser.
This meant that, when some poor soul came along trying to view your
super-awesome ActiveX powered web page, and they had the misfortune to be using
Netscape Navigator, your website would at best look awful, and at worst explode
in several mysterious ways.

These people would then go away and tell their friends about your _crappy_
website that wouldn't even render properly! And they'd say that their friends
should use your competitor's website, even though your competitor can't even
_spell_ ActiveX! And you'd go out of business and your children would have to
go to a state school, and it would just be _horrible_.

So you needed some way to tell what features a browser had. There was a way to
do that, of course: Javascript. Unfortunately, some features couldn't be easily
detected in Javascript, and writing Javascript was, well, weird, and Javascript
was slow, and so lots of websites didn't want to do that (or didn't know they
should). What would they do instead?

Well, RFC 1945 and RFC 2616 (the HTTP 1.0 and HTTP 1.1 specifications) stated
that all browsers, web crawlers and other tools that interacted with web
servers should identify themselves using a special header in the HTTP they
send: the **User-Agent** header. This header should be (as much as possible)
unique to a specific _type_ of agent. This means that Internet Explorer should
send a User-Agent header that is different to all other browsers _and_ to all
other versions of IE.

_"Perfect!"_ cry the web developers. _"Our servers can check for this string,
and use its value to determine how to render the page!"_. And so begins the
the trouble.

### The Trouble

You see, the problem with using the User-Agent string to check for features is
that the User-Agent string tells you _nothing_ about what features a given
User-Agent has. After all, that's not what it's for! So you, naïve late-1990s
web programmer, might write your site when only Mozilla Firefox has support for
the hot new _Twiddlor_ feature (note: not a real feature). So you only server
_Twiddlor_-enabled pages to people whose User-Agent strings identify them as
being a version of Firefox.

The problem is, six months later the guys in Redmond get around to adding
_Twiddlor_ support to Internet Explorer. But all their users are still
complaining that none of their favourite websites will let them use _Twiddlor_,
instead claiming that the website is _"Best used in Mozilla Firefox"_ or some
such nonsense.

How does Microsoft get you to show them the _Twiddlor_-enabled page? Simple:
they change their User-Agent string! Sadly, I'm not even joking: this is
actually what happened. To prove it, I'm going to show you a few modern browser
UA strings.

Here's the UA string sent by Google Chrome version 27.0.1453.47 beta (yeah),
running on my Mac:

    Mozilla/5.0 (Macintosh; Intel Mac OS X 10_8_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/27.0.1453.47 Safari/537.36

_"What is all that crap?"_, I hear you ask, quite rightly. Why does it say it's
Mozilla? It's not Mozilla! You're quite right. But enough people have tested
for Firefox by just checking that the word 'Mozilla' is in the UA string that
everyone puts it there. And I mean __everyone__. Check out Safari, also on my
Mac:

    Mozilla/5.0 (Macintosh; Intel Mac OS X 10_8_2) AppleWebKit/536.26.17 (KHTML, like Gecko) Version/6.0.2 Safari/536.26.17

Notice that both Safari and Chrome claim to be versions of Safari. That's
pretty damn weird.

What about Internet Explorer 10, on my Windows machine?

    Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.2; WOW64; Trident/6.0)

At least it's not claiming to be Safari! In fact, this is the best UA string
I've seen, being a fairly honest representation of the browser.

Finally, let's check Firefox, also on my Windows box.

    Mozilla/5.0 (Windows NT 6.2; WOW64; rv:20.0) Gecko/20100101 Firefox/20.0

### What Should A User-Agent String Look Like?

To see an example of how these were supposed to look when the standard was
originally proposed, we can see what
[Requests](http://docs.python-requests.org/en/latest/) sends.

    python-requests/1.2.0 CPython/2.7.2 Darwin/12.2.0

Short and to the point. The 'browser' and its version, the 'platform' and its
version, and the OS (sort of) and its version.

### Why Does This Matter?

In principle, the new Javascript-heavy world should have cured us of this
problem. People should write JS that tests for features and then uses them, and
serves a less interesting version of the web page if you don't support it. And,
mostly, this is what happens! Libraries like JQuery have taken a lot of the
hard work out of doing this, so most websites you'll encounter nowadays do the
right thing.

The problem is, sometimes they don't. And when they don't, you can encounter
strange and confusing bugs. These bugs then tie up developer time and generally
make everyone's life worse. To provide an example, I'm going to briefly walk
you through a bug that appeared on the Requests GitHub page a few days ago.

### An Example

A user reported that, when he accessed a specific web page by doing a simple
GET with no complicated stuff, he was getting a `httplib.IncompleteRead`
exception thrown into his face.

This was odd in itself. This exception is only ever thrown when either the user
or the remote server is using chunked encoding, but the user reported that he
didn't think either party was doing so. He also kindly provided the URL, so
that I could reproduce the bug locally. (This is excellent practice, by the
way: I'm far more likely to help out if I can easily reproduce your bug on my
machine.)

When I made the same request, I also got the `IncompleteRead` exception thrown
in my face. Further investigation showed that the web server claimed to be
serving using
[chunked encoding](http://en.wikipedia.org/wiki/Chunked_encoding), but in fact
was just sending the page as normal. This is pretty bad, and there's not much
Requests can do about this: the web server is simply doing the wrong thing.
**First note for website developers: do NOT claim to be using chunked encoding
when you are not!**

I was interested to see if we could get the page data anyway, so I patched my
local copy of the standard library to see what we got when I returned the data
instead of throwing an exception. What I saw was the second unpleasant thing
this web site had done. The HTML for this page was about 20 lines long. All it
did was embed, at full size, a
[frame](http://en.wikipedia.org/wiki/Framing_%28World_Wide_Web%29) containing
another page, or a warning if your browser doesn't support frames.

This is pretty obnoxious: why not just server the other page? Why require
frames? You aren't even _doing_ anything with them, you're just using them for
the sake of using them! **Second note for website developers: do not use frames
when you don't need them! They are awkward for anything that isn't a browser.**

In an attempt to be helpful, I pulled the URL being framed out of the HTML and
suggested the user hit that instead. Out of sheer curiosity, I then did a
Requests GET on the URL.

Requests threw an exception again.

I was pretty surprised here, the page rendered fine in my browser. So I looked
at the exception. `Connection Reset By Peer`, read the socket error text. For
those who don't know their network protocols, this indicates that the
[TCP](http://en.wikipedia.org/wiki/Transmission_Control_Protocol) connection to
the web server was closed while we were expecting data on it.

This is very odd. Requests sent a totally compliant, basic HTTP GET request,
and the remote server was shutting the connection in response to it. Doing this
is totally against the HTTP specification. Any compliant server is _required_
to respond with an HTTP error code and a `Connection: close` header if it
wants to tear the connection down. Additionally, why did it work fine in Chrome
but fail in Requests?

There's really only one obvious thing to do. I grabbed Chrome's User-Agent
string and got Requests to send that instead of its own UA string. (For those
who want to spoof their UA string, Requests allows you to pass it as a header.
We only set one ourselves if you don't provide one for us.)

Success! The page rendered and returned to us.

For those who want a summary, what was happening here is that the remote site
was sniffing the User-Agent header. Instead of checking for features, however,
what it was doing was using the header as a gatekeeper! If you don't have the
right User-Agent, you don't just get a less feature-filled site: you get
_nothing_. Not even an HTTP error page.

This is probably the worst example of User-Agent sniffing I've ever seen. This
was a website developer using a bad practice to violate the HTTP specification.
In addition to simply being rude, this is also a genuine cost for many
developers. And crap like this leads to stupid UA strings like the ones I
showed above.

This is also the **third note for website developers: always send HTTP error
codes, don't just close connections**.

### The Moral Of The Story

The most important lesson, however, is this.

**Ignore the User-Agent string unless you absolutely have to.**

Detecting browser features is not what the User Agent string is for, so please
don't use it for that. And if you do (which I'm sure you will, because no-one
listens to me anyway), make sure that you don't _refuse service_ based on the
User-Agent. If you want to render a slightly different page, fine, I get that.
But don't refuse to render it at all. It's obnoxious, it's brittle, and it's
_so_ 1990s. And besides, as I showed above, all modern User-Agents can _lie_
in their User-Agent string! You can set it in Firefox, and in Chrome, and
(probably) in IE, Safari and Opera as well. So not only are you mis-using it,
you're not even getting accurate information!

And if you do abuse the User-Agent string, and I catch you, I will come through
the internet and _get_ you. And then I will write an angry blog post, and
publicly shame you.

So,
[St Kieran Catholic Church and School](http://www.catholicweb.com/splash/stkierens/).
Sort your website out. It's being obnoxious. And is that really the impression
you want to give?

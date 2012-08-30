## OAuth Needs To Die In A Fire

As a personal project I've recently been looking into creating a small Twitter
client in Objective-C/Cocoa. This is largely because I quite like Objective-C
as a language and haven't had the chance to write anything significant in it,
but also partly because
[Twitter have made the news recently](http://arstechnica.com/business/2012/08/new-api-severely-restricts-third-party-twitter-applications/)
with their API nuttiness and I wanted to play around with it.

So, I surfed over to the [Twitter API docs](https://dev.twitter.com/docs) to
take a look at how to implement the API. After reading and playing with the
API for a few days I've got some quite strong opinions on various parts of it
(few of them good), but far and away the defining characteristic of my
investigation of the API is OAuth.

### OAuth Background

Of course, I've read stuff about OAuth before, so I knew before I got started
that it was highly controversial. As a result, my natural reflex was to avoid
OAuth if possible. A cursory reading of the API docs, however, shows that
Twitter pretty aggressively rate-limit calls to the API: 100 requests per hour
for non-authenticated users, 350 for authenticated ones. As a result, I was
definitely going to have to authenticate, and OAuth is their authentication of
choice.

This meant I'd have to use OAuth. Here, my problems began.

### I Hate OAuth

Firstly, I had to familiarise myself with the OAuth authentication flow. It
turns out that this is nothing like as simple as I'd like it to be. To explain
why, I need to make sure you're familiar with the niche OAuth is intended to
fill.

The purpose of OAuth is to be able to provide applications with access to your
account without giving them your password. This apparently simple notion ends
up being really quite complicated. In short, to achieve this, OAuth requires
that you perform the following steps.

1. Register your application with your OAuth provider (in this case Twitter),
   and obtain from them your Consumer Key and Consumer Secret.
2. When the customer starts up your application for the first time, send a
   request to a URL provided by the OAuth provider with various OAuth headers
   set in the HTTP, including your consumer key and secret. Some providers
   (hello again Twitter!) also require you to set custom headers. This caused
   me [some unrelated difficulty](http://code.google.com/p/oauth/issues/detail?id=230).
   This procedure should return an OAuth response that provides a 'request
   token'.
3. Send your user to a URL, along with the request token, to ask the user to
   authenticate your application. For desktop applications, this involves some
   bizarre out-of-band procedure involving PINs. At least Cocoa makes this
   part easy. The user is required to log in.
4. This will either cause a callback to a different URL, or the user will
   return to your application to input the PIN (or whatever out-of-band
   weirdness your OAuth provider uses).
5. Now you need to make *another* web request (yes, really) to the OAuth
   provider, this time to get an authorisation token. This token can be stored
   away (e.g. in the OS X keychain), and has to be used to sign all subsequent
   requests to the API.

Keeping track, that's three web requests to obtain an authorisation token.
Additionally, for desktop apps, the user is required to context shift to a web
browser in order to authorise the application. This is a serious set of
disadvantages.

Here's the thing: it's really not worth it. Yes, we don't have to put password
security in the hands of application developers, which is a huge advantage.
Yes, we don't transfer the password information with each request, which is an
advantage.

Here's the thing: it's way too hard to get working. OAuth libraries are a pain
in the ass to write, and I don't think I've encountered one yet that functions
perfectly. Additionally, the standard is a nightmare. The RFC for OAuth 1 is
[35 pages long](http://tools.ietf.org/html/rfc5849), and the lead author on
the OAuth 2 standard has actually [withdrawn his name from the spec](http://hueniverse.com/2012/07/oauth-2-0-and-the-road-to-hell/).
This complexity makes it almost impossible to guarantee how any single OAuth
implementation will behave, which makes very close reading of the relevant API
docs a must. Indeed, a subtle mis-reading of Twitter's API documentation cost
me nearly a day of work single-stepping through code execution when the error
turned out to be in one of the HTTP headers instead.

It seems as though the complexity associated with OAuth must be unnecessary. I
am not a serious big-time web developer, and one of those people might be
better placed to judge the difficulty of the task, but I can't help but feel
that there must be an easier way to achieve this. Certainly OAuth adds a
number of points of failure and a stunning quantity of boilerplate code.

It also discourages casual API exploration. When deciding whether or not I am
going to use an API for the first time, I usually fire up a Python shell,
`import requests` and start wandering through the API. OAuth adds some
significant hurdles to this process. I need to work out how to obtain my
authorised token with which I will sign my subsequent requests, and there are
some bugs present in the OAuth implementation in Requests that have not yet
been ironed out. (Shameless plug: do you know lots about OAuth? If so,
[come help us out](https://github.com/kennethreitz/requests)!)

All of this means that I am less inclined to use your API. I can't be the only
person who thinks this when they encounter an OAuth-based API, and as a result
it is certainly possible that many developers who could have written killer
apps for a service have called it quits early. This is a terrible shame.

### Fix It

So here is my plea: can someone please fix this? I don't have the knowledge,
the skill, or the authority to attempt to replace or improve OAuth. I would
happily provide feedback on any implementation that seeks to replace it, but
I simply don't have the prerequisites to develop it myself. That said, I
desperately want it. I want it to be fixed. So, for the sake of developers
everywhere, can someone step up to the plate?

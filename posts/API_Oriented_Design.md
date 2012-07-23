## The Importance of API-Oriented Design

Many programmers (or coders, or software engineers, or computer wizards, or
whatever term you would like to use; the arguments had over this could and
probably will fill a whole blog post on their own) find themselves involved to
a greater or lesser extent in the development of libraries and programming
utilities for the use of their colleagues in the discipline. There is no
denying that the development of good libraries is a noble and worthy pursuit,
although I have to say that from time to time I do wonder if perhaps some of
the effort expended on doing that could be diverted to creating useful tools
for non-technical types. Not the point though, so I'll return to what I meant
to say.

My single real involvement in the open-source movement of any note whatsoever
(and believe me, it is of very minor note) is my occasional contributions to
the [Python Requests](http://python-requests.org) library. My reasons for
choosing this library to contribute to are many-fold, and include
[the friendliness of the maintainer](http://lukasa.co.uk/2012/05/Politeness_And_Open_Source/)
and the cleanliness and structure of the code. My primary reason, however, was
because I used it myself. And the reason I used it myself is very simple: its
API is brilliant (for those too lazy to Google: the definition of
[API](http://en.wikipedia.org/wiki/Api).

### The Importance of Good APIs

When I started out writing code, not very long ago, I was in the position that
I suspect many new developers are in of being unable to tell the difference
between a 'good' API and a 'bad' one. The act of writing code was so much
conscious effort for me that I found no particular method of accessing a
resource 'more intuitive' than any other.

Initially this led me, with all of the arrogance of the man who doesn't know
how little he knows, to dismiss the idea that there existed too much
variation in API design. My particular train of thought rested on the notion
that the API would fundamentally be shaped by the implementation, and that
there were probably not many possible implementations for a given
functionality (a notion I now consider to be laughable).

Then I tried to write a simple web scraper in Python.

I wanted to grab the entire archive of XKCD's images, and save them with
numbered filenames. The experienced amongst you are already scoffing at the
simplicity of the task, but for me at the time it was a huge hurdle, and
required me to sit down and seriously think about how to do what I wanted to
do. The first step for me was to see what built-in modules Python had that
could help me do the job (and in fact this is still my intuition now). This
led me to
[urllib2](http://docs.python.org/library/urllib2.html#module-urllib2).

At the time, I had what I would consider a 'barely functional' knowledge of
HTTP (although, to be fair, I still think there's a huge amount of HTTP I
don't know, and even more I don't fully understand). I knew about
[HTTP verbs](http://en.wikipedia.org/wiki/HTTP_Verbs#Request_methods), so
I knew what query I needed to make to get the page I wanted. Despite that,
I spent ages sitting, working through that documentation, in order to get the
web data I wanted.

A few days later, I was browsing the web when I noticed someone praising
Requests. I was intrigued, and looked into it. When I saw
[the comparison with urllib2](https://gist.github.com/973705), my jaw hit the
floor. *This is brilliant!*, I thought.

I didn't know the half of it. Eventually, I swung around onto the GitHub repo
for Requests, and (thanks to the clean structure of the module) was able to
quickly find where the API was declared.
[When I saw this](https://github.com/kennethreitz/requests/blob/develop/requests/api.py),
I was astounded at how simple and easy to understand this was. The fact that
it uses an astonishingly tiny number of methods and is very well documented
is certainly helpful, but the more helpful feature is the fact that the API
has an almost perfect 1-to-1 mapping to the underlying behaviour. You want to
make an HTTP GET? `requests.get`. Want to make a POST? `requests.post`.

Clearly, even in my infantile programming state, I was able to see that there
was something about this API that was different to urllib2. Each did the same
thing, but somehow Requests was intuitive where urllib2 wasn't. Clearly, this
API had whatever 'x-factor' makes APIs good.

The goodness of this API translated into immediate productivity improvements.
Use a quarter of the lines of code of urllib2 to get the same job done. The
fact that each method takes the same arguments means fewer references to the
documentation to write the correct code. The simplicity of Requests is such
that it makes investigation using the interactive interpreter (or, even
better, [bpython](http://www.bpython-interpreter.org/)) trivially easy, which
means you can prototype faster. For web-based Python work, I have never found
a library since that made my life so single-handedly easier than Requests did
and continues to do.

So, how did [@KennethReitz](https://twitter.com/kennethreitz) manage to
compose so good an API when the authors of urllib2 failed?

### API-Oriented Design

The answer is surprisingly simple: design the API first. Before writing any
library code whatsoever, work out how you'd like the user of the library to
interact with it. What methods will it use?
[Will it use classes](http://youtu.be/o9pEzgHorH0)? Should the developer have
to set environment variables or write code to initialise the library? (Hint:
the answer to that last one is almost always 'no'.) Sketch out on paper the
function calls you'd make on your ideal library to achieve a variety of
common tasks. This should include the names of the methods. Think carefully
about the functionality your library is providing. Does it wrap some protocol?
Should you emulate its syntax, or frame it in terms likely to be familiar to
the developers using it? (As an example of the second, consider all
[ORMs](http://en.wikipedia.org/wiki/Object-relational_mapping).)

When you've made the design decisions for that, sit down and write the
function and class declarations required. Don't put code in them, just the
declarations. Put that in a file (e.g. an `api.py` file, or a `.h` file for C)
and then **leave it alone**. Resist the urge to add anything to that at all.
You want a minimal API that exposes all the necessary functionality in the
simplest and most intuitive way possible. If you add to it to make writing the
library easier, then you have copped out and your API just got worse.

Once you have your API, write all of the necessary code to make it work. This
code doesn't have to be at all intuitive: use [magic](http://www.rafekettler.com/magicmethods.html),
use operator overloading, use import hooks, anything. Just make sure that your
API works.

The better the coder you are, the better your underlying code will be. As it
turns out, I am more than capable of debugging most minor problems people have
with Requests (and do my best to do so, to save smarter people from having to
spend the brain time on it), in large part because the codebase is so well
structured. But even the worst coders are capable of writing a passably good
API by hiding the complexity away from the API itself, and burying it in the
library.

I call this API-oriented design, although I'm sure someone else has the claim
to using that name first. If you do it, you will be writing libraries that
other developers will want to use. This is because the reality of the
situation is that most developers just want to use a library to do one small
part of their thing, and have no interest in developing the library or
understanding its structure.

So don't force them to. Be intuitive and simple, and the rest will follow.

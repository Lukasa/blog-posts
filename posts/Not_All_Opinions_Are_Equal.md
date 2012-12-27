## Not All Opinions Are Equal

This blog post is a reaction to
[this](http://david.heinemeierhansson.com/2012/rails-is-omakase.html) blog post
by David Hansson. If you have the time, I highly recommend reading it. If you
don't, I'll summarise the most relevant bits as I go.

### Having Things Your Way

David Hansson has recently written a blog post about
[Rails](http://rubyonrails.org/), and in particular about how Rails is an
opinionated piece of software. Rails, much like
[Django](https://www.djangoproject.com/) in the Python world, is very much a
'batteries included' framework. It makes a series of design choices based on
what its maintainers consider to be its primary use cases, which often include
which [ORM](http://en.wikipedia.org/wiki/Object-relational_mapping) to use, or
which templating language/engine they'll use.

Hansson's post was inspired by the regular requests to
change some of the 'batteries' in Rails. One of his off-the-cuff examples is
the inclusion of [CoffeeScript](http://coffeescript.org/) in Rails. His point
is that, despite it being a setting that's possible to override, some people
get annoyed about its inclusion at all.

This is understandable: everyone has opinions, and its natural to want to voice
them. You and I will probably not see eye-to-eye on some things (maybe even
most things). This is great, and us hashing out our opinions is almost always
going to be a good thing.

However, while you're entitled to have your opinion, I'm not required to listen
to it.

### This Problem Is Common

Open source software development is
communal. Most projects of any value are sufficiently large that it's hard to
manage them on your own, and so its important to have people to help, even if
[they only do the easy stuff](/2012/11/Not_Everyone_Needs_To_Be_A_Rockstar/).

This means that third-party
contributions are encouraged: maintainers want your ideas and your code! But,
when you do this, it's important to remember the following things:

1. Ideas are cheap.
2. 'Good' is often subjective.
3. **All opinions are not equal**.

The first two points have been covered to death, but the third is interesting.
To illustrate it, I want to take an example from Python Requests.

Recently,
[Requests went v1.0](http://kennethreitz.com/announcing-requests-v100.html).
This was a major code change, and along with it came a change in the API.
Mostly the changes were fairly mild, with the commonly-used paths being almost
identical, but there was at least one significant change in
the API used for Requests sessions.

Previously, it was possible to write code that looked like this:

    import requests
    params = {'hello': 'world'}
    s = requests.session(params=params)

Essentially, the common configuration you wanted in the session could be passed
in on the constructor. V1.0 of Requests changed the API such that you could no
longer do that. Instead, the equivalent code now read:

    import requests
    params = {'hello': 'world'}
    s = requests.session()
    s.params = params

This change was not universally popular. In fact, in the time between the API
change and the documentation update,
[at](https://github.com/kennethreitz/requests/issues/1034)
[least](https://github.com/kennethreitz/requests/issues/1038)
[six](https://github.com/kennethreitz/requests/issues/1039)
[issues](https://github.com/kennethreitz/requests/issues/1040)
[were](https://github.com/kennethreitz/requests/issues/1042)
[opened](https://github.com/kennethreitz/requests/issues/1057). To be clear,
this is fine: reporting a bug or inconsistency in documentation is a great
thing.

However, if you go through those issues, you'll begin to see a common theme. In
particular, one user who had not previously contributed to the project became
insistent about reverting the API, despite Kenneth making it clear
that the API would not be changed back.

This is the point I want to highlight. If you have an idea, please say it. This
user did, and that was the right thing to do. He then went one step further and
provided the code to implement his idea, which was also good.
Up to this point, he had done
everything right, and his contribution would have been well-regarded, even if
ultimately rejected.

However, when Kenneth said he did not want to change the API, that should have
been the end of the discussion. This is because of reason number 3: not all
opinions are equal. Fundamentally, with an open-source project, the buck stops
with the maintainer. It doesn't matter how good your idea or how well you
advance it, if the maintainer doesn't like it you should either let it go or
fork the project. It is nothing short of rude to repeatedly complain about how
your _obviously better_ idea isn't being implemented.

The reason for this is simple: if you've come out of nowhere, there are loads
of things about the project _you don't know_. You don't know its philosophy,
its priorities, its ideals. These things come from exposure to the project, and
that comes from contributing over time. For instance, I know that, for Kenneth,
Requests' API is sacrosanct. The API must be as intuitive and as simple as
possible, and anything that gets in the way of this will never, ever be
accepted. I also know that Kenneth's definition of 'simple' is not necessarily
'fewest lines of code'.

This means that I, along with the other regular Requests contributors, don't
propose changing the API very much, and any code change we make that might
affect the API gets Kenneth's sign off before we even open a Pull Request. In
turn, these learned behaviours mean that when (if) we _do_ propose an API
change, Kenneth takes it way more seriously than when someone less familiar
with the project does.

### You Get One Go

David Hansson puts it more succinctly than I do:

> The less time you've spent in our establishment, the further away you are
> from helping out in the kitchen, the less weight your opinion will carry.
> Especially if it's expressed in a loud, obnoxious fashion.

If no-one knows who you are, your opinion is one amongst thousands. Bear
it in mind. In a good project, you'll get a fair hearing, but if someone closer
to the kitchen disagrees with you, that will probably spell the death of your
idea.

So bear this in mind: you get _one_ shot at advancing your opinion. Be concise.
Be friendly. Provide code. And if all that fails, _gracefully_ accept the
decision. Throwing a fit when you don't get your way is childish, and it wins
you no friends.

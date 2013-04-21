## I am a Persona Identity Provider

Three weeks ago to the day
[I posted](//lukasa.co.uk/2013/03/HTTPS_All_The_Things/) about how I had added
SSL support to this website. In that post I mostly discussed why SSL is awesome
and why people should use it, but at the end I cryptically hinted that I had
some more notable motivation for adding SSL:

> [So] what is special about now? The real reason I added HTTPS to this website
> _now_ is an interesting one, but I haven't finished with it yet.

Guess what? I've finished with it. And in case you hadn't noticed, the article
title gives the game away.

### A Question Of Identity

Online identity is important and controversial. The modern web relies very
strongly on knowing who you are, so that services can be provided to you.
Social media sites want to know who you are so they can connect you with
others. Shopping websites want to know who you are so they can sell you things.
Other websites just want to know who you are so they can make sure you paid for
the service you're using.

For almost the entire history of the web, this has meant having a username and
a password. Each site was their own domain, and you had your own username and
password for each.

This had some advantages. You could choose what identity you wished to use. If
you were a revolutionary, you could have a Facebook page that was clean and
sanitised so your government didn't think you were a threat, while using a
totally different username, email address and password for your rabble-rousing
blog. Each site and service had a different notion of what it was to be _you_.

Unfortunately, while this advantage is strong, it turns out that storing
passwords securely is
[really](http://gizmodo.com/5988057/evernote-was-hacked-and-your-encrypted-passwords-got-stolen),
[really](http://mashable.com/2012/06/08/linkedin-stolen-passwords-list/)
[hard](http://www.pcworld.com/article/259135/hackers_publish_over_450000_emails_and_passwords_allegedly_stolen_from_yahoo.html).
Securing your website against these kinds of intrusions is really hard, and
almost all websites don't really want to manage password databases.

This has led to a collection of services that centralise identity. These
services offer the ability users to log in to other websites using their
identity on the primary service.

For example, consider this image of the login page of the excellent
[500px](https://500px.com/login?r=/):

<figure>
    <img src="https://s3.amazonaws.com/django-blog/img/resources/500px_login.png" alt="500px login page" />
    <figcaption>The 500px login page.</figcaption>
</figure>

You can see that 500px lets you login with a 500px account, but also using your
Facebook, Twitter and Klout accounts. What 500px is doing here is delegating
the responsibility of establishing your identity to a third party.

The principle here is a good one. Doing identity is difficult, and for the vast
majority of websites it is not really related to the service they want to
provide. So why not let Facebook, and Twitter, and Google Plus, and GitHub, and
whoever else manage our identities for us? Website owners win, because they
don't have to manage passwords, and user win, because they only need one
password.

### The Problem

The problem here is, as [Dan Callahan](https://twitter.com/callahad) has
pointed out before, each of the services offering a definition of identity is
less than ideal. The least scary problem is that some of them, like, oh, I
don't know, [OpenID](http://openid.net/), have the crappiest interface known to
man. It's no good having a centralised identity service if no-one can work out
how to use it.

Much scarier, however, are the Facebooks and Twitters of the world. These
websites _desperately want_ to be the sole definition of your identity. The
problem is, they don't want that for your benefit. They want it so that they
can _sell_ and _control_ access to your identity. Does anyone think for even
a moment that Facebook wouldn't charge developers for using their identity
service if they had a monopoly? This is leaving aside the slightly scary
[real name policies](http://en.wikipedia.org/wiki/Nymwars) possessed by these
websites.

### The Solution

Happily, the bright bunnies at Mozilla have a solution. That solution is to
take the best aspect of these centralised identity services, which is allowing
developers to not manage usernames and passwords, and combined it with the best
aspect of traditional internet identity, which is _choice_. To do that, they
developed [Persona](http://www.mozilla.org/en-US/persona/).

I recommend you go and read about Persona, so I won't spend too long on it
here. Basically, the intention is that identity online is associated with email
addresses. Each email address you have represents a specific identity. When a
website wants to verify your identity, they should ask the organisation who
actually knows: namely, your email provider.

This idea resonates with me. I have many email addresses: some are informal and
used when I don't really care about my identity. Some are formal and associate
me with a specific organisation (like my work address). Still more represent
the identity I _want_ to have, more than the one I actually have. But each
represents a slightly different facet of me.

More importantly, though, I want to be in control of my identity. I don't want
Facebook to do it for me, and I don't really want Google to do it either. So
what is a guy like me to do?

Well, I actually own a domain name. And I have an email address at that domain
name. This means that I can do something that, even three years ago, I could
never have imagined: I can take control of my online identity and become a
Mozilla Identity Provider.

### When In Doubt, Just Do It

So I did it. It's not even all that hard. The exact mechanics of doing it are
a little complex, and I hit a few snags that I will want to write about fairly
soon so that others can avoid them, but basically it was only about an hour of
actual work. I lost a _lot_ of time to confusions with documentation, but I'm
assured that the Persona folks are working on it, and I'll hopefully document
those mistakes too.

Despite that, I am now in charge of my identity. Any website that implements
Persona can now ask my website if this guy claiming to be me is actually me.
And if my website says yes, the other website will accept it.

This is awesome. More importantly, the future it represents is awesome. Because
I don't just have to define my own identity. If someone else I know is
unwilling to let Google define their identity but only has an @gmail email
address, I can give them an @lukasa.co.uk one, and they can use that instead!

Persona lets me see a future where identity and trust are decentralised. Where
who we are is defined by the people we _choose_ to define it. Where those who
are sufficiently worried about identity can choose to assert their own. And
that rocks.

Either later today or sometime next week I'll write about my implementation of
a Persona Identity Provider, and hopefully will open-source a version of my
implementation to allow other sites to drag-and-drop a solution.

In the meantime, I choose to bask in the exhilarating feeling of being, once
again, in charge of who I am.

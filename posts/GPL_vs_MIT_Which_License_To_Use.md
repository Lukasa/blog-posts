## The GPL vs. The MIT License: Which License To Use

A great many developers, myself included, believe that it is important to
spend at least some time contributing to open-source software projects. These
projects will hopefully be licensed (if you haven't got a license on your
open-source project, you're doing it wrong), to ensure that your contributions
are used in the way you (or the project maintainer) wants them to be used.

This means that when you create your pet open-source project (at the minute,
mine is [Minimalog](https://github.com/Lukasa/minimalog)), one of the first
things you'll want to do is determine which open-source license you want to
use.

This will be your first problem. According to a quick count,
[Wikipedia](http://en.wikipedia.org/wiki/Comparison_of_free_software_licenses)
lists more than 42 licenses in their comparison page. This is an overwhelming
quantity of choice!

Never fear. In the next five minutes, I'll do my best to distill these choices
down to something more palatable. This is going to be absurdly reductionist,
and so I recommend doing more research on your own if you remain unsure.

### The Non-Starters

Right off the bat we can get rid of quite a few licenses. It is always safest
to use the more widely-used licenses. In part, this is because they have been
widely tested, both in terms of getting other people to comply with them and
in court (such as in Germany
[not once](http://www.groklaw.net/article.php?story=20040725150736471) but
[twice](http://gpl-violations.org/news/20060922-dlink-judgement_frankfurt.html)
). More importantly, developers who care about licenses will tend to get wary
around licenses with which they are unfamiliar.

The licenses seen most often are the following: the
[Apache license](http://en.wikipedia.org/wiki/Apache_License), the
[BSD license](http://en.wikipedia.org/wiki/BSD_license), the
[GPL](http://en.wikipedia.org/wiki/GNU_General_Public_License) (GNU General 
Public License), the
[LGPL](http://en.wikipedia.org/wiki/GNU_Lesser_General_Public_License) (GNU
Lesser General Public License), and the
[MIT license](http://en.wikipedia.org/wiki/MIT_license). Of these licenses,
the Apache license is seen pretty infrequently outside of Apache Software
Foundation software. This is obviously not an indictment of the license
itself, but it means developers will be less familiar with it. For that
reason, I would only use the Apache license if I was absolutely sure I wanted
it and nothing else.

Of the remaining four licenses, two of them (BSD and MIT) are considered to be
'permissive', with the GPL and LGPL being called 'copyleft' licenses. To
understand the difference, we will go briefly into what each term means.

The 'permissive' licenses are likely to be what a layperson thinks of when
(or indeed if) they think about open-source software. These licenses require
almost no effort to comply with. To comply with these licenses, the person or
organisation using the code simply has to reproduce the license and copyright
notice. Otherwise, they may do as they wish with the code, including bundling
it up and selling it as-is.

In contrast, the GPL is a 'copyleft' license. This is a much more complicated
idea. To begin to understand it, it helps to know that the GPL is written and
maintained by the
[Free Software Foundation](http://www.fsf.org/) (whose website is unexpectedly
ugly). The FSF is an organisation started by Richard Stallman, who is
notoriously *wacky* (I mean it, take a look at
[his Wikiquote page](http://en.wikiquote.org/wiki/Richard_Stallman) sometime)
when it comes to Free Software. The entire purpose of this license is to make
it impossible for someone who uses the code to make changes that they do not
then make available freely and openly.

As an example, if Minimalog was licensed under the GPL, you would not be free
to make changes to it, release it in a binary and sell it as "Steve's
Superawesome Superblog Software", no matter how much you liked the
alliteration, unless you also released your changed software under the same
license. More dramatically, the GPL is also *viral*. If I release a software
library licensed under the GPL (for example, if I had written
[libdvdcss](http://www.videolan.org/developers/libdvdcss.html), and I do wish
I had), you download the source and build it into a dynamic library (such as a
`.dll` or `.so`) which you then link against your own proprietary code in
order to use, guess what? The FSF argues that your work counts as a
'derivative work', which means that you also need to release it under the GPL.

Now, it's not actually that dramatic. The FSF also provides the LGPL, which
as far as I can tell exists to solve this one use case. A library provided
under the LGPL can be linked against without forcing your work to use the LGPL
as well.

### Ethical Issues

It's clear that there's a question of ethics involved in this choice. Stallman
and the FSF do not want to be considered part of the 'Open Source Software'
movement, as they think that open-source is not enough: software must also be
free as in freedom. Whether I agree or disagree with this perspective is a
subject for another time.

The only ethical choice I'm going to consider is this one: do you want other
people to be able to profit from the use of your code, or not? The permissive
licenses allow people almost limitless ability to use your code as they
please, including incorporating it directly into proprietary software. The GPL
does not.

### Communities

Interestingly, a bit of a divide in license choice has begun to appear in the
software community. If you sit and browse through an open-source repository
storage, such as GitHub, or Bitbucket, or Google Code, or Sourceforge, you'll
find that desktop developers tend to favour the GPL, while web developers tend
to favour MIT/BSD.

Obviously, this is a wild generalisation: the BSD UNIXes, for instance,
clearly use the BSD license, despite being operating systems. However, in
general, the more of the code is part of a web backend, the more likely it's
licensed permissively. Consider some of the big desktop projects: the GNU
tools are GPL (obviously), the Linux kernel is GPL, VLC is GPL, and MySQL is
GPL(ish). Then, consider some of the big web tools: nginx is BSD(ish), Ruby on
Rails is MIT, Django is BSD, Twitter Bootstrap is Apache (which is
permissive), and jQuery is either GPL or MIT (which means, functionally, it's
MIT). This shows a pretty major divide between the two camps.

This is likely to be a cultural divide. Web developers seem to me to be part
of an entrepreneurial culture that lives and breathes around the idea of
creating 'startups'. This is all well and good, but it means that these
developers want to be able to wrap their tools into their proprietary software
stacks if possible. As an example, if Ruby on Rails was licensed under the
GPL, it is reasonably likely that any Rails web app would need to be licensed
under the GPL as well, which means you could download and launch Twitter or
GitHub yourself with minimal effort. This is unlikely to be in the interests
of those two organisations, or any other startup that wants to rapidly expand.

### How To Choose

The secret is that really, it's easy. If you want to prevent people from
making changes to your work without providing them back to you, use the GPL.
If you want people to use your name when they use your work in something, use
MIT or BSD. If you honestly don't care at all, use the
[WTFPL](http://en.wikipedia.org/wiki/Do_What_The_Fuck_You_Want_To_Public_License)
(the Do What The Fuck You Want To Public License). That's the only decision
you need to make.

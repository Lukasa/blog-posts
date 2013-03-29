## Grade Calculator: Round 2

So, about 9 months ago I wrote a
[grade calculator](http://lukasa.co.uk/2012/06/Things_That_Are_Way_Too_Hard/)
for students of the University of St Andrews. I hacked the whole thing together
in about 8 hours of feverish inexperience, and it managed to just about work.

To commemorate the fact that final exams are coming around again in St Andrews
(even if I'm not there), I decided to give the calculator a face lift. The CSS
was pretty average, and frankly the design was nothing stellar. I obviously
wasn't planning to turn it into the sexiest webapp ever designed, but it could
at least try to look less totally awful.

So, [I did it](http://grades.lukasa.co.uk/). Altogether, I'd say I'm pretty
proud. Once again, this app is not exactly impressive: I'm fairly sure that
some (or most) of the Javascript is crap, and I had to work around some pretty
horrible behaviour on mobile devices. Nevertheless, it works just as well as it
did before, and manages to look a ton better while it does so.

Why don't I regale you with the changelist?

### Changelist

- **New Features**: Absolutely none!
- **Bugs Fixed**: Absolutely none!
- **Performance Changes**: The performance is now worse!

Yes, that's right, I finally bent to the whims of the internet and added JQuery
to one of my websites. I kept not doing so on the grounds that it's weighty and
unnecessary, but I finally bit the bullet. It works pretty well! Certainly the
API is ton better than the standard DOM API. Unfortunately, I paid the price of
the snazzy functionality: the website is noticeably slower to load. Of course,
the fact that there's a huge png file to download for the background doesn't
help either.

On the other hand, there are nice animations now, which is cool.

### Wait, that's it?

Yeah. Sorry, it's not as cool as I'd like it to be. Still, check it out: while
I'm still super-annoyed that it's necessary, it's also far-and-away the most
successful personal project I've ever done, so I'm weirdly proud of it.

As always, the
[source is on GitHub](https://github.com/Lukasa/st-andrews-grades). If anyone
else in the UK has a ridiculously complicated university grading scheme, I
encourage you to fork the repo and create a similar website for your uni.
Alternatively, get in touch and I can do it for you.

Sorry about the delay in posting anything of interest, I've been crazy busy.
Hopefully I'll have some more stuff to write about soon. In the meantime,
it's a bank holiday today. Go out and have fun.

## Things That Are Way Too Hard

When I went to university, I found myself studying Physics at the
[University of St Andrews](http://www.st-andrews.ac.uk/) in Scotland. This
worked out very well for me: I can wholeheartedly and unreservedly recommend
St Andrews as a place to study. The
[School of Physics](http://www.st-andrews.ac.uk/physics/staff_students/index.php)
is chock-full of great teachers and researchers, and my understanding is that
the rest of the university is at the same calibre.

Unfortunately, the University also uses a marking scheme that I have chosen to
call **The World's Most Confusing Marking Scheme**. It is so complicated that
I have devoted an entire section to trying to explain it to you. If you aren't
interested in the absurd complexities of the University of St Andrews' marking
scheme, skip to the section entitled **Mitigate The Problem**.

### How Hard Can Grading Be?

Most universities grade individual modules and items of work on a percentage
scale. This system is easily understood, and makes it simple to evaluate the
relative differences in achievement between individuals and modules.

St Andrews, however, only takes the brightest of students, and it attempts to
challenge them in every possible way. This means using the **The World's Most
Complicated Marking Scheme**. Allow me to explain it to you.

Let's begin by considering examinations. Each department (or 'School') sets
its own examinations. These examinations are marked out of some number,
defined by the School. The more sensible Schools (*cough cough* Physics) mark
exams out of some number that goes into 100 nicely, like 50 or 100. The less
sensible Schools mark out of some weird number, like 72.

The examination mark is converted into a percentage. At this point it would
be perfectly possible to report this percentage to students and then call it a
day. However, the next stage is actually to convert this percentage grade to a
grade on the so-called '20 point mark scale'. This scale is a number between
0 and 20 inclusive, with a single decimal point allowed (so isn't that a 200
point mark scale?).

So far it's complex, but not complicated. It's easy to establish a linear
relationship between a percentage mark and a 200 point mark scale. However,
what happens next is utterly bizarre. Rather than use a linear conversion,
a *non-linear* conversion is used. This means that the difference between 30%
and 40% is *different* on the 20 point scale to the difference between 80% and
90%.

This is absurd. What is more absurd is that, as far as I can tell, these
conversions are not the same from School to School. The School of Physics
publishes their conversion rates (though the table is buried deep inside a PDF
document), but I've been unable to find a published version of any other
School's conversion. Additionally, many of the Schools in the Faculty of Arts
don't do the percentage grading, but instead grade straight onto the 20 point
scale! This makes it impossible to determine accurately what it means for a
student to have gotten a 16.

It gets worse, though. In the UK, most degrees are what are known as
[honours degrees](http://en.wikipedia.org/wiki/British_undergraduate_degree_classification).
This definition of 'honours degree' is not the same as that used in some other
countries (e.g. Australia), by the by. Anyway, the crux of that meaning is
that, upon graduation, your degree is given one of the following
classifications: First Class Honours, Upper Second Class Honours, Lower
Second Class Honours, Third Class Honours, and Pass.

In Universities with sensible marking schemes, a percentage grade is set as
the boundary between one classification and the next. At the end of the degree
you take an average of your modules (weighted by how many credits they were
worth), and wherever you fall defines your classification.

In St Andrews, you do that too, except you take a credit-weighted mean of your
20 point scale grades. Additionally, the university defines 0.5-point-wide
'border zones' where your degree classification depends on your median grade.
And not just any median, your *credit-weighted* median.

Needless to say, this mathematics rapidly becomes something that is not easy
for students to do. Lots of students assemble spreadsheets that do the mean
easily, and then struggle with the median. Later in my university career, an
Excel spreadsheet started doing the rounds that could do the median, but was
using macros (Macros! In this day and age!) to do it. This seemed absurd to
me. People who weren't familiar with Excel struggled with the spreadsheet, it
looked like ass, and needed to be emailed around the student body. It had to
be possible to do better.

### What's The Worst That Can Happen?

It occurred to me that I have just enough knowledge to be dangerous on this
front, so I went ahead to try to build a web app that would help. The result is
[here](http://grades.lukasa.co.uk/). The name isn't exactly snappy, but it's
fairly attractive and works pretty well.

This was my first introduction to Javascript: before beginning to write this
app I had never written a line of it. This blog uses only Javascript written
by others (namely Google and Disqus), and so I didn't really know the first
thing about it. Still, I knew that I wanted there to be a dynamic number of
modules, and short of forcing the user through an extra web-page for them to
POST a form telling me how many modules they did, I would have to manipulate
the DOM, and that means Javascript.

With that in mind, then, I decided I should aim to scale as well as possible
while keeping costs as low as possible. This means a static site using one
dyno on [Heroku](http://www.heroku.com/), which makes my hosting cost zero. I
used a sub-domain of my blog domain, which means no domain registration fee. I
hosted the single minified CSS file on S3, which costs almost nothing to host
and serve. The only other way I could think of to improve scalability would be
to cause all the calculation to be client-side, in the Javascript itself.

The user would already have to download some JS, which means they'd already
have to hit S3 twice, so the only cost increase would be bandwidth and storage
of a larger JS file. Because the calculation is client-side, my Heroku dyno
can process requests incredibly quickly, as it will only ever take a single
request from a given visitor.

The only downside is that Javascript isn't usually all that fast. However, it
turns out that, for this relatively simple calculation, Javascript is 'fast
enough'. Neither I nor anyone I've spoken to has noticed any lag with the JS,
even on mobile devices. Altogether, Javascript is probably faster for the user
than having to POST onto the server and have it do the calculation!

All that I needed to do then was wrap the static `index.html` page in the
thinnest Heroku-suitable client I could find (in this case,
[Sinatra](http://www.sinatrarb.com/)) and viola! A fully functioning web app,
from conception to completion in about 8 hours.

### Mitigate The Problem

Why bother? Well, the [app](http://grades.lukasa.co.uk/') has some major
advantages over the Excel spreadsheet. For one, it's a lot more approachable
for non-technical users, especially those who are unfamiliar with Excel. It's
also easily available, without requiring an installed copy of Office, and
works on mobile devices. This makes it widely usable by the student body.

This wide usage base is important, because, and I cannot stress this enough,
this **should not be a problem students face**. The fact that I was able to
identify a problem and build something that actually helps in 8 hours suggests
that the university has made a serious *faux pas* with the grading system. In
the event that a university official swings by this blog (unlikely, but
possible), it'd be great if you'd drop me a line to explain why you use a
system this apparently-braindead.

More importantly, though, it gave me an opportunity to mitigate the problem.
I have personal experience of students, myself included, being irritated by
attempting to work out what class of degree they can expect. I can't fix the
issue, because I don't work for the university, but I can contribute something
that makes peoples' lives a little bit better. This gives me the warm fuzzies,
so that alone was worth it.

Finally, several personal firsts for me. I wrote the CSS entirely from
scratch, which I've never done, and so I now have a much better idea of what
it's like to write CSS. I now have some limited experience with Javascript,
which is great, although I've still got work to do on this front. In
particular, I want to write some tests and clean up the Javascript: it looks
ugly to me. This is also my first Ruby app (although, honestly, it barely
counts, as it only uses Sinatra as a thin shell over the index page).

All-in-all, I'd call the development of this app a productive use of 8 hours,
wouldn't you?

## Politeness and Open-Source Software

Over the last couple of weeks, there has been what can really only be called a
'brouhaha' brewing over the role of politeness in open-source software. This
argument was largely brought to the fore by a comment made by Linus Torvalds
in response to a GitHub pull request. You can see the comment Linus made
[here](https://github.com/torvalds/linux/pull/17#issuecomment-5659970). The
comment that earned Linus' ire has been deleted, but was not posted by the
person making the pull request. It read, approximately, "I did not realise
Linus' shit does not stink. Thanks for clearing that up."

This caused the internet, in particular
[r/programming](http://www.reddit.com/r/programming) and
[Hacker News](https://news.ycombinator.com/) to explode. The comment was
posted on both news aggregators: the Reddit discussion can be found
[here](http://www.reddit.com/r/programming/comments/tionj/linus_torvalds_doesnt_do_github_pull_requests/),
and the HN discussion [here](http://news.ycombinator.com/item?id=3960876).
Each discussion lead to hundreds of comments, even on the significantly less
traffic-heavy Hacker News.

Of course, the obvious follow-up occurred, and the discussion hit the blogs. I
can happily say that I missed the vast majority of the blog-based noise.
However, I was a little startled to find, about a week after the event, my RSS
feed telling me that the normally fairly quiet
[Kenneth Reitz](http://kennethreitz.com/) had made a blog post tangentially
related to the issue.

[Kenneth's post](http://kennethreitz.com/be-cordial-or-be-on-your-way.html)
did not explicitly refer to Linus, and is in fact not intended to be directly
relevant to the 'Linus' scenario. However, it seems inarguable to me that
Linus' indelicate comment provided the final inspiration for Kenneth to write
a post on an issue that is clearly very important to him.

Now that some of the madness has died down, I'd like to weigh in a little bit.
Much like Kenneth's post, it will not be directly aimed at Linus. However, it
will be related to Linus.

### Caveats

Before I go any further with my discussion, I want to practice some full
disclosure.

First, let me say that I have limitless respect for Linus Torvalds. If I ever
have the privilege of meeting him in person, I will almost certainly embarrass
myself by coming over all starstruck. Linus is undeniably an incredibly
talented programmer, software architect, community leader and communicator. He
has successfully developed and coordinated the development of one of the most
significant open-source software projects in history, and if I could count
myself as even a fraction of the man he is I would be happy.

Second, I should note that I have interacted with Kenneth Reitz from time to
time. His [Requests library](https://github.com/kennethreitz/requests), aside
from being an example of brilliant API design, was the first open source
library I ever contributed to. My contributions were exceptionally minor, but
he was either thankful for them or pretended to be. It made for a gentle
introduction to the world of open-source software development, and has
provided me with an enthusiasm for the process that has stuck with me.

### How Does Politeness Relate to Open-Source Software?

Good question. Let me answer it with an example. My first contribution to the
Requests library was a three character change. It is hosted on GitHub, so you
can see the change [here](https://github.com/kennethreitz/requests/pull/425).
All I did was add three characters to the list of allowed characters in
cookies. This change fixed bugs, but is nevertheless fairly trivial. Despite
this, Kenneth expressed thanks for the fix.

For me, this was a big deal. Before this point, I had only written code for
myself or during my internship. When writing code for myself, it was
acceptable and not embarrassing to write less-than-perfect code. After all,
no-one would ever see it. During my internship, I was subject to code review,
and expected to be learning on the job, so once again my ugly or incorrect
code was not a source of embarrassment.

This, however, was different. I was providing a (trivial) fix to a real bug
in one of the most popular Python libraries around. This code would be
subject to scrutiny by any number of people, almost all of whom would be more
experienced than me, and many of whom would be at least as intelligent as me.
My code was in a public arena for the first time, and it terrified me.

To have Kenneth's response be unambiguously positive was exactly the kind of
response I needed. Admittedly, it was unlikely that Kenneth would be mean,
because I had done nothing to deserve it, but he could easily have been
ambivalent or lukewarm about my contribution. His evident appreciation of my
minor contribution made me feel valuable to a project that I greatly admired.


### Linus' Role

So how does Linus Torvalds fit in to this?

Linus directs the development of, amongst other things, the Linux kernel. This
is far-and-away the most famous and notable open source project (sorry GNU
folks, but you know it's true).

It is also an unforgiving development location. The Linux kernel is written in
an intimidating combination of GCC C and inline assembler, and the code base
is enormous. Furthermore, because it is an OS kernel, bugs are unacceptable.
As a result, fools, particularly those whose opinion of their own skill is not
backed up by tangible results, are not tolerated lightly.

Linus has a long history of being what I would euphemistically call 'direct'
about his opinions. A quick browse through
[his page on Wikiquote](http://en.wikiquote.org/wiki/Linus_Torvalds) will
turn up some gems.

However, the bulk of this is not assholery; that is to say, it is not Linus
lashing out at the less-intelligent or less-talented simply because he can.
In the majority of these cases, the person bearing the brunt of Linus' wrath
made an *ad hominem* comment to begin with. In other cases, it is those who
repeatedly show themselves unable to meet the standards required for kernel
development.

In the rough-and-tumble world of kernel development, this is OK. I would
suggest that first-time open-source contributors should not be contributing
to the Linux kernel. The requirements of your code are so significant that it
is likely that your contributions will be considered inadequate.

Here's the thing: that's OK! I'm going to stand up right now and say that I am
insufficiently experienced to contribute to the Linux kernel, and that that's
totally understandable. I have time to develop that skill set if I want it.

### Get to the point already!

OK, here's the point. Open-source needs to be welcoming to contributors who
have not contributed before. This means being positive about work whenever you
can, even if all it means is one positive sentence. It's twenty seconds out of
your day, and it'll encourage a developer to spend more time on your pet
project, so why not do it?

This includes developers on more challenging projects. If a kernel newbie like
myself submits a patch to the Linux kernel code, I fully expect that patch to
be rejected, or at least returned to me for editing. However, there is a
constructive and polite way of doing this, and the word 'moron' is not
involved in it.

Now, to be clear, I don't know if Linus is constructive and polite or not. In
reality, I am fairly sure he doesn't see the low-level changes: they are
probably submitted to other developers who combine changes together and push
them upwards towards Linus. I suspect, on balance, that Linus probably isn't
an asshole.

However, this goes both ways. Contributors to open-source projects should not
expect project coordinators to treat their work like a donation. The
coordinator must take time out of their day to review your changes and ensure
that you haven't made any mistakes or introduced new and subtle bugs. This is
a time-consuming process. If they decide that your code requires work before
it can be integrated, or that it solves a problem they do not believe needs
solving, then **do not** get haughty about it. In the end, the project is
theirs, and they have the final say on what does and does not get added to it.

To sum up: let's try not to be assholes! Contributors, don't get so invested
in your code that rejection or editing of it angers you to the point that you
make a personal comment about someone. Coordinators, try to be constructive
and polite when you critique submitted code. If everyone does this, we can
encourage more developers to donate their time to open-source projects. And
that really only ends well for us all.

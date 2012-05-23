## The Success Of Git: Why Subversion Needs To Die

I've not been programming for all that long, but in that time I've used two
different version control systems. In my internship I used Subversion (SVN),
and in my personal life and open-source stuff I've used Git. I find the
relationship between the various major VCSes to be quite interesting, and
wanted to devote a little time to talking about them.

For the sake of fairness (and because I'm a Python hacker), I should point out
that the vast majority of the things I say about Git apply to Mercurial. I
have not used Mercurial very much at all, but my understanding is that the
general usage model for Mercurial is more-or-less identical to Git. Certainly
Mercurial and Git are both what is known as *distributed* version control
systems.

### Woah, woah. Distributed?

Oh, yeah, I should probably go back a few steps. So, the reason I want to
compare Git and SVN is because the very different usage patterns the two tools
create is fundamentally based upon a single ideological difference. This
difference can be summarised as follows: in Git, no repository is special; in
Subversion, the central repository is special.

Traditional version control systems like Subversion use a system that is a bit
like a client-server model. You set up a central Subversion repository on a
server somewhere, and check your initial code into it. If you want a copy of
the codebase, you check out from that repository. At that point, you have a
copy of the code on your machine, but you cannot treat your local repo as
identical to the central repo.

Git (and Mercurial) differ in a single core idea: each repository is on equal
footing with every other repository. I can find any repository of a codebase,
clone it, and then have a completely functional repository on my machine. Any
code check-in is local, into the repository on my machine; I can create and
merge branches without affecting any other repository; and other people can
clone the repository on my machine and merge into any other repository. This
lack of central authority means that VCSes like Git are called *distributed*
VCSes.

### Ok. Does this difference matter?

For me, yes. I have a habit (potentially an unfortunate one) of using VCS as
an indication of how 'real' my code is. This means that I tend to want to
check my half-finished code into souce control, including code that isn't
passing tests, or even code that fails to build.

Subversion requires me to fight this tendency. Anything I check into source
control **must** run properly, because anyone who wants a copy of the code
will check out my changes. At my internship the build machines would check out
a copy of the Subversion repository to run the build, so I would have had the
added ignominy of breaking the build. This never happened to me, thankfully,
but the embarrassment would be...significant.

Git does not force this behaviour on me. If I feel I've made progress towards
a problem I can commit those changes into my local Git repository. This means
that if I make another change that ruins everything, I won't lose those
changes. Once again, this has never happened to me, but it is the kind of
nightmare that ferments inside my twisted brain.

Another advantage of the local nature of Git is that the repository history
becomes flexible. As long as a change has not been pushed out to any other
copy of the repository, I can edit that change to my heart's content. This
means that I can not only make half-baked commits into my Git repo, but when
I have solved the problem I can edit the previous commits I made. The number
of times I have done this is nearly beyond counting, and my Git log reads much
more cleanly than it would otherwise.

Editing the repository history must be done with caution, of course.
`git bisect` is a great tool, and it becomes significantly less useful if your
edited history makes certain commits fail tests. I hear that history editing
is a lot less common in the Mercurial community, but I can't really speak from
experience.

Once I've become comfortable with my changes and edited the history so the
whole thing looks like I'm a sane developer instead of a crazy person who has
no idea what he's doing, I can push the changes to some more public location.
Once I've done this I need to resist the temptation to edit those commits, as
if you do you end up making future merges hellish.

### Oh yeah, merging.

Speaking of merging, anyone tried to do a merge in Subversion? Yeah, it's not
fun. I've worked with some very, very intelligent developers on Subversion
repositories, and each one of them was apprehensive each time they had to
perform a merge.

This difficulty with merging causes an unwillingness to branch in Subversion
as well. Of course, institutional requirements mean that damn it, you *will*
make feature branches, whether you like it or not; but in my experience
these branches were used for large scale changes (e.g. new releases), not for
individual features.

Conversely, with Git I'll create branches for features large and small,
because doing so has essentially no cost associated with it. This has the
added advantage of ensuring that I always have a copy of my code that will
definitely, absolutely work: the `master` branch.

### So Subversion sucks. Why should it die?

Because it's my belief that Subversion actively makes it difficult for
developers to develop. Your VCS should encourage you to version your code, and
to isolate your development. Subversion's single repository nature discourages
the checking in of your code until you're absolutely confident of it, which
causes SVN histories to be filled with enomous commits.

This reluctance to check in code means that the diffs between commits are
enormous, and then knowing which commit introduced a bug becomes almost
entirely unhelpful to the debugging process. Having debugged in just such a
circumstance, I can tell you that if I'd had `git bisect` back then my life
would have been significantly cheerier.

Similarly, the difficulty merging that Subversion introduces causes developers
to be reluctant to create feature branches for anything less than a really
significant change. This causes all the developers in a group to commit their
changes to the master branch. This wildly clutters up the history of the
repository, but more importantly, can put the repository in a state where the
default checkout fails to build.

Finally, you can't commit when you're unable to connect to the central
repository! This is so helpful it's almost impossible to describe to someone
who hasn't used it. Commits become fast and plentiful, and it becomes
incredibly easy to work away from your central repository.

I would optimistically like to predict the eventual death of Subversion. Much
like CVS is (almost) dead, Subversion will become increasingly less popular
for new projects. This will push Subversion first into being predominantly
used in industry (a state that is almost true already), and then into being
used only in legacy projects (why hello there COBOL!).

Subversion has been a highly useful tool for developers that was a significant
step forward from previous VCSes. However, the new wave of distributed VSEes
promises to make everyone's life easier, and so I for one will be happy to
take Subversion out the back and shoot it in the head.

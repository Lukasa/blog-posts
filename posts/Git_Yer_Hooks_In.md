## Git Yer Hooks In

Git, like all good source control systems, allows you to customise the
behaviour of the SCM when certain actions occur. For lots of people in lots
of projects it's not vital to use these hooks, but from time-to-time you find
yourself repeating the same commands time-and-again. In these scenarios, it is
worth looking into setting up a hook for your SCM to run on your behalf.

Because I'm one of the cool new generation of software monkeys, I use
[Git](http://git-scm.com/) as my chosen SCM. One of the great things about Git
is that it has a very simple hooks system which is both easy to understand and
to use. Allow me to demonstrate.

### My Example, Because It's All About Me

To demonstrate the usage of Git hooks, I will walk through an example of a Git
hook I created for my own use. This will be a Git commit hook: that is to say,
an action that runs whenever I commit into a specific repository.

The repository I'll be using today is the one in which this website is stored.
This website is hosted on [Heroku](http://www.heroku.com/), which means I push
and update the website using Git. For this reason, I have a vested interest in
minimising the quantity of commands I run on a given update.

My particular issue was pointed out by a shall-remain-anonymous member of the
PC Gamer Steam chat room, who noted that the CSS for this blog is served from
S3 in an unminified format. This is less than ideal, as it increases both the
time it takes to download the four CSS files and the bandwidth costs I pay to
supply it to users from S3.

Of course, I want to keep the unminified files in source control, because
no-one wants to have to edit minified CSS files. That means I need to balance
the need to keep hold of the unminified files while supplying the minified
ones. This is exactly the kind of situation where a Git hook comes in handy.

### Prepare For Git Hook

So let's consider what we need to do. I already have
[yuicompressor](http://developer.yahoo.com/yui/compressor/) installed on my
machine, so my hook will definitely use that. The way it ought to behave is
that, on a commit, Git should minify the full-size CSS files, move them to
the `static/` directory, and then run `python manage.py collectstatic` to
upload them to S3.

This requires that I do a few things. Firstly, I need to copy the current
static files out of the `static/` directory and into a new one, where there
will be no name conflict. I called this directory `_static/` (yeah, not an
inventive name, but never mind). I then `cp -r`'ed the contents of `static/`
into `_static/`. Finally, I added this new folder to the repository and 
committed the change.

Next, I write the script I want to run when I commit to the repository. This
can actually be any executable file, but because I want to run a series of
shell commands I've decided to just write a simple Bash script:

<script src="https://gist.github.com/3379184.js?file=gistfile1.sh"></script>

This script should be put in the `.git/hooks` directory of your git
repository. Call it `post-commit`, and run `chmod +x .git/hooks/post-commit`.

This script will then run every time you make a commit into the repository.
It will use `yuicompressor` to minify the CSS files, place them in the static
files directory and then upload them to S3.

### Finishing Up

Of course, this is a very simple look at Git hooks. Git allows you to place
hooks on many other parts of the Git use process; in fact, I would be willing
to put significant sums of money on the notion that it is Git server hooks
that allow Heroku to function the way it does.

These hooks turn Git from a simple content versioning system into one that can
alleviate the most repetitive tasks associated with your work flow, ensuring
that boring commands get run every time you need them to be.

Happily, other good SCMs allow you to do this too: certainly, Mercurial does.
This means that, regardless of what you use, you should be able to make your
development life a bit easier with Git hooks. Even if all you do is save
yourself and your visitors a tiny bit of bandwidth.

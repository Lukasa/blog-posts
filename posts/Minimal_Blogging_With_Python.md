## Building a Minimalist Blog in Python (or, How I Learned to Stop Worrying and Love Web Development).

As of today, I have taken my final examination as an undergraduate student of
Physics. With my graduate software engineering job beginning in September, I
am one graduation ceremony away from being a Software Engineer instead of a 
Physics Student.

Despite the fact that the world needs another programming/tech blog like a
hole in the head, I enjoy writing. Given that tech is one of the few subjects
on which I'm likely to have something to say, the world will just have to
suffer through it. Additionally, my first-choice programming language is
Python, and as we know, [GitHub is a pythonista's resume](http://pydanny.blogspot.co.uk/2011/08/github-is-my-resume.html).
I see this blog as being a front-page for my GitHub resume.

With that in mind, we should look at the most significant project I've hosted
on my GitHub: this blog!

### Minimalist Blogging

Before I started building this blog, I had done almost no web development of
any kind. Despite my go-to programming language being Python, the bulk of my
extended work has been local applications. Some of these have been in Python,
but the bulk have actually been in C# or Java.

None of these have been web apps, however. Given the increasing importance of
web applications, it is increasingly indefensible for me to be so totally
unfamiliar with web application development.

My need for a blog and my lack of web application knowledge gave me the
opportunity to kill two birds with one stone. So, I resolved that I would
build a blog from the ground up.

I was able to limit the scope of my job by considering the limited number of
things I needed my blog to do. For the moment, at least, it is likely to be
very low-traffic, which meant I didn't need it to handle huge loads or a wide
variety of content. I'm technically competent, which means I could write my
blog posts in some form of markup, rather than needing a WYSIWYG interface to
write them. I don't expect much in the way of comment spam, so I haven't
implemented a spam filter. In essence, all this blog does is store and display
posts and comments.

Finally, if I was writing Ruby I would probably just have downloaded one of
the many, many blog frameworks for Ruby on Rails. Happily,
[Django](https://www.djangoproject.com/) exists, so I can write my webapp in
Python! Doing things in Python always makes it better.

### Right, but you said Minimalist? This doesn't seem all that Minimal.

Part of the minimalism was explained before, with the limited functionality
of the blog. The rest of the minimalism was designed in for my own aesthetic
appreciation. The blog CSS is based upon the delightfully minimal
[Skeleton](http://www.getskeleton.com/) framework with a few adjustments. The
blog (in its basic form) uses no Javascript of any kind. We expect to host no
static files, so it's easy to host on Heroku.

As I progressed, I found that this minimalism had some advantages. The absence
of an admin GUI interface means that it is significantly more difficult to
have your blog stolen or hacked. The lack of Javascript and the relatively
simple CSS ensures the page loads fast and looks classy. The absence of
Javascript, or of commenting accounts, means that the blog is as
privacy-friendly as a blog can possibly be.

Finally, the blog code itself is small and (fairly) simple. This makes it
easy to deploy and, hopefully, easy to tinker with.

These advantages mean you can host a blog for very little. You can use a free
Heroku account, combined with S3 storage (which is cheap) for the CSS, and get
a blog that's totally under your control for almost nothing. If you want a
larger database than Heroku provides, you can always get an EC2 instance.
Right now I haven't, but in the longer term I'll be considering doing just
that. If I do, I'll probably blog about it.

### Great. So what?

So nothing, really. I wrote some software that I thought my be of interest to
others, so I released the non-site-specific stuff for others to use. If you
feel like you want a simple self-hosted blog, you should consider adapting
this framework for your own use. If you use it and find it is missing a
feature you think would be popular, or you need to make some changes to use it
on a production server of your own, I would be delighted to accept pull
requests. If all you want to do is clean the code up, I'd be happy to accept
pull requests for that too.

The code should be sufficiently well-contained that you can integrate it with
your own Django application as well!

### Where to?

In the long term, I'd like to add a spam filter for comments. I haven't
considered the best way to do this yet, so if you have any ideas or an
implementation you'd like me to consider, feel free to raise an issue on the
[GitHub page](https://github.com/Lukasa/minimalog). Otherwise, I'm probably
keeping the project minimal. But by all means, fork the project if you think
you can use it!


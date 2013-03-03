## collectr: Static File Management for All Of Us

A little while ago I wrote a blog post talking about
[Git hooks](http://lukasa.co.uk/2012/08/Git_Yer_Hooks_In/). As an example in
that post I wrote a post-commit hook that would minify and upload my static
files to S3.

This has spiralled a little bit out of control since then, and I finally drew
the line at maintaining that script. It was time to create something more
powerful. To this end I give you:
[collectr](https://github.com/Lukasa/collectr).

### About collectr

collectr has one goal: to control the management of static files. In its
current form it is a wrapper around the [boto](http://docs.pythonboto.org/)
library. It targets Amazon's S3 as the back end data store. Its sole purpose in
life is to prepare and upload static files to S3 according to the user's
wishes.

In the basic case, where all you want to do is upload some files in a directory
to S3, you can keep things simple:

    import collectr
    collectr.update('path/to/dir', 'bucket_name')

But collectr allows much more than that. In its current early state, collectr
supports a few other behaviours: it can minify files, it can ignore files in
the directory, and it can apply S3 metadata. A very simple API controls this
functionality. For instance, you can apply metadata:

    import collectr
    directory = collectr.StaticDir('path/to/dir')
    directory.metadata = {'Cache-Control': 'max-age=3600'}

You can ignore files:

    import collectr
    directory = collectr.StaticDir('path/to/dir')
    directory.ignore = ['.*\.json']

And you can minify.

### Minification

Minification has the most functionality at the moment. To minify files, you
want to provide a dictionary. The keys of this dictionary should correspond to
the file extensions, and the values correspond to the command-line you want to
run to minify the files. This allows collectr to run using whatever
minification tool is your favourite (or alternatively, whatever you have
installed). For example:

    import collectr
    directory = collectr.StaticDir('path/to/dir')
    directory.minifier = {'js': 'jsmin < {in_name} > {out_name}',
                          'css': 'yuicompressor -o {out_name} {in_name}'}

Each of the files in the directory is checked, and if it has a matching
extension, the command-line is executed. It executes in its own shell, so
globbing and redirection should work fine. All you need is to remember to put
in both an `in_name` and `out_name` field in the format string. The `out_name`
is the same as the name of the file, with '-min' appended to it. This will
likely become user-configurable in the future.

You might want to keep your pre-minified files in a separate directory. You can
do that: call the directory that contains the pre-minified files the input
directory like this:

    import collectr
    directory = collectr.StaticDir('path/to/dir')
    directory.input_directory = 'other/dir'
    directory.minifier = 'yuicompressor -o {out_name} {in_name}'

The `out_file` format string variable will move from the first directory to the
second, so if you use this feature don't forget to use `out_file`!

The above also shows the special-case where the minifier property is a string.
If you do this, collectr treats the string as a command-line to be applied to
both Javascript and CSS files. This is ideal for using with something like
[yuicompressor](http://yui.github.com/yuicompressor/).

You might have noticed that there's nothing here that specifies minification.
This is correct: in fact, you can do any form of pre-processing you like using
this method. Enjoy the freedom!

### Command-line tools

As a freebie, just to show a simple example of how you can replace your bash
scripts with something nicer, collectr comes with a simple script entitled
`collect_static` (hi there [Django](https://www.djangoproject.com/)!). This
script is intended to be as close a match as possible to the script I wrote for
my post-commit hook. It's roughly the same length, but the vast majority of it
is code for
[argparse](http://docs.python.org/2.7/library/argparse.html#module-argparse)
and some `if __name__ == '__main__'` stuff. In fact, there is only one line of
code to handle all of the rest. collectr is really that easy.

### The Future

Currently collectr is in version 0.0.4: that is to say, it's only just started!
I'll be working on this for the near future, and I'd like people's help as
well. If you have an idea for a feature, I want to hear it: even better if you
have a Pull Request that implements that feature. I'm looking at supporting
other back ends and providing more advanced functionality, so let me know what
you want from something like this and I'll provide it.

Feedback is always appreciated. Let me know what you think, and if you're using
collectr!

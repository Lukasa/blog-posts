## A Brief Digression About Logging

Logging is an interesting subject. For most developers, most of the time, we
don't think too hard about logging. Maybe we've got some coding standards that
mandate some minimal level of logging, but otherwise we're just not concerned
about how we log. But one day you'll find yourself five hours into a marathon
debugging session, throwing `print` statements in random places and thinking to
yourself "If only I'd put in some logging here, I'd have some idea of what was
going on in this system!"

So you turn to the Python
[`logging`](https://docs.python.org/2/library/logging.html), open the docs and
recoil in horror. All you want to do is log things! What's this nonsense about
loggers, and handlers, and LogRecords?

It's OK, I'm here to help.

### My Advice? Ignore It All.

The `logging` module is incredibly powerful, but as a result it's also
incredibly complicated. The reality is, most of us will never need to touch its
complexity.

Here's my promise. **The following lines of code will solve 95% of your logging
problems**.

#### Libraries

When you're writing a library and you want it to log, here's what you do. Find
your top-level `__init__.py` file and add the following two lines to it:

    import logging
    logging.getLogger(__name__).addHandler(logging.NullHandler())

This essentially enables the default logging behaviour: do nothing.

Then, in each file where you want to emit logs, add this line at the top:

    log = logging.getLogger(__name__)

That's it, you're done. Seriously. To log, you now just call the logging
methods: `log.debug()`, `log.info()`, `log.warning()`, `log.error()` and
`log.critical()`, depending on the severity of log you want.

#### Applications and Executables

What if you're an application developer, or you're writing the top-level
script? You know your libraries are logging, and you are too, but how do you
turn it on? Easy:

    import logging
    logging.basicConfig()

That's it. Done. You'll now get your default logging to `stderr`. Want to put
it somewhere else, or log at a different level?
[Here are the docs](https://docs.python.org/2/library/logging.html#logging.basicConfig).
Some highlights are logging at a specific level
(`logging.basicConfig(level=logging.DEBUG)`) and logging to file
(`logging.basicConfig(filename='myapp.log')`).

That's it.

### Guidelines

Remember, _never use the root logger_. Always call `.debug()` and friends on a
logger you got by calling `logging.getLogger(__name__)`.

Otherwise, there's really nothing else. If you need anything else in the
logging library, you probably already understand it.

See? Logging is easy.

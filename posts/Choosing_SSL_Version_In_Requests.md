## Choosing The SSL Version In Python Requests

Over the last few months (and probably for quite a while before then too), a
few issues have been raised on the
[Requests GitHub](https://github.com/kennethreitz/requests) page asking how to
select the version of SSL used by Requests. This is actually simple once you
know how, so I thought I'd write a short post to show you how it's done.

A quick note before we begin: **it is not possible to select the version of SSL
you want to use before Requests v1.0.0** without changing the underlying
library code. The following set of instructions will not work if you're running
an earlier version.

### How It's Done

Altogether this is relatively simple. To change the SSL version used in HTTPS,
you are expected to subclass the `HTTPAdapter` class and mount it to a
`Session` object. If, for example, you wanted to force the use of TLSv1, your
new Transport Adapter will look like this:

    from requests.adapters import HTTPAdapter
    from requests.packages.urllib3.poolmanager import PoolManager
    import ssl

    class MyAdapter(HTTPAdapter):
        def init_poolmanager(self, connections, maxsize, block=False):
            self.poolmanager = PoolManager(num_pools=connections,
                                           maxsize=maxsize,
                                           block=block,
                                           ssl_version=ssl.PROTOCOL_TLSv1)

With that done, you can mount it to a Requests `Session` object:

    import requests
    s = requests.Session()
    s.mount('https://', MyAdapter())

Of course, this is so easy that it's simple to write a Transport Adapter that
can take an arbitrary SSL type from the `ssl` package in the constructor and
use that. Whack this in a file and `import` it into whatever you're doing:

    from requests.adapters import HTTPAdapter
    from requests.packages.urllib3.poolmanager import PoolManager

    class SSLAdapter(HTTPAdapter):
        '''An HTTPS Transport Adapter that uses an arbitrary SSL version.'''
        def __init__(self, ssl_version=None, **kwargs):
            self.ssl_version = ssl_version

            super(SSLAdapter, self).__init__(**kwargs)

        def init_poolmanager(self, connections, maxsize, block=False):
            self.poolmanager = PoolManager(num_pools=connections,
                                           maxsize=maxsize,
                                           block=block,
                                           ssl_version=self.ssl_version)

You can mount it to a `Session` object and just go to town.

Hopefully this will be of use to people. If you have any problems or
improvements, leave a note in the comments or drop me a line on Twitter (the
url is in the sidebar).

**EDIT**: Fixed broken code in constructor.

**EDIT (22/06/13)**: Brought code up to date with Requests v1.2.3.

**EDIT (28/08/13)**: Actually brought code up to date with Requests v1.2.3.

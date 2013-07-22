## Requests: The Difference Between Params and Data

This question pops up a lot on Stack Overflow, on GitHub, and in the IRC
channel, so I thought I'd write a short post to address it. The question is,
broadly, this:

> How do I send data on a POST? I tried `params`, but that didn't work!

The answer is that Requests has two different arguments for 'sending' data on
a HTTP request: `params` and `data`. Each one does something different, but in
their simplest form they both accept a dictionary of keys and values.

`params` is all about the query string, and so is primarily used on GET
requests. To see it in action, take a look at this chunk of code:

    >>> params = {'hi': 'there', 'what': 'ho'}
    >>> r = requests.get('http://httpbin.org/get', params=params)
    >>> print r.request.url
    http://httpbin.org/get?hi=there&what=ho

The key take away here is that the data that was passed to `params` ended up in
the URL query string. This is what it's for. Any string data passed in there
will be correctly escaped and encoded, then added to the URL.

`data` works differently: it's all about the request body. Again:

    >>> data = {'hi': 'there', 'what': 'ho'}
    >>> r = requests.post('http://httpbin.org/post', data=data)
    >>> print r.request.body
    what=ho&hi=there

Why are two different parameters used? Why don't we just have `params` set the
request body on a POST, and the query string on anything else? The answer is
that you might want to use a query string _and_ a request body at the same
time. There _needs_ to be a different keyword.

So now you know: `params` for query string, `data` for request body. Simple.

## Writing A Persona Identity Provider

As I said in my [last post](//lukasa.co.uk/2013/04/Persona_Identity_Provider/),
I recently added Persona Identity Provider functionality to this blog. If you
follow me on Twitter (and really, why wouldn't you?), you might have noticed
that this wasn't an entirely smooth process.

Mozilla are a great organisation, and there are lots of great developers
working on the Persona project. However, the official documentation is of
varying quality. Large sections of it are very clear. Unfortunately, there are
significant chunks that range from incomplete to outright contradictory. Some
parts appear to be out of date. Other parts specify that they follow a
third-party specification, only to deviate from that specification without
apparent reasoning.

This makes implementing a complying Identity Provider something of a moving
target. If you're running a Node.js-based website then Mozilla kindly provide
a reference implementation, but otherwise the only really effective way of
working out what you're supposed to do in these ambiguous situations is to
undertake a code-reading of the Node.js reference implementation to see the
expected behaviour.

So I wanted to be as clear as possible about what you need to do to implement a
Persona IdP. I'll be focusing on Python, but as much as possible I'll try to
describe things in a generic, cross-platform kind of way.

### What You Need

You will need to implement the following services.

1. You will need to serve a browser support document. This document will be
   fetched by each website that asks you to verify someone's identity. It's a
   JSON-formatted document that contains a public key and the URL of two of the
   other services.
2. You will need to provide a login page. This login page will need to call
   some Persona-specific Javascript, so depending on how flexible your website
   is you may or may not be able to re-use one of your already existing login
   pages.
3. You will need to provide a 'provisioning' page. This will be loaded in a
   background frame during the login process, so don't worry about styling it.
   The provisioning page is basically just a glue page: it works out if the
   user is logged in, and tells the Persona API about it.
4. You will need a way to receive data from the browser and sign it. Bizarrely,
   the current Persona docs omit anything about this portion of the process.

For the rest of this article, I'm going to assume you already have a working
website. With that said, I'm also going to assume your website is pretty damn
basic: much like mine.

You'll need to serve the Persona pages over HTTPS. Given that you will have
to do this anyway, you should also take this opportunity to just
[serve your whole site over HTTPS](//lukasa.co.uk/2013/03/HTTPS_All_The_Things/).
This means you'll need to get an SSL certificate. I'm not going to go into
detail right now, but you can find plenty of guides about how to do so.

### The Support Document

Let's just get started. The very first thing you're going to want to do is to
generate an RSA key. Persona relies on
[public key cryptography](//en.wikipedia.org/wiki/Public_key_cryptography) for
authentication, and that means you need a key pair. If you're on Mac OS X or
Linux, you should already have [OpenSSL](http://www.openssl.org/) installed. No
guarantees for those of you on Windows. Make sure you have it, then run the
following two commands in a command shell:

    openssl genrsa -out private-key.pem 2048
    openssl rsa -in private-key.pem -pubout > public-key.pem

You now have two files. If you want, you can check your public key into your
revision control software of choice. **I strongly recommend you do not check
the private key in**, for security reasons.

Next, decide what URLs you want to serve your provisioning and login pages on.
For my website, I chose `/persona/provision/` and `/persona/sign_in/`
respectively.

When you have all that sorted, you've got everything you need to create your
support document. This _must_ be served from a specific URL (it's the only
thing you don't get to choose): specifically, `/.well-known/browserid/`.

The support document is a JSON-formatted page, which contains three pieces of
information: your public key (in a confusing format I'll describe shortly), the
URL for your login page and the URL for your provisioning page. To see my
version of this file, [click here](//lukasa.co.uk/.well-known/browserid/).

The JSON should look like this (the XXXX values are placeholders I'm using to
hide long strings of data):

    {
        "public-key": {
            "algorithm": "RS",
            "n": XXXX,
            "e": XXXX
        },
        "authentication": "/persona/sign_in/",
        "provisioning": "/persona/provision/"
    }

You should be able to see where to put the two URLs you just created, but
filling in the public key object is harder. To get the strings you need to put
in place of each XXXX, I recommend you run the following Python script on your
machine. It takes one argument: the path to your private key file.

    #!/usr/bin/env python
    import subprocess
    import sys

    def keyfield_as_int(field_name, data):
        start = data.index(field_name + ':') + 1
        index = start

        while (True):
            index += 1

            if not data[index].startswith(' '):
                break

        result = output[start:index]
        result = ''.join([x.strip() for x in result]).replace(':', '')

        return int(result, 16)

    def get_public_exponent(data):
        for element in data:
            if element.startswith('publicExponent'):
                return element.split(' ')[1]

    keyfile = sys.argv[1]
    proc = subprocess.Popen(['openssl', 'rsa', '-in', keyfile, '-text', '-noout'],
                            stdout=subprocess.PIPE)
    output = proc.communicate()[0].split('\n')

    modulus = keyfield_as_int('modulus', output)
    privateExponent = keyfield_as_int('privateExponent', output)
    publicExponent = get_public_exponent(output)

    print '\n'.join(['Modulus (n):', str(modulus),
                     'Public Exponent (e):', str(publicExponent),
                     'Private Exponent (d):', str(privateExponent)])

The script prints out three things, which are given the letters _n_, _e_ and
_d_. How exactly these relate to your private and public keys is left as an
exercise for the interested reader. What you care about right now is that this
script provides exactly the values for _n_ and _e_ in your support document.
Wrap them in quotes so they become a JSON string, and then set them in the
right place (one of the XXXX locations above). Keep hold of the value for _d_,
we'll need it later.

With that step finished, your support document should be done! Make sure you
serve it from the correct URL, with the correct `Content-Type` header set, and
then you're good to go.

### The Login Page

Let's tackle the login page next. This page should only respond to a HTTP GET
request. It is shown directly to the user, so you want to apply your CSS styles
to it so that the user can recognise it as yours. If you already have a login
page, I recommend you either extend or copy it.

This page needs to be able to do two things: tell if your user is already
logged in, and be able to log them in if they aren't. The second one of these
is left as an exercise to the reader, but your web framework of choice is
likely to provide you with useful tools. If you're using Django, I highly
recommend simply using the default authentication methods, it'll work great.

The reason you can't just use your normal login page is that you need some
extra stuff. Firstly, you need to include some more Javascript. Add this line
somewhere to your code (I recommend the header, against most performance
advice. You'll see why in a second):

    <script src="https://login.persona.org/authentication_api.js"></script>

After the place you add that line, you'll want to add some inline Javascript.
This should immediately call `navigator.id.beginAuthentication()`. This
function, provided by Persona, takes a single argument: a callback function
that itself takes only one argument, the email address being verified. Inside
the callback, you want to check whether the user is logged in. If they are,
you'll want to call `navigator.id.completeAuthentication()`.

(Personal side note: I actually did this confirmation server-side. If the user
had an active session cookie, I unconditionally called
`completeAuthentication()`. Do whatever is easiest for you.)

An example code block would be:

    navigator.id.beginAuthentication(function(email) {
        if userHasActiveSession(email) {
            navigator.id.completeAuthentication();
        }
    });

Here, `userHasActiveSession` would check whether the user in question has an
active login session with your domain.

If it turns out they do, the call to `navigator.id.completeAuthentication()`
will immediately route them away from your login page, so nothing will happen.
Otherwise, the user will be faced with your login page. They'll be asked to
log in, and then will be looped back to the beginning of the Persona
authentication process.

If you need to signal any failure, either an error or the user cancelling, just
call the `navigator.id.raiseAuthenticationFailure()` function. This takes a
single argument: a string indicating the reason for failure. Calling this
function will abort the Persona login process.

In some ways, this page is even easier to write than the support document. This
shouldn't take you too long, especially if you already have a login page.

### The Provisioning Page

This is the hardest of the actual web pages to build, and take heart my
friend, because it's not that hard either. This page is _not_ user-facing.
Persona will load this in an invisible frame, so all that matters is the
Javascript. Note, however, that you must allow this page to be served in a
frame! This means no setting the `X-Frame-Options` header to `DENY`!

This page needs to load some Javascript, and then execute some Javascript.
Begin by including the following:

    <script src="https://login.persona.org/provisioning_api.js"></script>

After adding that line, you'll want to add some inline Javascript. This should
call a function that then calls `navigator.id.beginProvisioning()`.
This function takes a callback: the callback takes two arguments, the user's
email address (a String) and the requested certificate duration (a Number).

Inside the callback, you need to work out whether the user has an active
session with your domain, and whether it's for the email address requested.
You can repeat the logic used in the **Login Page** for this.

**WARNING:** If the user has third-party cookies disabled, they won't send
their session cookie to this page. I consider this a failure case, and for the
sake of security I strongly recommend you do too.

If the user _doesn't_ have an active session, you will want to immediately call
`navigator.id.raiseProvisioningFailure()`. This takes an error message as its
first parameter, which you should provide. Calling this will abort the process
and pass the user to your login page.

If the user _does_ have an active session, you'll want to next call
`navigator.id.getKeyPair()`. This takes a callback: the callback takes one
argument, which is the _user's_ public key (a String).

Inside the callback for this function, you will need to send the user's public
key and the requested certificate duration (originally from
`navigator.id.beginProvisioning()`) to your server. Your server will then need
to generate what Mozilla calls a 'signed assertion'. We'll come back to how to
do this. In the meantime, let's just encapsulate that logic in a function
called `signServerSide()`.

Once you've got the signed certificate (passed by `signServerSide()` into a
callback), you will want to pass it as the only argument to
`navigator.id.registerCertificate()`. This instructs Persona to store the
assertion in the browser, allowing users to login as themselves in Persona
websites for up to the requested certificate duration.

Confused? Yeah, I would be too. An example code flow looks like this:

        <script type="text/javascript">
            navigator.id.beginProvisioning(function(email, certDuration) {
                if (email === <EMAIL_FROM_SESSION>) {
                    navigator.id.genKeyPair(function(publicKey) {
                        try{
                            signServerSide(email, publicKey, certDuration, function(certificate) {
                                navigator.id.registerCertificate(certificate);
                            });
                        } catch (e) {
                            navigator.id.raiseProvisioningFailure("unexpected error occurred");
                        }
                    });
                } else {
                    navigator.id.raiseProvisioningFailure("user is not authenticated as target user");
                }
            });
        </script>

I populate the `<EMAIL_FROM_SESSION>` space server-side, so it should be very
difficult to interrupt the execution of this code block. The server-side
signing code (we'll come back to it) also checks, though, so this is not a
single point of failure.

This is all your sign-in page needs to do. We'll tackle the `signServerSide()`
function next.

### Certificate Signing Part 1: The Client

We're going to break the certificate signing into two parts. The first part is
writing the `signServerSide()` function. This has a very simple role: it should
upload the user's certificate to the server, tell the server how long it should
sign for, and then receive the response back.

The way I recommend doing this is to use an HTTP POST. If you want, you can
form-encode everything into the body, but I found it was easiest to provide the
certificate duration (and the user's email) as parameters of the query string,
and to POST the (JSON-encoded) public key in the body.

You can do this however you want: if you're already using JQuery, you may as
well use it's XMLHttpRequest functionality. I'm going to show you a code
example that works with vanilla Javascript: no third-party libraries needed.

We begin by building up the URL. You'll have to select what the relative URL
for your signing service should be: I chose `/persona/sign/`. Your URL will be
that, with a query string appended to it. To build the query string you'll
want to urlencode the email address and the query duration (always sanitise
your inputs, guys), and build them into a query string.

You'll then want to create an `XMLHttpRequest` object and prepare it for
POSTing to your newly-build URL. When that's done, register an
`onreadystatechange` event handler. This handler should check whether the
request has completed, and if it has, call the callback with the returned data.
If the request fails, it should raise an exception.

Then, you'll want to set the Content-Type header and launch the request.

My code for doing all this is below:

    var signServerSide = function(email, publicKey, certDuration, callback) {
        var encEmail = encodeURIComponent(email);
        var encDuration = encodeURIComponent(certDuration);

        var request = new XMLHttpRequest();
        var url = "/persona/sign/?email=" + encEmail + "&duration=" + encDuration;
        request.open("POST", url, true);
        request.onreadystatechange = function() {
            if (request.readyState == 4) {
                if (request.status == 200) {
                    callback(request.responseText);
                } else {
                    throw {
                        name: "InvalidRequestError",
                        message: "Unexpected status code: " + request.status
                    };
                }
            }
        };

        request.setRequestHeader("Content-Type", "application/json; charset=utf-8");
        request.send(publicKey);
    };

This Javascript should be used on your **Provisioning Page**, as described in
the relevant section. Next, we implement the server-side portion of the logic.

### Certificate Signing Part 2: The Server

We need to implement some functionality on the server side of this, at the URL
you created in the previous section. The server has a job that is simple to
explain but not entirely simple to implement.

The signing page should verify that the user is logged in, and that the email
passed to it is the same as the email that is passed to it. The server should
then generate the expiry time of the new certificate. Once that's done, build
the JSON-encoded assertion, then sign it with your private key. Finally, return
all the data back in response.

How you do much of this is dependent on your web framework and programming
language. Your web framework should make it easy to verify that all the data
you expect has been passed to you, and that it's roughly the right shape. I
won't go into that portion of this stage in any detail. Instead, I'll talk
about some pitfalls you might encounter.

First, let's consider the query duration. The format of the assertion you're
going to build requires that you provide the expiry time, in milliseconds since
the epoch. All fairly standard. However, the query duration you've been passed
is in _seconds_. You will need to multiply this by 1000 before adding it to the
current time, which must also be in milliseconds past the epoch. This tripped
me up twice, which says rather more about me than it does about this problem.

Next, you'll need a signing routine. If you find you have to implement one of
these yourself I wish you all the best, because the specification is about as
clear as mud. However, Mozilla have got utility libraries for
[Node.js](https://github.com/mozilla/browserid) and
[Python](https://github.com/mozilla/PyBrowserID), at the very least. I'll
discuss the Python interface here.

You'll first need to build your private key. To do that, you need the function
`browserid.jwt.load_key()`. This takes two arguments. The first is a string
indicating the type of key you've used: in our case, it'll be `RS256`. The
second is a dictionary of keys to values. Each key is one of the letters
associated with the private key, as output by my helper script in the first
section: `e`, `n` and `d`. Their values are the numerical values output by that
same script.

The return value from this function is an object representing the private key.
You'll pass this into the signing function later.

Your next step is to build up the assertion dictionary. It's easiest for me to
show you how it looks in code:

    {
        'iss': X,
        'exp': Y,
        'public-key': Z,
        'principal': {'email': A}
    }

In the above:

- X is a string representing the issuer. In this case, it should be the domain
  you're signing for. For me, that was `lukasa.co.uk`.
- Y is a number representing the expiry date in milliseconds past the epoch.
  You should already have calculated this.
- Z is the user's public key. This is a JSON object, and you must not use a
  string here, so make sure you called json.loads() on the POSTed data.
- A is the email you're signing for. Again, this was passed in.

With that created, you will want to call `browserid.jwt.generate()`. This takes
two arguments: the first is the dict we just created, the second is the key
object.

The returned data is the string form of the assertion, to be returned to the
Javascript. You should simply return it back, with the Content-Type set to
`text/plain`.

My implementation of this is written for Django, and you can see it below, with
added comments for clarity:

    from django.http import (HttpResponseNotAllowed, HttpResponseForbidden,
                             HttpResponse, HttpResponseBadRequest)
    from django.views.decorators.csrf import csrf_exempt
    import time
    import os
    from browserid.jwt import generate, load_key

    @csrf_exempt
    def sign(request):
        """
        Handles the Persona signing page. This should only be POSTed to.
        Confirm that the user is authenticated and that their email matches the
        email in the query string. Sign using that email and the given duration.
        If user isn't authenticated, send 403. If user sends non-matching email,
        send 400. Otherwise, sign the certificate and return it.
        """
        # Validate some fields.
        if request.method != 'POST':
            return HttpResponseNotAllowed(['POST'])

        if not request.user.is_authenticated():
            return HttpResponseForbidden()

        # Save data from the query string.
        query_email = request.GET.get('email', None)
        query_duration = request.GET.get('duration', None)

        # Check the email address matches the one on the session and that the
        # query duration has been intelligently specified
        if ((request.user.email != query_email) or (not query_duration)):
            return HttpResponseBadRequest()

        # Turn the query_duration into a number.
        query_duration = float(query_duration)

        # Check we did get some data from the request.
        if len(request.raw_post_data) == 0:
            return HttpResponseBadRequest()

        # Expiry time must be in ms past the epoch, not seconds past it.
        expiry_time = int(time.time() + float(query_duration)) * 1000

        # Create the signing key.
        exponent = os.environ['PERSONA_E']
        modulus = os.environ['PERSONA_N']
        private_modulus = os.environ['PERSONA_D']
        key = load_key('RS256', {'e': exponent,
                                 'n': modulus,
                                 'd': private_modulus})

        # Get the user's public key.
        public_key = json.loads(request.raw_post_data)

        # Build a JWT dict. The documentation for this is pretty sparse, but I
        # gather this is what it's supposed to look like.
        data = {'iss': 'lukasa.co.uk',
                'exp': expiry_time,
                'public-key': public_key,
                'principal': {'email': request.user.email}}

        # Sign it.
        signed_data = generate(data, key)

        return HttpResponse(signed_data, content_type='text/plain')

Hopefully you can adapt this for use on your web framework.

### All Done!

See? It's simple!

OK, not entirely simple, but fairly simple. Four steps, not too complicated,
and you're there. For most people it should be possible to implement all of
this is an afternoon.

If you don't have a useful Persona library, though, you might find this takes
longer. You'll need to read up on the spec for the signed assertion. It's not
entirely clear, but I think I've got my head around it. If you find you need
help with it, I recommend dropping me a line on Twitter. If I can't help you,
I'll point you to the real masters, like
[Dan Callahan](https://twitter.com/callahad).

Let me know if you have problems or comments on Twitter, or in the comments.

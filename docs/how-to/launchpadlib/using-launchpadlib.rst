Get started with launchpadlib
=============================

This document shows how to use a Python client to read and write Launchpad's
data using the launchpadlib library. It doesn't cover the HTTP requests and
responses that go back and forth behind the scenes: for that, see `the
"hacking" document <../Hacking>`__. This document also doesn't cover the full
range of what's possible with Launchpad's web service: for that, see the `web
service reference documentation <http://launchpad.net/+apidoc/>`__. Check out
the `API examples page <API/Examples>`__ if you would like to see more sample
code.

Launchpad's web service currently exposes the following major parts of
Launchpad:

-  People and teams
-  Team memberships
-  Bugs and bugtasks
-  The project registry
-  Hosted files, such as bug attachments and mugshots.

As new features and capabilities are added to the web service, you'll be able
to access most of them without having to update your copy of launchpadlib. You
*will* have to upgrade launchpadlib to get new client-side features (like
support for uploaded files). The Launchpad team will put out an announcement
whenever a server-side change means you should upgrade launchpadlib.

Installation
------------

Install on Ubuntu
^^^^^^^^^^^^^^^^^

If you have the latest version of Ubuntu then to install the launchpadlib
available in the Ubuntu repositories, open a terminal and run the command:

::

     sudo apt-get install python3-launchpadlib

And you are done!

If you have an older version of Ubuntu then parts of the instructions below may
not work with the version from the repositories but you should be able to
install the latest version of launchpadlib manually.

Install using pip
^^^^^^^^^^^^^^^^^

On any platform with a working Python and pip, you should be able to
say:

::

     pip install launchpadlib

Getting started
---------------

The first step towards using Launchpad's web service is to choose a cache
directory. The documents you retrieve from Launchpad will be stored here, which
will save you a lot of time. Run this code in a Python session, substituting an
appropriate directory on your computer:

::

       cachedir = "/home/me/.launchpadlib/cache/"

The next step is to set up credentials for your client. For quick read-only
access to Launchpad data, you can get anonymous access.  Otherwise, you'll need
to authenticate with Launchpad using OAuth.

Anonymous access
^^^^^^^^^^^^^^^^

The \`Launchpad.login_anonymously()\` method will give you automatic read-only
access to public Launchpad data.

::

       from launchpadlib.launchpad import Launchpad
       launchpad = Launchpad.login_anonymously('just testing', 'production', cachedir, version='devel')

The first argument to \`Launchpad.login_anonymously()\` is a string that
identifies the web service client. We use this string to gauge client
popularity and detect buggy or inefficient clients. Here, though, we're just
testing.

The second argument tells launchpadlib which Launchpad instance to run against.
Here, we're using \`production`, which is mapped to the web service root on the
production Launchpad server: "https://api.launchpad.net/". Anonymous access
cannot change the Launchpad dataset, so there's no concern about a bad test
program accidentally overwriting data. (If you want to play it safe, you could
use 'staging' instead - though staging is sometimes down for extended periods.)

The \`version\` argument specifies the API version to use. For historical
reasons the default is \`'1.0'\`, and you may want to use this in certain
special circumstances, but in most cases you should use \`'devel'\` so that you
get a reasonably complete and current interface.

The \`login_anonymously()\` method automatically negotiates a read-only
credential with Launchpad. You can use your \`Launchpad\` object right away.

::

       bug_one = launchpad.bugs[1]
       print(bug_one.title)
       # Microsoft has a majority market share

You'll get an error if you try to modify the dataset, access private
data, or access objects like \`launchpad.me\` which assume a particular
user is logged in.

Note that \`login_anonymously()\` is only available in launchpadlib
starting in version 1.5.4.

Authenticated access
^^^^^^^^^^^^^^^^^^^^

To get read-write access to Launchpad, or to see a user's private data, you'll
need to get an OAuth credential for that user. If you're writing an application
that the user will run on their own computer, this is easy: just call the
\`Launchpad.login_with()\` method.

This method takes two important arguments: the name of your application, and
the name of the Launchpad server you want to run against. The default server
name is 'staging', so that you don't destroy data by accident while developing.
When you do a release, you can change this to 'production'.

::

       from launchpadlib.launchpad import Launchpad
       launchpad = Launchpad.login_with('My Application', 'staging', version='devel')

*If this code complains that 'staging' isn't a URL, then you're not running the
most up-to-date launchpadlib. You can replace 'staging' with
\`launchpadlib.launchpad.STAGING_SERVICE_ROOT\` to make it work in Ubuntu
9.10's launchpadlib.*

If you have an existing desktop-wide Launchpad credential, launchpadlib will
find it and use it. If there's no existing desktop credential (because you've
never used a launchpadlib application on this computer, or because you had a
credential that expired), launchpadlib will guide you through authorizing a new
credential, as seen `here <API/ThirdPartyIntegration>`__.

There's one other important argument to \`login_with()`. You can pass in a
callback function as \`credential_save_failed`. That function will be invoked
if a desktop credential can't be created--either because the end-user refused
to perform the authorization, or because there was a problem storing the
credential after authorization.

::

       import sys
       from launchpadlib.launchpad import Launchpad

       def no_credential():
           print("Can't proceed without Launchpad credential.")
       sys.exit()

       launchpad = Launchpad.login_with(
           'My Application', 'staging', credential_save_failed=no_credential, version='devel')

Authenticated access for website integration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Things are a little more difficult if you want to integrate Launchpad's
functionality into your own website. You can't call \`Launchpad.login_with()`,
because that will open up a web browser on your webserver--not on your user's
computer.

Instead, you create a \`Credentials\` object identifying your website and call
\`get_request_token()\` to ask Launchpad for a request token.  Be sure to pass
in the name of the Launchpad server you want to use (probably "production" as
\`web_root`.

::

       from launchpadlib.credentials import Credentials
       credentials = Credentials("my website")
       request_token_info = credentials.get_request_token(web_root="production")

You'll get back a string that looks like
'https://launchpad.net/+authorize-token?oauth_token=...' This is the URL your
end-user needs to visit in order to authorize your token.

At this point, you should redirect your user to that URL. Then, start
periodically calling \`exchange_request_token_for_access_token()`:

::

       from lazr.restfulclient.errors import HTTPError
       complete = False
       while not complete:
           try:
               credentials.exchange_request_token_for_access_token(
               web_root="production")
           complete = True
           except HTTPError:
           # The user hasn't authorized the token yet.

Once \`exchange_request_token_for_access_token()\` successfully executes, an
authorized access token will be present in \`credentials.access_token`. You can
then pass the \`Credentials\` object into the \`Launchpad\` constructor.

::

       from launchpadlib.launchpad import Launchpad
       launchpad = Launchpad(credentials, service_root="production")

While this system is not ideal, we don't know of any third-party websites that
are integrating Launchpad functionality in a way that requires OAuth tokens.

Getting help
^^^^^^^^^^^^

If you don't know the capabilities of one of the objects you've got, you can
call dir() on it. You'll see all of its fields and all the custom methods it
supports. Unfortunately, you'll also see a bunch of launchpadlib-specific junk
that you don't care about. That's why we've made available these four lists:

-  \`lp_attributes`: Data fields of this object. You can read from these
   might be able to write to some of them.
-  \`lp_collections`: List of launchpad objects associated with this
   object.
-  \`lp_entries`: Other Launchpad objects associated with this one.
-  \`lp_operations`: The names of Launchpad methods you can call on the
   object.

.. raw:: html

   <!-- end list -->

::

      print(sorted(bug_one.lp_attributes))
      # ['date_created', 'date_last_message', 'date_last_updated', ... 'tags', 'title']
      print(sorted(bug_one.lp_operations))
      # ['addAttachment', 'addWatch', 'subscribe', 'unsubscribe']

If you need more detailed help, you can look the object up in `the reference
documentation <http://launchpad.net/+apidoc>`__. First, find out the type of
the object.

::

       print(repr(bug_one))
       # <bug at https://api.staging.launchpad.net/beta/bugs/1>

This is a 'bug' type object. Now you use the type of the object as an anchor
into the reference documentation. To find out the capabilities of this object
and what data is stored inside it, you'd visit
https://api.launchpad.net/devel.html#bug.

As you'll see, the reference documentation still needs some work, and it's
geared more towards web service hackers than launchpadlib users, but it will
tell you about all of this object's attributes and all the supported
operations.

-  The "Default representation" section tells you about the available
   attributes.

.. raw:: html

   <!-- end list -->

-  The "Custom POST methods" and "Custom GET methods" sections tell you
   about methods the object supports other than the default methods
   described below. The methods take whatever parameters are listed in
   "Request query parameters". (You can ignore the "ws.op" parameter
   because you're using launchpadlib; that's just the name of the
   method.)

The top-level objects
---------------------

The Launchpad object has attributes corresponding to the major parts of
Launchpad. These are:

-  \`.bugs`: All the bugs in Launchpad
-  \`.people`: All the people in Launchpad
-  \`.me`: You
-  \`.distributions`: All the distributions in Launchpad
-  \`.projects`: All the projects in Launchpad
-  \`.project_groups`: All the project groups in Launchpad

As a super special secret, \`distributions`, \`projects\` and
\`project_groups\` are all actually the same thing.

::

       me = launchpad.me
       print(me.name)
       # This should be your user name, e.g. 'salgado'

The \`launchpad.people\` attribute gives you access to other people who use
Launchpad. This code uses launchpad.people to look up the person with the
Launchpad name "salgado".

::

       people = launchpad.people
       salgado = people['salgado']
       print(salgado.display_name)
       # Guilherme Salgado

You can search for objects in other ways. Here's another way of finding
"salgado".

::

       salgado = people.getByEmail(email="guilherme.salgado@canonical.com")
       print(salgado.display_name)
       # Guilherme Salgado

Some searches return more than one object.

::

       for person in people.find(text="salgado"):
           print(person.name)
       # agustin-salgado
       # ariel-salgado
       # axel-salgado
       # bruno-salgado
       # camilosalgado
       # ...

Note that, unlike typical Python methods, these methods--`find()\` and
\`getByEmail()`--don't support positional arguments, only keyword arguments.
You can't call \`people.find("salgado")`; it has to be
\`people.find(text="salgado")`.

Entries
-------

Bugs, people, projects, team memberships, and most other objects published
through Launchpad's web service, all work pretty much the same way. We call all
these objects "entries". Each corresponds to a single piece of data within
Launchpad.

You can use the web service to discover various facts about an entry.  The
launchpadlib makes the facts available as attributes of the entry object.

\`name\` and \`display_name\` are facts about people.

::

       print(salgado.name)
       # salgado

       print(salgado.display_name)
       # Guilherme Salgado

\`private\` and \`description\` are facts about bugs.

::

       print(bug_one.private)
       # False

       print(bug_one.description)
       # Microsoft has a majority market share in the new desktop PC marketplace.
       # This is a bug, which Ubuntu is designed to fix.
       # ...

Some of an object's attributes are links to other entries. Bugs have an
attribute \`owner`, but the owner of a bug is a person, with attributes of its
own.

::

       owner = bug_one.owner
       print(repr(owner))
       # <person at https://api.staging.launchpad.net/beta/~sabdfl>
       print(owner.name)
       # sabdfl
       print(owner.display_name)
       # Mark Shuttleworth

If you have permission, you can change an entry's attributes and write the data
back to the server using \`lp_save()`.

::

       me = people['my-user-name']
       me.display_name = 'A user who edits through the Launchpad web service.'
       me.lp_save()

       print(people['my-user-name'].display_name)
       # A user who edits through the Launchpad web service.

Having permission means not only being authorized to perform an operation on
the Launchpad side, but using a launchpadlib \`Credentials\` object that
authorizes the operation. If you've set up your launchpadlib Credentials for
read-only access, you won't be able to change data through launchpadlib.

self_link: the permanent ID
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Every entry has a \`self_link\` attribute. You can treat this as a permanent ID
for the entry. If your program needs to keep track of Launchpad objects across
multiple runs, a simple way to do it is to keep track of the \`self_link`s.

::

       print(salgado.self_link)
       # https://api.staging.launchpad.net/beta/~salgado

       bug_one.self_link
       # https://api.staging.launchpad.net/beta/bugs/1

web_link: the link to the Launchpad website
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Most of the entries published by the web service correspond to pages on
the Launchpad website. You can get the website URL of an entry with the
\`web_link\` attribute:

::

       print(salgado.web_link)
       # https://staging.launchpad.net/~salgado

       bug_one.web_link
       # https://bugs.staging.launchpad.net/bugs/1

Some entries don't correspond to any page on the Launchpad website:
these entries won't have a \`web_link`.

Named operations
^^^^^^^^^^^^^^^^

Entries can support special operations just like collections, but again note
that, these methods don't support positional arguments, only keyword arguments.

Errors
------

When the Launchpad web service encounters an error, it sends back an error
message to launchpadlib, which raises an HTTPError exception.  You'll see
information about the HTTP request that caused the error, and the server-side
error message. Depending on the error, you may be able to recover or change
your code and try again.

If you're using an old version of launchpadlib, the HTTPError may not be this
helpful. To see the server-side error message, you'll need to print out the
.content of the HTTPError exception.

::

   #!python
   try:
       failing_thing()
   except HTTPError as http_error:
       print(http_error.content)

Collections
-----------

When Launchpad groups similar entries together, we call it a collection.
You've already seen one collection: the list of people you get back when you
call launchpad.people.find.

::

       for person in launchpad.people.find(text="salgado"):
           print(person.name)

That's a collection of \`people`-type entries. You can iterate over a
collection as you can any Python list.

Some of an entry's attributes are links to related collections. Bug #1 has a
number of associated bug tasks, represented as a collection of 'bug task'
entries.

::

       tasks = bug_one.bug_tasks
       print(len(tasks))
       # 17
       for task in tasks:
           print(task.bug_target_display_name)
       # Computer Science Ubuntu
       # Ichthux
       # JAK LINUX
       # ...

The person 'salgado' understands two languages, represented here as a
collection of two \`language\` entries.

::

       for language in salgado.languages:
           print(language.self_link)
       # https://api.staging.launchpad.net/beta/+languages/en
       # https://api.staging.launchpad.net/beta/+languages/pt_BR

Because collections can be very large, it's usually a bad idea to iterate over
them. Bugs generally have a manageable number of bug tasks, and people
understand a manageable number of languages, but Launchpad tracks over 250,000
bugs. If you just iterate over a list, launchpadlib will just keep pulling down
entries until it runs out, which might be forever (or, realistically, until
your client is banned for making too many requests).

That's why we recommend you slice Launchpad's collections into Python lists,
and operate on the lists. Here's code that prints descriptions for the 10 most
recently filed bugs.

::

       bugs = launchpad.bugs[:10]
       for bug in bugs:
           print(bug.description)

For performance reasons, we've put a couple restrictions on collection slices
that don't apply to slices on regular Python lists. You can only slice from the
beginning of a collection, not the end.

::

       launchpad.bugs[-5:]
       # *** ValueError: Collection slices must have a nonnegative start point.

And your slice needs to have a definite end point: you can't slice to the end
of a collection.

::

       bugs[10:]
       # *** ValueError: Collection slices must have a definite, nonnegative end point.

       bugs[:-5]
       # *** ValueError: Collection slices must have a definite, nonnegative end point.

On the plus side, you can include a step number with your slice, as with a
normal Python list:

::

       every_other_bug = launchpad.bugs[0:10:2]
       len(every_other_bug)
       # 5

Hosted files
------------

Launchpad stores some data in the form of binary files. A good example is
people's mugshots. With launchpadlib, you can read and write these binary files
programatically.

If you have a launchpadlib reference to one of these hosted files, you can read
its data by calling the \`open()\` method and treating the result as an open
filehandle.

::

       mugshot = launchpad.me.mugshot
       mugshot_handle = mugshot.open()
       mugshot_handle.read()
       # [binary data]
       mugshot_handle.content_type
       # 'image/jpeg'
       mugshot_handle.last_modified
       # 'Wed, 12 Mar 2008 21:47:05 GMT'

You'll get an error if the file doesn't exist: for instance, if a person
doesn't have a mugshot.

::

       launchpad.people['has-no-mugshot'].mugshot
       # *** HTTPError: HTTP Error 404: Not Found

To create or overwrite a file, open the hosted file object for write.  You'll
need to provide the access mode ("w"), the MIME type of the file you're sending
to Launchpad, and the filename you want to give it on the server side.

::

       with mugshot.open("w", "image/jpeg", "my-image.jpg") as mugshot_handle:
           mugshot_handle.write("image data goes here")

If there's something wrong--maybe you provide a file of the wrong type--you'll
get an \`HTTPError\` with a status code of 400. The \`content\` attribute will
contain an error message.

::

       print(http_error.content)
       # This image is not exactly 192x192 pixels in size.

       print(http_error.content)
       # The file uploaded was not recognized as an image; please
       # check it and retry.

Persistent references to Launchpad objects
------------------------------------------

Every entry and collection has a unique ID: its URL. You can get this unique ID
by calling \`str()\` on the object.

::

       print(str(bug_one))
       # https://api.staging.launchpad.net/beta/bugs/1

If you need to keep track of Launchpad objects over time, or pass references to
Launchpad objects to other programs, use these strings. If you've got one of
these strings, you can turn it into the corresponding Launchpad object by
calling \`launchpad.load()`.

::

       bug_one = launchpad.load("https://api.staging.launchpad.net/beta/bugs/1")
       print(bug_one.title)
       Microsoft has a majority market share

You're bookmarking the Launchpad objects and coming back to them later, just
like you'd bookmark pages in your web browser.

Three things to make your client faster
---------------------------------------

1. **Use the latest launchpadlib.** (The versions in the current Ubuntu
release should be fine; otherwise run from the branch or the latest
tarball.)

2. **Profile:**

::

       import httplib2
       httplib2.debuglevel = 1

3. **Fetch objects only once:**

Don't do this:

::

       if bug.person is not None:
           print(bug.person.name)

instead

::

       p = bug.person
       if p is not None:
           print(p.name)

(From `the
blog <http://blog.launchpad.net/api/three-tips-for-faster-launchpadlib-api-clients>`__).

Planned improvements
--------------------

launchpadlib still has deficiencies. We track bugs in the launchpadlib
bug tracker (https://bugs.launchpad.net/launchpadlib) and will be
working to improve launchpadlib throughout the limited beta.

See also
^^^^^^^^

-  `web service reference
   documentation <http://launchpad.net/+apidoc/>`__ for a list of all
   objects, operations, etc

launchpadlib API compatibility
------------------------------

The API compatibility of the launchpadlib Python library has not always
been maintained as well as an author of a program using it would hope.

== >= 1.5.5 (>= Ubuntu Lucid Lynx) == Version 1.5.5 added the support
for accessing various different versions of the remote web-service API
(at the time of writing, these versions are known as "beta", "1.0" and
"devel"). Whilst the new \`version\` parameters were compatibly added,
there was an incompatible change to the URLs that launchpadlib would
accept as \`service_root\` parameters:

-  launchpadlib 1.5.4 and earlier **requires** URLs of the form
   \`https://api.launchpad.net/beta/\`
-  launchpadlib 1.5.5 and later **requires** URLs of the form
   \`https://api.launchpad.net/\`

and either will break in non-obvious ways if you give it the wrong form.

1.8.0 (no Ubuntu release)
^^^^^^^^^^^^^^^^^^^^^^^^^

Version 1.8.0 changed things, which 1.9.0 then changed again. And, it
never went into any final Ubuntu release. Probably best to just pretend
it doesn't exist.

== >= 1.9.0 (>= Ubuntu Natty Narwhal) == Version 1.9.x's changes versus
1.6.x include a major refactor of how authentication tokens are
obtained. Notable consequences:

-  Different kinds of tokens are obtained, and they are stored
   differently by default (in GNOME keyring or similar technologies
   instead of files), meaning it's highly unlikely that tokens stored by
   launchpadlib 1.6.x will be noticed by 1.9.x, so users will have to
   re-authorize.
-  Most of the methods by which a \`Launchpad\` object is obtained have
   changed significantly: *(Note, positional parameter indices referred
   to below are 1-based)*

**Launchpad.__init__:**

-  Parameters \`authorization_engine\` and \`credential_store\` inserted
   at position 2. **It is no longer safe to call \`Launchpad.__init__\`
   with positional parameters beyond the first in compatible
   applications!**

**Launchpad.login:**

-  Method is now deprecated.
-  Five new parameters inserted at position 8. Mitigating factor: the
   o[Inly parameter after this was \`version`, which was probably being
   passed as a keyword argument anyway.

**Launchpad.get_token_and_login:**

-  Method is now deprecated.
-  Parameter 6 renamed from \`authorizer_class\` to
   \`authorization_engine`.
-  Parameters \`credential_store\` and \`credential_save_failed\`
   inserted at position 9. Mitigating factor: the only parameter after
   this was \`version`, which was probably being passed as a keyword
   argument anyway.

**Launchpad.login_with:**

-  Positional parameter 1 changed from \`consumer_name\` to
   \`application_name\` to attempt to force common use-cases to acquire
   a desktop integration rather than consumer-specific token without
   code changes.
-  Parameter 6 renamed from \`authorizer_class\` to
   \`authorization_engine`.
-  New parameter \`consumer_name\` appended to replace the incarnation
   removed at position 1. (But it does not actually work - see
   https://launchpad.net/bugs/755313)
-  New parameters \`credential_save_failed\` and \`credential_store\`
   appended.


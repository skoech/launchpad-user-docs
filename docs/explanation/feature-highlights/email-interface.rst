
Launchpad's email interface
===========================

Launchpad's bug tracker sends you `email <Bugs/Subscriptions>`__ about
the bugs you're interested in. If you see something that requires your
attention - for example, you want to comment on a bug - rather than
leaving your email client to fire up a web browser, all you need to do
is reply to the email.

It's not just limited to replying to bug notifications, though. The bug
tracker's email interface allows you to do just about everything you can
in the web interface. Within time, you may find that email is the main
way you interact with the bug tracker.

Before you start
----------------

Launchpad verifies incoming email by looking for a GNU Privacy Guard
(GPG) signature by the sender, or a DKIM signature by a trusted sender
domain, such as GMail. Read our `guide on adding your GPG
key <YourAccount/ImportingYourPGPKey>`__ to your Launchpad account.

Messages that just add comments to a bug or merge proposal are not
required to be signed. Messages that contain commands to change the
state of an object do need to be signed.

Anatomy of an email to the bug tracker
--------------------------------------

Let's look at the elements of a bug report email:

-  **From address:** the address from which you send the email must be
   `registered in your Launchpad
   account <https://launchpad.net/people/+me/+editemails>`__.
-  **To address:** new@bugs.launchpad.net for new bugs;
   \```bugnumber@bugs.launchpad.net``\` to manipulate an existing bug
   report; \```edit@bugs.launchpad.net``\` for bulk edits.
-  **Subject:** Launchpad uses this as the bug report or comment
   summary.
-  **Email body:** the text of your email forms the bug report or
   comment detail. This is also where you can supply commands to
   manipulate the bug.

That last item, the email body, needs a little more explanation. When
you want to use one of the email interface's commands, **you need to
start the line with a space**. Otherwise, Launchpad will treat your
command as a comment only and not as a command.

.. note::
    All commands are also posted as a comment to the bug. This is something we plan to fix.

Managing bugs through email
---------------------------

If you prefer, you can entirely manage bugs using email.

Using the email interface you can:

-  file a new bug
-  comment on an existing bug
-  assign a bug to someone
-  change the status of a bug
-  record that a bug affects a different community.

Email notification of new bugs in a project or package
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can ask Launchpad to email you every time someone reports a bug
against a project or package that interests you, by registering as a bug
contact for that project or package.

By replying to the new bug notification email, you can assign the bug to
a Launchpad user or change its status, or just comment on the bug. For
the details of how to use the email interface, please read our guide to
UsingMaloneEmail.

Subscribing to specific bugs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you are particularly interested in a specific bug, you can subscribe
to it. Launchpad will email you every time someone comments on the bug
or changes its status. If you reply to any of these emails, your
comments will be sent to all of the other bug subscribers, and presented
on the web site too. This is the best way to keep up to date with the
progress of a bug.

Of course email notifications also work with remote bug watches. When
you create a bug watch linking the Launchpad bug to a report in another
community's bug tracker, Launchpad will notify all subscribers when
there is a change to the remote bug's status. This is an excellent way
to keep track of the big picture of community work on a bug.

Using teams to track bugs
~~~~~~~~~~~~~~~~~~~~~~~~~

Sometimes, a group of people may want to receive email notifications for
a set of packages, or projects. With Launchpad, you can create a team
and either subscribe that team to the bug or make the team a bug
contact.

Launchpad will email everyone in that team every time a new bug is filed
against the project or package.

You can find out more about Launchpad teams in the next step on our
tour, `Understanding Launchpad
teams <FeatureHighlights/TeamManagement>`__.

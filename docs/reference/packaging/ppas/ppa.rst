Personal Package Archive
========================

Using a Personal Package Archive (PPA), you can distribute software and
updates directly to Ubuntu users. Create your source package, upload it
and Launchpad will build binaries and then host them in your own apt
repository.

That means Ubuntu users can install your packages in just the same way
they install standard Ubuntu packages and they'll automatically receive
updates as and when you make them.

Every individual and team in Launchpad can have one or more PPAs, each
with its own URL.

Packages you publish in your PPA will remain there until you remove
them, they're superseded by another package that you upload or the
version of Ubuntu against which they're built becomes obsolete.

.. note::
    `CommercialHosting <CommercialHosting>`__ allow you to have private PPAs.

.. note:: 
    We do not allow uploading pre-built binary packages.

Size and transfer limits
------------------------

New PPAs get 8 GiB of disk space. If you need more space for a
particular PPA, `ask us <https://answers.launchpad.net/soyuz>`__.

While we don't enforce a strict limit on data transfer, we will get in
touch with you if your data transfer looks unusually high.

Supported architectures
-----------------------

When Launchpad builds a source package in a PPA, by default it creates
binaries for:

-  `x86 <http://en.wikipedia.org/wiki/X86>`__
-  `AMD64 <http://en.wikipedia.org/wiki/AMD64>`__

You may also request builds for arm64, armhf, ppc64el, and/or s390x. Use
the "Change details" page for the PPA to enable the architectures you
want.

Changing the set of architectures for which a PPA builds does not create
new builds for source packages that are already published in that PPA;
it only affects which builds will be created for new uploads. If you
need to create builds for newly-enabled architectures without
reuploading, go to "View package details" and then "Copy packages",
select all the packages for which you want to create builds, select
"This PPA", "The same series", and "Copy existing binaries", and submit
the form using the "Copy Packages" button.

We use OpenStack clouds for security during the build process,
ensuring that each build has a clean build environment and different
developers cannot affect one another's builds accidentally. These clouds
do not yet have support for the powerpc and s390x architectures; when
they do, it will also be possible to request those architectures in
PPAs.

Supported series
----------------

When building a source package you can specify one of the supported
series in your `changelog
file <http://packaging.ubuntu.com/html/debian-dir-overview.html#the-changelog>`__
which are listed at `the Launchpad PPA
page <https://launchpad.net/ubuntu/+ppas>`__.

If you specify a different series the build will fail.

Supporting multiple series
--------------------------

If you want to provide a package for multiple series, you can do one of
the following:

-  build the package for the oldest of the releases you care about, then
   once the build has finished and been published, `copy <https://help.launchpad.net/Packaging/PPA/Copying>`__ the binaries forward to the newer releases
   (e.g. build for bionic and then publish those binaries for focal and jammy as well).

-  create one source package version per release. People taking the "one source package version per release" approach often end up automating it
using  :literal:`source`   ``package``   `recipes <https://help.launchpad.net/Packaging/SourceBuilds>`__.

Activating a PPA
----------------

Before you can start using a PPA, whether it's your own or it belongs to
a team, you need to activate it on your `profile
page <https://launchpad.net/people/+me/>`__ or the team's overview page.
If you already have one or more PPAs, this is also where you'll be able
to create additional archives.

Your PPA's key
~~~~~~~~~~~~~~

Launchpad generates a unique key for each PPA and uses it to sign any
packages built in that PPA.

This means that people downloading/installing packages from your PPA can
verify their source. After you've activated your PPA, uploading its
first package causes Launchpad to start generating your key, which can
take up to a couple of hours to complete.

Your key, and instructions for adding it to Ubuntu, are shown on the
PPA's overview page.

Deleting a PPA
--------------

When you no longer need your PPA, you can delete it. This deletes all of
the PPA's packages, and removes the repository from
\`ppa.launchpad.net`. You'll have to wait up to an hour before you can
recreate a PPA with the same name.

Upload permissions
------------------

Upload permissions are usually bound to the ppa-owning team. If you want
to grant upload permissions to somebody, but you do not want to add them
to the team, you can use the
https://code.launchpad.net/ubuntu-archive-tools as following:

::

   edit-acl -A ppa:OWNER/ubuntu/ARCHIVE -p PERSON -c main -t upload add

Further information
-------------------

You can familiarise yourself with how PPAs work by `installing a package
from an existing PPA <Packaging/PPA/InstallingSoftware>`__. You can also
jump straight into `uploading your source
packages <Packaging/PPA/Uploading>`__.
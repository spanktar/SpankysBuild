Introduction
============
This is an example buildout that uses multiple files to achieve a robust,
reusable, multi-environment setup for Zope, ZEO, and Plone.  It allows you to
have base configs that you can then pick and choose from.

Files
-----

All of the files that are prefixed with "run" are meant to be passed to
buildout as a config file using the "-c" flag.  Using this prefix allows for
nice sorting and tab completion at the command line.

The regular 'buildout.cfg' file is used for local development and is
mostly self contained.

Environments
------------

This buildout is set up so that you can have multiple similar environments
running either alongside each other on the same devices (virtual or otherwise),
or on separate devices.

For example, we have 3 development environments running the full stack
(Zope and ZEO) running on the same box. These are called dev01/02/03.

Then we have some QA environments that exist as separate devices. They are
broken into separate Zope and ZEO servers.

Finally we have our Stage and Production environments that are also separated
onto dedicated hardware. Production servers are multi-homed (not shown here).

For environments that are separated, you'll note that there is a separate
runfile for Zope and for ZEO, because obviosuly they are run separately on
different machines.  The ZEO runfiles have a "-zeo" suffix.

Features
--------

These buildouts show off a few features you might find useful:

*Setting environment variables*
    In the Zope part, I set some environment variables that then can be used
    from within Plone.  For example, we set the API server name, used when my
    code needs to call the backend:

    -- environment-vars = API_SERVER ${environ:api_server}

*Custom logging*
    We log locally, and to a remote syslog server.  This is set up by using the
    "event-log-custom" directive in the Zope part and adding a <syslog> stanza.

*Beaker*
    We use beaker for shared session management across Zope instances.  You can
    see this setup in the "zope-conf-additional" directive.  Note that we use a
    central memcached server to store the sessions.

*Supervisor*
    We use supervisor for all daemonization.  Not sure what would happen if you
    enabled zope- and zeo-supervisor on the same box.  My guess is that they
    would overwrite each other.

*Backups*
    I am now using a custom branch of collective.recipe.backup (awaiting a
    pull request response).  This branch allows us to do full AND incremental
    backups without having to define a [backup] part twice.  This branch gives
    us the new "fullbackup" script for that purpose.

    We store all of the backups to a shared NFS mount
    and compartmentalized by server name.

    -- location = /mnt/zeobackups/${environ:servername}/backups

*Cron jobs*
    We use the "z3c.recipe.usercrontab" recipe to set up the cron jobs for our
    backups.  One does a full backup weekly, the other a daily incremental
    backup.  Finally we cron the packing job as well.

*Repo Variable*
    In the runfiles, we have a [repos] part that defines the "repo" variable
    which makes it handy to switch branches in packages.cfg. If you look in
    there, you'll see an example of how to use it in the [sources] part.

    -- SAMPLE.DEVELOPMENT.PACKAGE svn ${repos:repo}/SAMPLE.DEVELOPMENT.PACKAGE

*Reusable Parts*
    By leveraging the "<=" operator we're able to instantiate mutliple instances
    using only one part, then overriding the values that are specific to that
    part.  We also make heavy use of "${:_buildout_section_name_}" which is
    super useful.
    
* Buildout init script
	This buildout will install one supervisor instance for each runfile, which
	may seem weird but if you think about it for a minute makes sense.  They
	run on separate ports, by default, determined by the port-suffix.
	This init script will start all instances.  Just update the "instances"
	variable if you add a new one.


Usage
=====

In order to use this for your own builds, all you need to do is make your own
runfiles.  In the runfile, you'll notice how the parts directive includes all
of the options available to the entire build.  You simply comment out what you
don't need for a particular build.  You should never need to edit the
underlying configs, just the runfiles.

If you need more instances, you can use the "<=" operator to add as many as
you'd like.  See "run-example.cfg" for an example.

There are a few useful variables set up for you to use if you wish. By no
means are you requried to use them, they just help me not miss filling out
a value somewhere.  They are:

port-suffix
    A number to append to the end of the Zope/ZEO ports. So, if you set the
    value to "3", Zope will listen on 8083, ZEO on 8103.  Obviously this value
    doesn't make sense to use if you're building multiple instances since they
    need to listen on unqiue ports.

server-prefix
    All of our servers live at the same domain, so I added this to simplify
    setting the name.  Server prefix is added to a few custom values in the
    [environ] part.

Notes
=====

I hope you find this useful. It took me a long time to figure out exactly how
to make everything this explicitly independent and re-usable.  I welcome
feedback and comments.

~Spanky


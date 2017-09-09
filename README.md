<a href="https://www.patreon.com/sanityinc"><img alt="Support me" src="https://img.shields.io/badge/Support%20Me-%F0%9F%92%97-ff69b4.svg"></a>

Runit services for heavy-duty Rails hosting
===========================================

Here's a bunch of scripts for deploying a full production Rails app
that runs under runit.

This scheme is currently used to run several large production Rails
apps, including [looktothesters.org](https://www.looktothestars.org/),
and it has worked very nicely for a couple of years now.  More
reliable than system-wide init.d scripts, and fewer permission
headaches.


How to use it
=============

Create an unprivileged user for your Rails app, then set up a
system-wide Runit service to manage user-specific services for that
user -- see [this page for
details](http://www.sanityinc.com/articles/init-scripts-considered-harmful).

All further steps assume that you are working as the unprivileged user.

Now, clone this project into the user's home directory:

```
cd
git clone git://github.com/purcell/rails-runit.git
```

Symlink your rails app's root directory to this directory with the
name 'app':

```
ln -s ~appuser/releases/current ~/rails-runit/app
```

Now you can create some services:

```
cd ~/rails-runit
./add-mongrel 4001
./add-mongrel 4002
./add-mongrel 4003
./add-haproxy 4000
./add-nginx 4020 4000
```

This creates (potential) runit services under ~/rails-runit/service.
Symlink them to ~/service, and runit will start them for you.

Once started, you can set your system-wide web server to proxy requests
for your Rails app to port 4000 (haproxy => mongrel * 3), or to port 4020
(nginx => haproxy => mongrel * 3).

Browse to port 3998 (= haproxy port - 2) to see the haproxy status
page.

You can then easily restart your app (e.g. as part of a capistrano
deployment) like this:

```
sv restart ~/service/mongrel-*
```

Further "add-*" and "*-run" scripts are provided for varnish and
ferret_server.


Status
======

The haproxy service assumes that the mongrels will be found on
the 10 ports higher than itself.

The use of the 'production' RAILS_ENV is hard-coded, so look out
for that if you try it on your own machine.

The entire scheme is subject to change -- see below.


Future plans
============

I'd like to rewrite all this in ruby so we can have templates for the
config files in ERB, and to make the port mapping a bit more flexible.
The 'run' scripts would then look up their config templates under
RAILS_ROOT/config first, and fall back to ~/rails-runit/ if necessary.

It'd be nice to automatically generate monitrc fragments that could
be included by a system-wide monit instance for the purposes of
monitoring the app.

# Rambling about a similar idea but no manager

In this project I'm looking at solutions for having state (a database file,
object storage, etc.) combined with some sort of auto setup.

I've already looked at various options. For example Fedora CoreOS which configures
itself on first boot. It does support persistent volumes on `/etc` and `/var`.

Every application must run in a container either run by Docker of Podman. The
main issue I have with this is that you need to [reboot the system for every
config change](https://github.com/coreos/fedora-coreos-tracker/issues/470).
Want to deploy a new version, reboot. New application on the server which
already runs some application, whole server needs to reboot. It just isn't a
good fit for applications with embedded databases. Which have difficulty
scaling horizontally but are easy to maintain. Something I prefer right now.

My goal is to build an application that manages applications on one server. No
horizontal scaling or anything like that. The application just goes to an HTTP
endpoint specific to that server which contains the binary files it should run.
Or maybe a tar file it should open. Some config for `/etc/rc.d` to run the
applications. `rcctl` is idempotent which is nice for this kind of use case.

It should get the binary if the changed timestamp is larger than the current
files it is managing.

## What about Ansible and Puppet?

I'm just getting bored with Ansible. Puppet also doesn't really tickle my
interests. Ansible also requires Python. Python is really nice, but I don't
want it on my servers just for the sake of running Ansible. I want to do my
own thing, that is what it project is really about.

## Design

I was thinking about a simple HTTP server which support uploads of new
binaries and configurations per server (just a directory per server).

But maybe a link to Git repo is an idea? But then it is more of a CI/CD
application. It would solve the issue of cross-compiling from Linux to OpenBSD.
But let's keep it simple for now. Just an application manager pulling binaries
and configs from an HTTP server. Or possibly something like an object store?
Don't know yet.

### State

We could get all the state from file statistics. Is changed timestamp smaller
than time t. Update, otherwise do nothing. So the HTTP server is the source
of truth? Does a directory no longer exist then the application should be
shutdown and removed from the server.

Something simple like a httpd index would be good enough.

Here is an example of a httpd index of PXE and OpenBSD autoinstall config files:

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>Index of /</title>
<style type="text/css"><!--
body { background-color: white; color: black; font-family: sans-serif; }
hr { border: 0; border-bottom: 1px dashed; }
@media (prefers-color-scheme: dark) {
body { background-color: #1E1F21; color: #EEEFF1; }
a { color: #BAD7FF; }
}
--></style>
</head>
<body>
<h1>Index of /</h1>
<hr>
<pre>
<a href="../">../</a>                                                23-Jan-2022 16:56                   -
<a href="beef-disklabel">beef-disklabel</a>                                     22-Feb-2022 16:07                 126
<a href="db1.dest.lan-install.conf">db1.dest.lan-install.conf</a>                          22-Feb-2022 16:04                1390
<a href="db2.dest.lan-install.conf">db2.dest.lan-install.conf</a>                          22-Feb-2022 16:04                1390
<a href="db3.dest.lan-install.conf">db3.dest.lan-install.conf</a>                          22-Feb-2022 16:04                1390
<a href="disklabel">disklabel</a>                                          25-Jan-2022 19:04                 175
<a href="install.conf">install.conf</a>                                       22-Feb-2022 11:56                1385
<a href="pub/">pub/</a>                                               23-Jan-2022 16:58                   -
</pre>
<hr>
</body>
</html>
```

Directories don't have file size in bytes and contain a `/` postfix, so they
are easily recognisable. Files have changed at date with a file size in bytes.
And it is certainly parsable by the `time.Parse()` function. So looks good.

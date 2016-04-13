pts-letsencrypt: Convenient and secure wrapper for letsencrypt

pts-letsencrypt is a convenient and secure wrapper for the letsencrypt
command-line tool. It calls letsencrypt with the right arguments to create,
renew and revoke domain SSL certificates. It runs in batch mode (i.e. no
interactive output from the user), and it can be called from cron jobs.

Which letsencrypt.org client to use in production
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
* The reference client (the letsencrypt command-line tool). Probably not,
  because it runs as root and has too many library dependencies, and this
  doesn't sound secure.

* pts-letsencrypt (https://github.com/pts/pts-letsencrypt). See the section
  below why.

* letsencrypt-nosude (https://github.com/diafygi/letsencrypt-nosudo). It can
  easily be run on a different (non-production) server.

* lego (written in Go, https://github.com/xenolf/lego). Versatile, and can
  be compiled to a single, statically linked binary.

* Probably there are other good ones, see them on
  https://news.ycombinator.com/item?id=11480708  

Why use pts-letsencrypt over the original letsencrypt command-line tool
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
* pts-letsencrypt should be run as non-root by design.

* pts-letsencrypt is batch-only (non-interactive): it never asks any questions,
  it just does what's mentioned in the command-line.

* pts-letsencrypt is easier to install from binary on Linux i386 and amd64
  (see in the section below).
  letsencrypt has lots of Python and C library dependencies.

* pts-letsencrypt doesn't try to auto-update itself when run.
  (letsencrypt-auto does, as root, potentially triggering lots of library
  installation, gcc invocation etc.)

* pts-letsencrypt doesn't modify system directories (e.g. /etc/letsencrypt)
  by default.

* pts-letsencrypt doesn't install code to hidden directories.
  (letsencrypt-auto installs Python libraries to
  /root/.local/share/letsencrypt .)

* Both the code and the config directories of pts-letsencrypt are very easy
  to copy around, even to another server. It's obvious where files are, and
  what needs to be copied.

* pts-letsencrypt doesn't try to be smart to edit webserver (e.g. Apache)
  configs or automatically restart webservers.

* pts-letsencrypt needs only few command-line flags to be specified.

* pts-letsencrypt restricts the permissions of the files and directories it
  creates, so other (non-root) users on the same machine won't be able to
  make a copy of the private key (to be used e.g. for decrypting captured
  SSL traffic). It also hides the private keys (privkey*.pem files) and
  everything except the webroot dir from all other users (except for root).

A limitation of pts-letsencrypt: currently it supports only 1 domain name per
certificate. (This limitation can be lifted in the future.)

Installation instructions
~~~~~~~~~~~~~~~~~~~~~~~~~
letsencrypt needs to be installed on the server running the webserver, because
it needs the be run again each 3 months to renew the certificate.

If you are running Linux on i386 or x86_64 architectures, there is a binary
release. Here is how to download and use it (as non-root, don't type the
leading $s):

  $ rm -f pts-letsencrypt-0.5.0.3.sfx.7z
  $ wget https://github.com/pts/pts-letsencrypt/releases/download/v0.5.0.3/pts-letsencrypt-0.5.0.3.sfx.7z
  $ chmod +x pts-letsencrypt-0.5.0.3.sfx.7z
  $ ./pts-letsencrypt-0.5.0.3.sfx.7z
  $ ls -l pts-letsencrypt-0.5.0.3/bin/pts-letsencrypt
  $ pts-letsencrypt-0.5.0.3/bin/pts-letsencrypt --help
  $ pts-letsencrypt-0.5.0.3/bin/pts-letsencrypt --help all

The download size is about 11 MB, and it will use about 55 MB of space when
extracted. It works on any Linux (i386 or x86_64) system, it's
self-contained, it doesn't use any system libraries (e.g. libc) neither for
installation nor for running.

You can move the installation directory (pts-letsencrypt-0.5.0.3) around, and
you can create symlinks to the pts-letsencrypt executable. Users running
pts-letsencrypt need only read-only access to the directory.

The first 3 items in the version number (e.g. 0.5.0) refer to the letsencrypt
version used, as released on PyPI.

---

If you are running any other Unix system, you need to install letsencrypt >=
0.5.0 first in a way that it is available as a python import, so this must
work in the end:

  $ python -c 'import letsencrypt.cli'

The installation process to get there may get complicated because
letsencrypt has lots of Python and C library dependencies. So you have to
install Python 2.7, pip, some C libraries (such as OpenSSL). gcc (for
compiling Python extensions). For example, on Debian and Ubuntu Linux systems
(as root, don't type the leading #):

  # apt-get install python2.7 python2.7-dev gcc dialog libssl-dev libffi-dev ca-certificates

On other systems you may get inspiration on how to install libraries from the
https://github.com/letsencrypt/letsencrypt/blob/master/letsencrypt-auto
script.

Then you can execute (as root):

  # pip install letsencrypt

Please note that the command above would probably fail for the first time,
because some development packages are missing.

To install a specific version:

  # pip install -vI letsencrypt==0.5.0

Once letsencrypt is installed, pts-letsencrypt is easy to install:

  $ rm -f pts-letsencrypt
  $ wget https://raw.githubusercontent.com/pts/pts-letsencrypt/master/pts-letsencrypt
  $ chmod +x pts-letsencrypt
  $ ./pts-letsencrypt --help
  $ ./pts-letsencrypt --help all

You can move the pts-letsencrypt executable file around, and you can create
symlinks to it. Users running it need only read-only access.

Creating your first SSL certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Decide which hostname (can be non-top-level domain) you want an SSL
certificate for. From now on we call it HOSTNAME.

Decide about an administrative e-mail address letsencrypt.org will use to
contact you. From now on we call it EMAIL.

Have a DNS record (A or CNAME) pointing to your server running a webserver
for that hostname.

Start the webserver, and configure it so that it can serve static files on
http://HOSTNAME/ . Verify it from your web browser.

Install pts-letsencrypt onto the server (see in the section above).

Select (or create) a non-root user you'll run pts-letsencrypt as.

As that non-root user, create a directory which can hold a couple of
megabytes of data for each SSL certificate. We'll refer to that directory as
CONFIGDIR from now on, and its absolute path as /ABSCONFIGDIR. You don't
have to create CONFIGDIR, pts-letsencrypt will create it for you.

Make sure you are able to run `pts-letsencrypt --help all' as that non-root
user.

Run (as the non-root user):

  $ pts-letsencrypt --config_dir=CONFIGDIR --domains=HOSTNAME --email=EMAIL

This will fail quickly, showing an error message like this:

  error: Got from webserver for http://HOSTNAME/.well-known/acme-challenge/plemark.txt: (404, 'Not Found', '<html>\r\n<head><title>404 Not Found</title></head>\r\n<body bgcolor="white">\r\n<center><h1>404 Not Found</h1></center>\r\n<hr><center>')
         Expected: (200, 'OK', '570d60faf745e4574fd0a35ddb339d91a5c8bad3b758a1892ea5804c16a893721c53\n')
  fatal: Please set up your webserver on localhost so that http://HOSTNAME/.well-known/acme-challenge/ serves /ABSCONFIGDIR/webroot/.well-known/acme-challenge/ , and rerun this command.

Do as it asks you. For example, if you use ngninx as the webserver, you
need to add a `location' tag, like this:

  http {
    ...
    server {
      listen ...:80;
      server_name HOSTNAME;
      root ...;
      location /.well-known/acme-challenge/ {
        alias /ABSCONFIGDIR/webroot/.well-known/acme-challenge/;
      }
    }
  }

Give enough permission for the Unix user the webserver is using (e.g.
www-data) to read (chmod a+wx) directories in the path to
/ABSCONFIGDIR/webroot/.well-known/acme-challenge . pts-letsencrypt will
verify this and issue a warning (displaying the chmod command you have to
run) if permissions aren't broad enough.

Make your webserver reload its config. For example, if you use nginx (as root):

  # nginx -t
  # /etc/init.d/nginx reload

Rerun (as the non-root user):

  $ pts-letsencrypt --config_dir=CONFIGDIR --domains=HOSTNAME --email=EMAIL

If everything was done properly, this succeeds in about 10 seconds, and
creates the certificate as the files CONFIGDIR/live/HOSTNAME/*.pem . If you
get an error instead, you need to fix it, and retry.

Edit the configuration of your webserver to make it start listening on the
HTTPS port (443) with the certificate. For example, if you use nginx, the
config snippet looks like this:

  http {
    ...
    # Don't modify this one (port 80), keep it is it was.
    server {
      listen ...:80;
      server_name HOSTNAME;
      root ...;
      location /.well-known/acme-challenge/ {
        alias /ABSCONFIGDIR/webroot/.well-known/acme-challenge/;
      }
    }
    # Add a new block for port 443.
    server {
      # ... must be the IP address corresponding to HOSTNAME.
      listen ...:443;
      server_name HOSTNAME;
      ssl on;
      ssl_certificate     /ABSCONFIGDIR/live/HOSTNAME/fullchain.pem;
      ssl_certificate_key /ABSCONFIGDIR/live/HOSTNAME/privkey.pem;
      root ...;
      # No need for the acme-challenge block.
    }
  }

Make your webserver reload its config. For example, if you use nginx (as root):

  # nginx -t
  # /etc/init.d/nginx reload

Check from Google Chrome or Mozilla Firefox that https://HOSTNAME/ works now,
and the browser doesn't complain about the certificate.

Please note that pts-letsencrypt (just like letsencrypt) supports the
--staging flag, which instructs the letsencrypt.org server to sign the
certificates with the (invalid) staging, test keys. The advantage is that it
succeeds more than a couple of times a week, so it's more suitable for
software tool development.

Don't forget to renew the certificate every 3 months! (See the details in
the next section.)

Auto-renewing SSL certificates every 3 months
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
TODO(pts): Write this.

Set up cron (or some other tool which retries on failure) to run these commands
every day:

  # Run this as the non-root user to renew the certificate.
  pts-letsencrypt --config_dir=CONFIGDIR

  # As root, make the webserver reload its config, so that it will detect the
  # new certificate.

__END__

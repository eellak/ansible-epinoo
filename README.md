<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Epinoo Installation Manifests](#epinoo-installation-manifests)
  - [Intro](#intro)
  - [Automation Manifests](#automation-manifests)
  - [Assumptions](#assumptions)
  - [Preinstallation](#preinstallation)
  - [Makefile](#makefile)
  - [Makefile Parameters](#makefile-parameters)
  - [Makefile Targets](#makefile-targets)
    - [test](#test)
    - [init](#init)
    - [puppet_modules](#puppet_modules)
    - [apache](#apache)
    - [mysql](#mysql)
    - [ffmpeg](#ffmpeg)
    - [bbb](#bbb)
    - [moodle](#moodle)
    - [wordpress](#wordpress)
    - [ajenti](#ajenti)
    - [all](#all)
  - [Big Blue Button](#big-blue-button)
    - [ffmpeg issues](#ffmpeg-issues)
    - [Further configuration](#further-configuration)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Epinoo Installation Manifests

## Intro

The [Epinoo Project](http://www.epinoo.gr/) is an initiative of the
[National Technical University of Athens](http://www.ntua.gr) and the
[Institute of Communications and Computer Systems](http://www.iccs.gr) in
collaboration with the [Municipality of Athens](https://www.cityofathens.gr/),
aimed at attracting novel ideas and supporting their evolution into commercially
viable products.

As part of its activities, the Epinoo Project has developed a software platform
to facilitate its operations. The Epinoo Platform is based on already existing
Open Source Software, such as Moodle, WordPress and Big Blue Button, integrated
for the purposes of Epinoo, along with specially developed software, such as
plugins and modules, to support its special needs.

## Automation Manifests

To install and setup the abovementioned pieces of software, this repository
contains several systems manifests which can be used to quickly and easily
setup these various parts of the Epinoo Platform.

The installation process is composed of various installation tasks, each one of
which is codified into a target in the included Makefile. Thus the administrator
of a system can invoke various targets of the Makefile to carry out the
installation of an Epinoo Platform.

The modular nature of this setup as opposed to a monolithic installation script,
allows one to easily choose which parts of the installation to carry out.
Hereafter is given an outline of the various targets comprising the Makefile.

## Assumptions

OS distribution: **Ubuntu 14.04**

Big Blue Button must be installed in its own server due to requirements
constraints. Moodle and wordpress can be install in the same server.

It is assumed that the working directory is `/root/epinoo-installation-scripts/`.

## Preinstallation

Prior to applying the puppet modules you have to install `git` and `make`:

```
sudo apt-get install git make
```

Then, on the server where the platforms will be installed, clone this
repository:

```
sudo -i
git clone https://github.com/eellak/epinoo-installation-scripts.git
cd epinoo-installation-scripts
```

Finally, run the Makefile passing the right arguments as listed in the next
section.

## Makefile

All the targets listed bellow can be invoked by simply issuing

```
make [target]
```

in the repo directory, replacing `[target]` with an actual target name.

It is possible to influence the installation by setting several configuration
parameters which have been preset to sane defaults. These defaults can be
overriden either by editing the Makefile directly, or by overriding in the
command line, e.g.

```
make [target] parameter1=value1 parameter2=value2
```

## Makefile Parameters

| Variable | Description |
| -------- | ----------- |
| `FACTER_WORDPRESS` | The web site name (vhost in apache) of the epinoo website, defaults to epinoodev.ellak.gr |
| `FACTER_MOODLE` |  The web site name (vhost in apache) of the moodle website, default to moodledev.ellak.gr |
| `FACTER_MOODLE_DB_PWD` | A password to set for the Moodle MySQL account. User should change this to something other than the default. |
| `FACTER_DEFAULT_IP` | The IP address to which the apache web server listens. |
| `FACTER_WORDPRESS_IP` | Defaults to `FACTER_DEFAULT_IP`, the IP address to which the Epinoo vhost listens. It is possible to have the Epinoo and the Moodle web sites listen to different addresses. |
| `FACTER_MOODLE_IP` | See above, the IP address to which the Moodle vhost listens. Defaults to FACTER_DEFAULT_IP. |
| `FACTER_MOODLE_SSL_DIR` | A directory containing the certificate/key pair of the Moodle web site. Defaults to `/etc/ssl/ellak` |
| `FACTER_MOODLE_SSL_CERT` | Defaults to `FACTER_MOODLE_SSL_DIR/ssl.crt`, the certificate used by the Moodle vhost. |
| `FACTER_MOODLE_SSL_KEY` | Defaults to `FACTER_MOODLE_SSL_DIR/ssl.key`, the private key used by the Moodle vhost. Please take special care not to place this file in a public location. |
| `FACTER_MOODLE_FULLNAME` | A verbose description of the Moodle web site. |
| `FACTER_MOODLE_SHORTNAME` | A short description of the Moodle web site. |
| `FACTER_MOODLE_ADMINPASS` | The administration password of the Moodle website. User should change this to something other than the default. |
| `FACTER_MOODLE_URL` | Defaults to `http://FACTER_MOODLE`, the URL of the Moodle website. It assumes there are existing ssl keys on the server. |
| `FACTER_LISTEN_PORT` | Defaults to 80, the port number to which the web server listens for HTTP connections. |
| `FACTER_LISTEN_SSL_PORT` | Defaults to 443, the port number to which the web server listens for HTTPS connections. |
| `FACTER_WWW_ROOT` | Defaults to `/var/www`, the root directory of the web server. |
| `FACTER_WORDPRESS_ROOT` | Defaults to `FACTER_WWW_ROOT/FACTER_WORDPRESS`, the root directory of the Epinoo vhost. |
| `FACTER_MOODLE_ROOT` | Defaults to `FACTER_WWW_ROOT/FACTER_MOODLE`, the root directory of the Moodle vhost. |
| `FACTER_WWW_USER` | Defaults to `www-data` (default web server user in Ubuntu), the user which has permissions to read and write the certain www directories, see moodle target bellow. |
| `FACTER_WORDPRESS_DB_PWD` | The password for the Epinoo database in the MySQL server. User should change this to something other than the default. |
| `FACTER_WORDPRESS_UNIQUE_PHRASE` | A unique passphrase for the Epinoo website administration. User should change this to something other than the default. |
| `FACTER_FFMPEG_VERSION` | The current supported ffmpeg version by BBB. |

## Makefile Targets

### test
Prints all the current configuration variables values. Does nothing else.

### init
Setup Puppet, install some SSH keys in the host, disable the Puppet agent in
the host. This is a required step before issuing any other Makefile target.

### puppet_modules
Installs the necessary puppet modules to carry out the rest of the targets.
User needn't use this explicitly, as will be invoked as necessary as part of
other targets. Documented here for posterity.

### apache
Installs a standard Apache HTTPD into the system, customizes the installation
accordingly for Moodle and wordpress. Also setup a Shibboleth SP pointing to
`wayf.grnet.gr` as its WAYF. To make this Shibboleth SP functional, its
Metadata should be published in the federation. Furthermore, the IdPs in the
federation should be accordingly setup to release eppn and mail to this SP.
See the file apache.pp for more details.

### mysql
Install an empty MySQL database server. See the file mysql.pp for more detals.

### ffmpeg
Build a Debian package of the FFMPEG software. This will clone a bazaar repo
and carry out the build process. Use this before attempting to install bbb.

An ffmpeg precompiled package is provided in the `src/` directory. To build a
newer version of ffmpeg just change the `FACTER_FFMPEG_VERSION` variable.
By default version 2.3.3 is used which is the one upstream suggests.

Run with:

```
make ffmpeg
```

Check BBB's [official documentation][bbbffmpeg] to see what the current
supported version is.

**NOTE:** The ffmpeg binary is distributed under the [GPLv2 license][ffmpeglic].

### bbb
Install Big Blue Button. This is a rather complex process, mostly because it
involves the creation of an ffmpeg package specialized for this need.

Run with:

```
make bbb
```

Read more below at [Big Blue Button](#big-blue-button).

### moodle
Install a moodle setup. Will download a Moodle software snapshot from the
Moodle repository (`git://git.moodle.org/moodle.git`) and customize accordingly.
The data root of the Moodle installation will belong to `FACTER_WWW_USER`, see
above. The user may have to install via the moodle web interface various other
components.

It is assumed that the default url scheme is under HTTPS. If you want to
change that behaviour you will have to change the `FACTER_MOODLE_URL`
parameter.

**Note:** The parameter `FACTER_WORDPRESS` needs to be set as well since
    the apache manifest depends on it. Will need to be fixed in the future.

Example run:

```
make moodle FACTER_WORDPRESS=wp.example.com FACTER_MOODLE=moodle.example.com FACTER_DEFAULT_IP=1.2.3.4
```

### wordpress
Install an Epinoo WordPress site into the web server. This essentially downloads
a snaphost of the wordpress codebase, installs it in an appropriate vhost and
creates a suitable wp-config.php for the installation. See wordpress.pp for more
details. The user may have to install via the wordpress/epinoo web interface
various other components.

**Note:** The parameter `FACTER_MOODLE` needs to be set as well since
    the apache manifest depends on it. Will need to be fixed in the future.

Example run:

```
make wordpress FACTER_WORDPRESS=wp.example.com FACTER_MOODLE=moodle.example.com FACTER_DEFAULT_IP=1.2.3.4
```

After the initial installation completes, visit your wordpress url to provide
some final details to install.

### ajenti
Install and start the [Ajenti Server Admin Panel](http://ajenti.org).
The Ajenti Server Admin Panel has its own web server listening on port 8000
and operates independently from the rest of the abovementioned software
packages.

Run:

```
make ajenti
```

The user should make subsequent adjustments to the installation by visiting
https://[server-hostname]:8000. See ajent.pp for more details.

Default username is `root` and password is the password of the root system user.

### all
Will apply all the above targets in the correct order. *BEWARE:* This will make
a tremendous amount of changes in the host and is probably not what a cautious
person would use.


## Big Blue Button

### ffmpeg issues

Due to licensing issues with the ffmpeg codebase, it is not allowed to
distribute a binary package of ffmpeg in certain countries. To work around this
issue, a small launchpad repo has been created as part of this project to host
the description necessary to create an ffmpeg .deb package on the fly. Of course,
having created the package once, the procedure need not be repeated again
elsewhere, but the .deb package can be freely copied to whatever host needs it.

To create the package, the script `ffmpeg.sh` is provided as part of the repo.
The user can either run ffmpeg.sh directly, or issue `make ffmpeg`, which will
run the script in exactly the same way. The ffmpeg.sh script will copy the
necessary source from the appropriate launchpad bazaar repo and the build the
debian package. The necessary compile parameters needed by the other parts of
the Big Blue Button have already been set appropriately. Once the ffmpeg.deb
package has been created, the bbb Makefile target can be invoked to install it.

### Further configuration

The installation of BBB is the basic one. For further configuration read the
official documentation from [this][bbb-install] step onwards.

[bbb-install]: http://docs.bigbluebutton.org/install/install.html#6-install-api-demos
[bbbffmpeg]: http://docs.bigbluebutton.org/install/install.html#4-install-ffmpeg
[ffmpeglic]: https://www.ffmpeg.org/legal.html

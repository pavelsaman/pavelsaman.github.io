---
layout: page
title:  "Working With Perl Modules"
date:   2020-04-12 20:20:19 +0200
categories: perl
---

This article is a short summary of some of the most important commands, utilities and ways used for working with Perl modules. As usual, I'll use my Manjaro distribution, therefore some of the examples will circle around pacman/pamac as a package manager.

### System package manager

The first and most obvious way is to use what a system has to offer, that is a package manager already present (in most cases anyway).

With pacman, it can't get much easier than typing:

```
$ pacman -Ss perl-* | head -30
core/pcre 8.44-1 [installed]
    A library that implements Perl 5-style regular expressions
core/pcre2 10.34-3 [installed]
    A library that implements Perl 5-style regular expressions. 2nd version
core/perl 5.30.2-1 [installed]
    A highly capable, feature-rich programming language
extra/cairo-perl 1.107-1
    Perl wrappers for cairo
extra/glib-perl 1.329-2
    Perl wrappers for glib 2.x, including GObject
extra/gtk2-perl 1.24993-1
    Perl bindings for GTK+ 2.x
extra/irssi 1.2.2-2
    Modular text mode IRC client with Perl scripting
extra/latex2html 2019-1
    a convertor written in Perl that converts LaTeX documents to HTML.
extra/pango-perl 1.227-11
    Perl bindings for Pango
extra/perl-alien-build 2.17-1 [installed]
    Build external dependencies for use in CPAN
extra/perl-alien-libxml2 0.15-1 [installed]
    Install the C libxml2 library on your system
extra/perl-alien-sdl 1.446-8
    Building, finding and using SDL binaries
extra/perl-anyevent 4:7.17-1
    The DBI of event loop programming
extra/perl-appconfig 1.71-5
    Perl/CPAN AppConfig module - Read configuration files and parse command line arguments
extra/perl-archive-zip 1.68-1
    Provide a perl interface to ZIP archive files
```

That's how many Perl modules could be installed on a system in the first place.

If someone uses pamac, it could be done in a similar way via a GUI:

![image](/images/pamac.png)

One drawback, however, is that not all Perl modules are available through a package manager. Sooner or later, you'd most likely need more tools to download, install Perl modules.

### CPAN

What you can do in situations when a system package manager can't find a module you want to install is to resort to CPAN utility.

What cpan can do for you is to simplify the download and install process of Perl modules. It has its own shell:

```
$ cpan

cpan shell -- CPAN exploration and modules installation (v2.27)
Enter 'h' for help.

cpan[1]> h

Display Information                                                  (ver 2.27)
 command  argument          description
 a,b,d,m  WORD or /REGEXP/  about authors, bundles, distributions, modules
 i        WORD or /REGEXP/  about any of the above
 ls       AUTHOR or GLOB    about files in the author's directory
    (with WORD being a module, bundle or author name or a distribution
    name of the form AUTHOR/DISTRIBUTION)

Download, Test, Make, Install...
 get      download                     clean    make clean
 make     make (implies get)           look     open subshell in dist directory
 test     make test (implies make)     readme   display these README files
 install  make install (implies test)  perldoc  display POD documentation

Upgrade installed modules
 r        WORDs or /REGEXP/ or NONE    report updates for some/matching/all
 upgrade  WORDs or /REGEXP/ or NONE    upgrade some/matching/all modules

Pragmas
 force  CMD    try hard to do command  fforce CMD    try harder
 notest CMD    skip testing

Other
 h,?           display this menu       ! perl-code   eval a perl command
 o conf [opt]  set and query options   q             quit the cpan shell
 reload cpan   load CPAN.pm again      reload index  load newer indices
 autobundle    Snapshot                recent        latest CPAN uploads
cpan[2]> 
```

Or it can be use as a command line tool. Let's see some examples.

To see what Perl modules are currently installed on my system, I'd use cpan like so (shortened for this site):

```
$ cpan -l | head -20
lib::Config::LastError	undef
lib::Config::Reader	0.006
Files::File	undef
ConfigReader::Simpler	undef
Corona::Statistics	1.03
lib::File::Stderr	0.008
lib::XML::Post	0.001
lib::XML::Post::OpeningHour	0.002
lib::XML::Post::OpeningHourOld	0.001
lib::XML::Post::Day	0.001
lib::XML::Post::Branch	0.003
lib::XML::Post::BranchOld	0.001
lib::Zip::Czech	0.004
Socket	2.029
DB_File	1.853
encoding	2.22
Moose	2.2012
Encode	3.05
Want	0.29
oose	2.2012
```

If I wanted to install a module that's not yet installed on my system, I can just type:

```
$ cpan New::Module::To::Install
```

To see what modules could be updated, I type:

```
$ cpan -O
```

And to update them all:

```
$ cpan -u
```

Of course, there's more to it, just have a look at the man page.

However, one disadvantage of cpan is, it can't get rid of already installed modules, there's no option for "uninstalling" modules.

### cpanm

Uninstalling Perl modules has to happen via some other tool. I don't doubt there're more such tools, but I use `cpanm`, or `cpanminus` as it's also called:

```
$ pacman -Qs cpanm
local/cpanminus 1.7044-3
    get, unpack, build and install modules from CPAN
```

And with this tool, there comes a `-U/--uninstall` option:

```
$ cpanm --uninstall Module::To::Uninstall
```

Cpanm can also be used for installing modules and many other things that I don't use the tool for.

### Manual installation

If you don't want to use any utilities that streamline the process, it's still possible to download a desired tarball, unpack it, and follow instructions in `INSTALL` file.

It will most likely come down to the following commands that have to be executed in this order (just like when installing other applications on Linux systems).

```
$ perl Makefile.PL
```

This command will create a `Makefile` file with further instructions on how the module should be installed.

```
$ make
```

Then run tests:

```
$ make tests
```

And finally install the module:

```
$ make install
```

The above-mentioned ways and tool should be enough to get started with Perl modules. I suppose that most of what you'll even need could be done with only these tools.

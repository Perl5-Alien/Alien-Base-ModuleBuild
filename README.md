# Alien::Base::ModuleBuild ![linux](https://github.com/PerlAlien/Alien-Base-ModuleBuild/workflows/linux/badge.svg) ![windows](https://github.com/PerlAlien/Alien-Base-ModuleBuild/workflows/windows/badge.svg) ![macos](https://github.com/PerlAlien/Alien-Base-ModuleBuild/workflows/macos/badge.svg)

A Module::Build subclass for building Alien:: modules and their libraries

# SYNOPSIS

In your Build.PL:

```perl
use Alien::Base::ModuleBuild;

my $builder = Alien::Base::ModuleBuild->new(
  module_name => 'Alien::MyLibrary',

  configure_requires => {
    'Alien::Base::ModuleBuild' => '0.005',
    'Module::Build' => '0.28'
  },
  requires => {
    'Alien::Base' => '0.005',
  },

  alien_name => 'mylibrary', # the pkg-config name if you want
                             # to use pkg-config to discover
                             # system version of the mylibrary

  alien_repository => {
    protocol => 'http',
    host     => 'myhost.org',
    location => '/path/to/tarballs',
    pattern  => qr{^mylibrary-([0-9\.]+)\.tar\.gz$},
  },

  # this is the default:
  alien_build_commands => [
    "%c --prefix=%s", # %c is a platform independent version of ./configure
    "make",
  ],

  # this is the default for install:
  alien_install_commands => [
    "make install",
  ],

  alien_isolate_dynamic => 1,
);
```

# DESCRIPTION

**NOTE**: Please consider for new development of [Alien](https://metacpan.org/pod/Alien)s that you use
[Alien::Build](https://metacpan.org/pod/Alien::Build) and [alienfile](https://metacpan.org/pod/alienfile) instead.  Like this module they work
with [Alien::Base](https://metacpan.org/pod/Alien::Base).  Unlike this module they are more easily customized
and handle a number of corner cases better.  For a good place to start,
please see [Alien::Build::Manual::AlienAuthor](https://metacpan.org/pod/Alien::Build::Manual::AlienAuthor).  Although the
Alien-Base / Alien-Build team will continue to maintain this module,
(we will continue to fix bugs where appropriate), we aren't adding any
new features to this module.

This is a subclass of [Module::Build](https://metacpan.org/pod/Module::Build), that with [Alien::Base](https://metacpan.org/pod/Alien::Base) allows
for easy creation of Alien distributions.  This module is used during the
build step of your distribution.  When properly configured it will

- use pkg-config to find and use the system version of the library
- download, build and install the library if the system does not provide it

# METHODS

## alien\_check\_installed\_version

\[version 0.001\]

```perl
my $version = $abmb->alien_check_installed_version;
```

This function determines if the library is already installed as part of
the operating system, and returns the version as a string.  If it can't
be detected then it should return empty list.

The default implementation relies on `pkg-config`, but you will probably
want to override this with your own implementation if the package you are
building does not use `pkg-config`.

## alien\_check\_built\_version

\[version 0.006\]

```perl
my $version = $amb->alien_check_built_version;
```

This function determines the version of the library after it has been
built from source.  This function only gets called if the operating
system version can not be found and the package is successfully built.
The version is returned on success.  If the version can't be detected
then it should return empty list.  Note that failing to detect a version
is considered a failure and the corresponding `./Build` action will
fail!

Any string is valid as a version as far as [Alien::Base](https://metacpan.org/pod/Alien::Base) is concerned.
The most useful value would be a number or dotted decimal that most
software developers recognize and that software tools can differentiate.
In some cases packages will not have a clear version number, in which
case the string `unknown` would be a reasonable choice.

The default implementation relies on `pkg-config`, and other heuristics,
but you will probably want to override this with your own implementation
if the package you are building does not use `pkg-config`.

When this method is called, the current working directory will be the
build root.

If you see an error message like this:

```
Library looks like it installed, but no version was determined
```

After the package is built from source code then you probably need to
provide an implementation for this method.

## alien\_extract\_archive

\[version 0.024\]

```perl
my $dir = $amb->alien_extract_archive($filename);
```

This function unpacks the given archive and returns the directory
containing the unpacked files.

The default implementation relies on [Archive::Extract](https://metacpan.org/pod/Archive::Extract) that is able
to handle most common formats. In order to handle other formats or
archives requiring some special treatment you may want to override
this method.

## alien\_do\_system

\[version 0.024\]

```perl
my %result = $amb->alien_do_system($cmd)
```

Similar to [Module::Build::do\_system](https://metacpan.org/pod/Module::Build::do_system), also sets the path and several
environment variables in accordance to the object configuration
(i.e. `alien_bin_requires`) and performs the interpolation of the
patterns described in ["COMMAND
INTERPOLATION" in Alien::Base::ModuleBuild::API](https://metacpan.org/pod/Alien::Base::ModuleBuild::API#COMMAND-INTERPOLATION).

Returns a set of key value pairs including `stdout`, `stderr`,
`success` and `command`.

## alien\_do\_commands

```
$amb->alien_do_commands($phase);
```

Executes the commands for the given phase.

## alien\_interpolate

```perl
my $string = $amb->alien_interpolate($string);
```

Takes the input string and interpolates the results.

# GUIDE TO DOCUMENTATION

The documentation for `Module::Build` is broken up into sections:

- General Usage ([Module::Build](https://metacpan.org/pod/Module::Build))

    This is the landing document for [Alien::Base::ModuleBuild](https://metacpan.org/pod/Alien::Base::ModuleBuild)'s parent class.
    It describes basic usage and background information.
    Its main purpose is to assist the user who wants to learn how to invoke
    and control `Module::Build` scripts at the command line.

    It also lists the extra documentation for its use. Users and authors of Alien::
    modules should familiarize themselves with these documents. [Module::Build::API](https://metacpan.org/pod/Module::Build::API)
    is of particular importance to authors.

- Alien-Specific Usage ([Alien::Base::ModuleBuild](https://metacpan.org/pod/Alien::Base::ModuleBuild))

    This is the document you are currently reading.

- Authoring Reference ([Alien::Base::Authoring](https://metacpan.org/pod/Alien::Base::Authoring))

    This document describes the structure and organization of
    `Alien::Base` based projects, beyond that contained in
    `Module::Build::Authoring`, and the relevant concepts needed by authors who are
    writing `Build.PL` scripts for a distribution or controlling
    `Alien::Base::ModuleBuild` processes programmatically.

    Note that as it contains information both for the build and use phases of
    [Alien::Base](https://metacpan.org/pod/Alien::Base) projects, it is located in the upper namespace.

- API Reference ([Alien::Base::ModuleBuild::API](https://metacpan.org/pod/Alien::Base::ModuleBuild::API))

    This is a reference to the `Alien::Base::ModuleBuild` API beyond that contained
    in `Module::Build::API`.

# ENVIRONMENT

- **ALIEN\_VERBOSE**

    Enables verbose output from [M::B::do\_system](https://metacpan.org/pod/Module::Build#do_system).

- **ALIEN\_FORCE**

    Skips checking for an installed version and forces reinstalling the Alien target.

- **ALIEN\_INSTALL\_TYPE**

    Set to 'share' or 'system' to override the install type.  Set to 'default' or unset
    to restore the default.

- **ALIEN\_ARCH**

    Set to a true value to install to an arch-specific directory.

- **ALIEN\_${MODULENAME}\_REPO\_${PROTOCOL}\_${KEY}**

    Overrides $KEY in the given module's repository configuration matching $PROTOCOL.
    For example, `ALIEN_OPENSSL_REPO_FTP_HOST=ftp.example.com`.

# SEE ALSO

- [Alien::Build](https://metacpan.org/pod/Alien::Build)
- [alienfile](https://metacpan.org/pod/alienfile)
- [Alien::Build::Manual::AlienAuthor](https://metacpan.org/pod/Alien::Build::Manual::AlienAuthor)
- [Alien](https://metacpan.org/pod/Alien)

# THANKS

Thanks also to

- Christian Walde (Mithaldu)

    For productive conversations about component interoperability.

- kmx

    For writing Alien::Tidyp from which I drew many of my initial ideas.

- David Mertens (run4flat)

    For productive conversations about implementation.

- Mark Nunberg (mordy, mnunberg)

    For graciously teaching me about rpath and dynamic loading,

# AUTHOR

Original author: Joel A Berger <joel.a.berger@gmail.com>

Current maintainer: Graham Ollis <plicease@cpan.org>

Contributors:

David Mertens (run4flat)

Mark Nunberg (mordy, mnunberg)

Christian Walde (Mithaldu)

Brian Wightman (MidLifeXis)

Graham Ollis (plicease)

Zaki Mughal (zmughal)

mohawk2

Vikas N Kumar (vikasnkumar)

Flavio Poletti (polettix)

Salvador Fandiño (salva)

Gianni Ceccarelli (dakkar)

Pavel Shaydo (zwon, trinitum)

Kang-min Liu (劉康民, gugod)

Nicholas Shipp (nshp)

Petr Písař (ppisar)

Alberto Simões (ambs)

# COPYRIGHT AND LICENSE

This software is copyright (c) 2012-2020 by Joel A Berger.

This is free software; you can redistribute it and/or modify it under
the same terms as the Perl 5 programming language system itself.

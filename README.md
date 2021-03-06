Proposed Migration
==================

Proposed migration is the process of moving things from the proposed "xyz-proposed" pocket to the release "xyz" pocket.

Migration is blocked if its uninstallibility count increases as a consequence of the migration. A package that has unmet dependencies is uninstallable, and packages that depend on uninstallable packages are themselves uninstallable.

Migration will also be blocked if a package's autopkgtests fail, which can happen if upgrading a package breaks something in the package or packages that depend on it.

[https://wiki.ubuntu.com/ProposedMigration] explains the process.

Dependencies
------------

There are two kinds of dependencies: source and binary.

Due to how the system architecture, multiple source package version can exist, but only one binary package may exist across all versions. To get around this limitation, we append a version number to shared object name (e.g. `libjpeg3`).

The proposed pocket is a staging area that is overlaid on top of release.


Package Transition Example
--------------------------

### Assume the following fictional packages:

#### Package src:jpegtools 1.0:

 * Produces binary package `libjpeg1 1.0`:
   - contains shared object `/usr/lib/libjpeg.so.1`
 * Produces binary package `libjpeg-dev 1.0`:
   - depends on `libjpeg1`
   - contains header `/usr/include/jpeg.h`
   - contains symlink `/usr/lib/libjpeg.so -> /usr/lib/libjpeg.so.1`

#### Package src:imageviewer 8.4:

 * Compiles with `/usr/include/jpeg.h` (`#include <jpeg.h>`)
 * Links against `/usr/lib/libjpeg.so.1` (`-ljpeg`)
 * Produces binary package `imageviewer 8.4`:
   - contains ELF executable `/usr/local/bin/imageviewer`

When package `src:imageviewer 8.4` is built, packaging tools pick up on this (via `dpkg-shlibdeps`) and create a `Depends: libjpeg1` via `${shlibs:Depends}`.

#### Once everything is built:

The following packages exist:

 * Package `src:jpegtools 1.0`.
 * Package `src:imageviewer 8.4`, which has a build dependency on `libjpeg-dev`.
 * Package `libjpeg1 1.0`.
 * Package `libjpeg-dev 1.0`, which depends on `libjpeg1`.
 * Package `imageviewer 8.4`, which depends on `libjpeg1`.

The system has the following files:

 * `/usr/lib/libjpeg.so.1`
 * `/usr/include/jpeg.h`
 * `/usr/lib/libjpeg.so -> /usr/lib/libjpeg.so.1`
 * `/usr/local/bin/imageviewer`


### Assume the following proposed package:

#### Source package jpegtools 2.0, replaces source package jpegtools 1.0:

 * Produces binary package	`libjpeg2 2.0`:
   - contains shared object `/usr/lib/libjpeg.so.2`
 * Produces binary package `libjpeg-dev 2.0`:
   - depends on `libjpeg2`
   - contains header `/usr/include/jpeg.h`
   - contains symlink `/usr/lib/libjpeg.so -> /usr/lib/libjpeg.so.2`

Assume that 2.0 only introduces new API calls. The existing API works as before (there's an ABI break, but no API break).

#### Once the new jpegtools is built:

The following packages exist:

 * Package `src:jpegtools 2.0`.
 * Package `src:imageviewer 8.4`, which has a build dependency on `libjpeg-dev`.
 * Package `libjpeg1 1.0`.
 * Package `libjpeg2 2.0`.
 * Package `libjpeg-dev 2.0`, which depends on `libjpeg2`.
 * Package `imageviewer 8.4`, which depends on `libjpeg1` (because it hasn't been rebuilt).

The system has the following files:

 * `/usr/lib/libjpeg.so.1`
 * `/usr/lib/libjpeg.so.2`
 * `/usr/include/jpeg.h`
 * `/usr/lib/libjpeg.so -> /usr/lib/libjpeg.so.2`
 * `/usr/local/bin/imageviewer`


### The Problem

 * We cannot remove binary package `libjpeg1` because `imageviewer 8.4` depends on it and would become uninstallable.
 * `libjpeg1` is now NBS (Not Built from Source) because its source package `src:jpegtools 1.0` is gone. We cannot do an SRU for `src:jpegtools 1.0` anymore (for example, a security update).


### The Solution

 * Do a "no change rebuild" of `src:imageviewer 8.4` so that it depends on `libjpeg2`.
 * Remove binary package `libjpeg1`

Note: You update the version of imageviewer using `dch --rebuild`

Now, the following packages exist:

 * Package `src:jpegtools 2.0`.
 * Package `src:imageviewer 8.4ubuntu1`, which has a build dependency on `libjpeg-dev`.
 * Package `libjpeg2 2.0`.
 * Package `libjpeg-dev 2.0`, which depends on `libjpeg2`.
 * Package `imageviewer 8.4ubuntu1`, which depends on `libjpeg2`.

The system has the following files:

 * `/usr/lib/libjpeg.so.2`
 * `/usr/include/jpeg.h`
 * `/usr/lib/libjpeg.so -> /usr/lib/libjpeg.so.2`
 * `/usr/local/bin/imageviewer`

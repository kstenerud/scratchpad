Example
-------

### Assume the following fictional packages:

#### Package src:jpegtools 1.0:

 * Produces binary package libjpeg1 1.0:
   - contains shared object `/usr/lib/libjpeg.so.1`
 * Produces binary package libjpeg-dev 1.0:
   - depends on libjpeg1
   - contains header `/usr/include/jpeg.h`
   - contains symlink `/usr/lib/libjpeg.so -> /usr/lib/libjpeg.so.1`

#### Package src:imageviewer 1.0:

 * Compiles with /usr/include/jpeg.h (`#include <jpeg.h>`)
 * Links against /usr/lib/libjpeg.so.1 (`-ljpeg`)
 * Produces binary package imageviewer:
   - contains ELF executable `/usr/local/bin/imageviewer`

When package `src:imageviewer` is built, packaging tools pick up on this (via `dpkg-shlibdeps`) and create a `Depends: libjpeg1` via `${shlibs:Depends}`.

#### Once everything is built:

The following packages exist:

 * Package `src:jpegtools 1.0`.
 * Package `src:imageviewer 1.0`, which depends on `libjpeg1` and `libjpeg-dev`.
 * Package `jpegtools 1.0`.
 * Package `libjpeg-dev 1.0`, which depends on `libjpeg1`.
 * Package `imageviewer 1.0`, which depends on `libjpeg1`.

The system has the following files:

 * `/usr/lib/libjpeg.so.1`
 * `/usr/include/jpeg.h`
 * `/usr/lib/libjpeg.so -> /usr/lib/libjpeg.so.1`
 * `/usr/local/bin/imageviewer`


### Assume the following proposed package:

#### Source package jpegtools 2.0, replaces source package jpegtools 1.0:

 * Produces binary package libjpeg2 2.0, containing:
   - shared object `/usr/lib/libjpeg.so.2`
 * Produces binary package libjpeg-dev 2.0, containing:
   - header `/usr/include/jpeg.h`
   - symlink `/usr/lib/libjpeg.so -> /usr/lib/libjpeg.so.2`

Assume that 2.0 only introduces new API calls. The existing API works as before (there's an ABI break, but no API break).

#### Once the new jpegtools is built:

The following packages exist:

 * Package `src:jpegtools 2.0`.
 * Package `src:imageviewer 1.0`, which depends on `libjpeg2` and `libjpeg-dev`.
 * Package `jpegtools 2.0`.
 * Package `libjpeg-dev 2.0`, which depends on `libjpeg2`.
 * Package `imageviewer 1.0`, which depends on `libjpeg1` (because it hasn't been rebuilt).

The system has the following files:

 * `/usr/lib/libjpeg.so.1`
 * `/usr/lib/libjpeg.so.2`
 * `/usr/include/jpeg.h`
 * `/usr/lib/libjpeg.so -> /usr/lib/libjpeg.so.2`
 * `/usr/local/bin/imageviewer`

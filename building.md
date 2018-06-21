# Building VisIt from source

This document is designed to serve as a reliable build "recipe" for VisIt, and
will be updated as the process changes and evolves.  Currently this recipe
involves using the SVN repo. However, VisIt is beginning the migration to
GitHub, so expect this to change sometime soon.

There is a `build_visit` script, which ironically is generally used to build
third party dependencies, and not VisIt itself.

## Setup

Create a VisIt directory somewhere, e.g. I usually have `/home/ahota/visit`
Since this is building from trunk, I usually create a `trunk/` directory in there, i.e. `/home/ahota/visit/trunk/`
Within the trunk directory, make `build` and `third_party` directories
So you should have:

```
$ pwd
/home/ahota
$ tree
.
`-- visit
    `-- trunk
        |-- build
        `-- third_party
```

This is the directory structure the rest of the document will assume.

The `third_party` directory will be used to download, build, and install
VisIt’s third party dependencies.  Required dependencies include Qt, VTK,
Python, CMake, etc.  VisIt does allow you to choose system installed versions
of these libraries, but I highly suggest just letting VisIt download and build
them itself.  This is done so that VisIt has full control over the exact
versions and features that are built into its dependencies, some of which may
be missing or incomplete in a system installation.  There are some exceptions
to this that I will highlight as needed.

The `build` directory is where we will be performing an out-of-source build of VisIt.

## Get VisIt source

The VisIt SVN repo is here: http://visit.ilight.com/svn/visit/trunk/src

As stated above, VisIt is beginning the process of migrating to GitHub. This
step will be updated as needed.

Checkout the SVN repository's `src` as a sibling of `build` and `third_party`

```
$ pwd
/home/ahota/visit/trunk/
$ svn co http://visit.ilight.com/svn/visit/trunk/src
```

Note, if you exclude the `src` at the end of the SVN repo URL, you will
download *a lot* of extra files that you don't need. Make sure to checkout only
the source directory.

## Build third-party libraries

Go into the `third_party` directory. We will invoke the `build_visit` script
from here.  In this example, we are requesting OSPRay and Mesa with OpenSWR.

```
$ pwd
/home/ahota/visit/trunk/third_party
$ ../src/svn_bin/build_visit --qt --vtk --ospray --llvm --mesagl --makeflags -j88 --no-visit
```

Change the number of threads in `--makeflags` to the number of cores/threads on
your machine. The `--no-visit` flag tells the build script to not build VisIt.
We will be doing that manually. I usually also add the `--netcdf --parallel`
flags to build support for NetCDF files and for the parallel engine. You can
run `build_visit --help` to see a list of available flags/dependencies to
build.

## CMake config file

Once you run the above command, the script will handle downloading, building,
and installing all the necessary/requested third-party libraries. By default it
will install libraries to a `visit` subdirectory in `third_party`, e.g. in
`/home/ahota/visit/trunk/third_party/visit`.  This is known as your `VISIT_DIR`
to VisIt.  The script will also create a CMake configuration file named
`<hostname>.cmake` in the `third_party` directory.  This CMake file acts as a
shortcut to tell VisIt where the various third party libraries are located, and
which other features are enabled/disabled.

Copy your CMake config file into `src/config-site`:

```
$ pwd
/home/ahota/visit/trunk/third_party
$ cp <hostname>.cmake ../src/config-site
```

## Build VisIt

Go to the `build` directory and run CMake to configure VisIt. Note that we
are specifically using the CMake installation from the `build_visit` script.

```
$ pwd
/home/ahota/visit/trunk/build
$ ../third_party/visit/cmake/<cmake version>/<architecture>/bin/cmake \
  -D VISIT_BUILD_DIAGNOSTICS=OFF \
  ../src
```

The `VISIT_BUILD_DIAGNOSTICS` flag is set to `OFF` to prevent a test program
from being compiled. This test application attempts to use a system OSMesa
installation, which may or may not exist.  If it doesn't exist, you will just
get a compile error and VisIt won't be compiled.  Ultimately we will be using
OSMesa from the Mesa installation in your `VISIT_DIR`, so this test does not
matter.

You should see towards the beginning of CMake's output that your
`<hostname>.cmake` configuration file was included.  You will probably also see
a long stream of warnings from CMake that the system libGL.so is being masked
by Mesa's libGL.so.  This is fine; we want our Mesa to be used instead of the
system.

Once CMake completes, simply run `make`:

```
$ pwd
/home/ahota/visit/trunk/build
$ make -j 88
```

Change the number of threads to the number of cores/threads on your machine.

## Basic run

Run visit in the `build/bin` directory. I usually pass a dataset to open on the command line to save a few clicks.

```
$ pwd
/home/ahota/visit/trunk/build/bin
$ ./visit -o /path/to/dataset
```

## Troubleshooting

### The GUI doesn't have any text in it

You probably also have errors in the terminal about missing fonts (e.g. "Note that Qt no longer ships fonts")
and/or about negative point sizes (e.g. "QFont::setPointSize: Point size <= 0 (-0.75000), must be greater than 0").

This is a problem with Qt. I've found this problem on Ubuntu 16.04 and 18.04, but it's not clear why it happens.
There are two steps you can try taking.

If you only see the missing fonts directory error, try creating a link to your system fonts directory in the Qt installation.
For example:

```
$ pwd
/home/ahota/visit/trunk/third_party/qt/<qt version>/<architecture>/lib
$ ln -s /usr/share/fonts fonts
```

If you also see the point size error, try building Qt externally. I found
success by copying the Qt tarball that VisIt downloads to a separate directory,
untarring it, and building it with a couple changed flags. The `configure`
command is based on the command found in `src/svn_bin/bv_support/bv_qt.sh`, but
flips `--nomake examples` to `--make examples` and `--nomake tests` to `--make
tests` (last line of the `configure` command shown below)

```
$ pwd
/home/ahota/packages/qt
$ mkdir install
$ tar xf <qt tarball>
$ cd <extracted directory>
$ export CFLAGS="-m64 -fPIC  -O2"
$ export CXXFLAGS="-m64 -fPIC  -O2"
$ ../configure -prefix /home/ahota/packages/qt/install \
  -platform linux-g++-64 -make libs \
  -make tools -no-separate-debug-info  \
  -no-dbus -no-sql-db2 \
  -no-sql-ibase -no-sql-mysql \
  -no-sql-oci -no-sql-odbc \
  -no-sql-psql -no-sql-sqlite \
  -no-sql-sqlite2 -no-sql-tds \
  -no-libjpeg -opensource \
  -confirm-license -skip 3d \
  -skip canvas3d -skip charts \
  -skip connectivity -skip datavis3d \
  -skip doc -skip gamepad \
  -skip graphicaleffects -skip location \
  -skip multimedia -skip networkauth \
  -skip purchasing -skip quickcontrols \
  -skip quickcontrols2 -skip remoteobjects \
  -skip scxml -skip sensors \
  -skip serialport -skip speech \
  -skip wayland -no-qml-debug \
  -qt-xcb -qt-xkbcommon \
  -make examples -make tests
$ make -j 88
$ make install
```

This will unfortunately make your life a little more difficult since we have to
tell VisIt to use an external Qt installation and rebuild. You may be able to
just delete the VisIt-installed Qt and QWT installations, but I would suggest
rebuilding entirely. Note the `--alt-qt-dir` flag.

```
$ pwd
/home/ahota/visit/third_party
$ rm -r visit
$ ../src/svn_bin/build_visit --alt-qt-dir /home/ahota/packages/qt/install --vtk --ospray \
  --llvm --mesagl --makeflags -j88 --no-visit
```

Once this is done, you will need to edit the resulting CMake config file.
In the config file, add the following line after `SETUP_APP_VERSION(QT 5.10.1)`:

```
VISIT_OPTION_DEFAULT(VISIT_QT_DIR /home/ahota/packages/qt/install/)
```

Note that anytime you re-invoke `build_visit`, you will have to make the above
edit.  Now copy this file to `src/config-site` again, and rebuild VisIt from
scratch.

```
$ pwd
/home/ahota/visit/trunk/build
$ rm -r *
$ ../third_party/visit/cmake/<cmake version>/<architecture>/bin/cmake \
  -D VISIT_BUILD_DIAGNOSTICS=OFF \
  ../src
$ make -j 88
```

## Making a binary package

The following command will create a distributable tarball for your VisIt build.

```
$ pwd
/home/ahota/visit/trunk/build
$ make package
```

The tarball will be located inside your `build` directory. When extracted, you
can run VisIt from the `visit<version_artchitecture>/bin` directory.

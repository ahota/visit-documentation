# Building VisIt from source

This document is designed to serve as a reliable build "recipe" for VisIt, and
will be updated as the process changes and evolves.  Currently this recipe
involves using the SVN repo. However, VisIt is beginning the migration to
GitHub, so expect this to change sometime soon.

There is a `build_visit` script, which ironically is generally used to build
third party dependencies, and not VisIt itself.

- [Setup](#setup)
- [Get VisIt source](#get-visit-source)
- [Build third-party libraries](#build-third-party-libraries)
- [CMake config file](#cmake-config-file)
- [Build VisIt](#build-visit)
- [Basic run](#basic-run)
- [Making a binary package](#making-a-binary-package)

## Setup

Create a VisIt directory somewhere, e.g. I usually have `/home/ahota/visit`
Since this is building from trunk, I usually create a `trunk/` directory in there, i.e. `/home/ahota/visit/trunk/`
Within the trunk directory, make `build` and `third_party` directories
So you should have:

```bash
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

```bash
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

```bash
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

```bash
$ pwd
/home/ahota/visit/trunk/third_party
$ cp <hostname>.cmake ../src/config-site
```

## Build VisIt

Go to the `build` directory and run CMake to configure VisIt. Note that we
are specifically using the CMake installation from the `build_visit` script.

```bash
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

```bash
$ pwd
/home/ahota/visit/trunk/build
$ make -j 88
```

Change the number of threads to the number of cores/threads on your machine.

## Basic run

Run visit in the `build/bin` directory. I usually pass a dataset to open on the command line to save a few clicks.

```bash
$ pwd
/home/ahota/visit/trunk/build/bin
$ ./visit -o /path/to/dataset
```

## Making a binary package

The following command will create a distributable tarball for your VisIt build.

```bash
$ pwd
/home/ahota/visit/trunk/build
$ make package
```

The tarball will be located inside your `build` directory. When extracted, you
can run VisIt from the `visit<version_artchitecture>/bin` directory.

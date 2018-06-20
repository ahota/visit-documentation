# Building VisIt from source

This document is designed to serve as a reliable build "recipe" for VisIt, and will be updated as the process changes and evolves.
Currently this recipe involves using the SVN repo. However, VisIt is beginning the migration to GitHub, so expect this to change sometime soon.

There is a build_visit script, which ironically is generally used to build third party dependencies, and not VisIt itself.

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

The `third_party` directory will be used to download, build, and install VisIt’s third party dependencies.
Required dependencies include Qt, VTK, Python, CMake, etc.
VisIt does allow you to choose system installed versions of these libraries, but I highly suggest just letting VisIt download and build them itself.
This is done so that VisIt has full control over the exact versions and features that are built into its dependencies, some of which may be missing or incomplete in a system installation.
There are some exceptions to this that I will highlight as needed.

The `build` directory is where we will be performing an out-of-source build of VisIt.

## Get VisIt source

The VisIt SVN repo is here: http://visit.ilight.com/svn/visit/trunk/src
As stated above, VisIt is beginning the process of migrating to GitHub. This step will be updated as needed

Checkout the SVN repository's `src` as a sibling of `build` and `third_party`

```
$ pwd
/home/ahota/visit/trunk/
$ svn co http://visit.ilight.com/svn/visit/trunk/src
```

Note, if you exclude the `src` at the end of the SVN repo URL, you will download *a lot* of extra files that you don't need. Make sure to checkout only the source directory.

## Build third-party libraries

## CMake config file

## Build VisIt

## Making a binary package

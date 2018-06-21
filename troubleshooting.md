# Troubleshooting VisIt

There's a lot of moving pieces. Things can go wrong :)

## The GUI doesn't have any text in it

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
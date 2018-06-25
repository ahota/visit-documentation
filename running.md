# Running VisIt

VisIt can be used in various ways.  The two primary methods of running VisIt
are (1) fully local and (2) with remote computation.  VisIt can also be used
with the full point-and-click GUI experience, with a scripted GUI, or with a
fully scripted CLI.

The VisIt+OSPRay implementation is currently limited to local OSPRay rendering.
A separate effort (that will hopefully be merged in the future) for scalable
server-side OSPRay rendering is being undertaken by Qi Wu at the University of
Utah.

This running guide assumes VisIt was built with OSPRay enabled as described in
the build instructions.

## Basic local run

Launch VisIt from the `build/bin` directory. We will be running VisIt fully
locally in this example.

```bash
$ pwd
/home/ahota/visit/trunk/build/bin
$ ./visit
```

To specifically run with OpenSWR enabled, you have to tell Gallium to use it as the main GL driver.

```bash
$ GALLIUM_DRIVER=swr ./visit
```

With this flag, you should see a message printed to the terminal, e.g. `SWR
detected AVX2 instruction support (using: libswrAVX2.so).`, or whichever
highest vector instruction set is available for your system and OpenSWR build.

Once VisIt loads up, you should see two windows open up: the main window and a
viewer window.

### Main window

The main window is the jumping point for all other features in VisIt. This is
also where you start creating "plots" to render in the viewer.

Click on either File -> Open file... or click the Open button under Sources
towards the top of the main window.  Navigate to a directory where you have a
dataset.  If you have a time series dataset, you'll find that VisIt will group
them together in the right column.  Opening this group will tell VisIt to try
to open the group as a time series that you can animate through.  If you want
to disable this and specifically open a single member of the group, turn File
grouping from Smart to Off.  Open a single volume dataset for now -- e.g.
`ironProt.vtk` or `ironProt.nc`.

Create a basic isosurface render by creating a Contour plot. Under the Plots
section of the main window, click Add -> Contour -> data.  Note that "data" is
the name of the variable in the dataset, so it will be different depending on
what you are opening.  You should see a Contour plot added to the list in the
main window. Double click it to bring up the Contour plot attributes window.
Here you can specify how you want the contouring filter to run, what isovalues
to use, the color map, and so on.

As a quick description, the "select by" option tells the filter how to
determine isovalues.  You can let VisIt automatically determine N equally-space
isovalues with "N levels", choose specific values within the data range with
"Value(s)", or choose specific relative positions within the data range with
"Percent(s)".  For this example, I set it to Value(s) and put "50 100 150" in
the box to specify those three values in our range 0-255.  I also set the color
map by switching the Contour Colors section to use Color table, then chose
Viridis.  Click Apply in the bottom left to apply all these changes.  You can
move this window to the side, but don't close it yet.

Now that the plot is ready. Click Draw under the Plots section. You should see
the plot appear in the viewer window.

### Viewer window

- left click and drag - rotate
- scroll mouse wheel - zoom with increments
- middle click and drag - zoom smoothly
- Ctrl+left click and drag - pan

You should see a purple surface with a teal and yellow surface within it.  By
default VisIt displays a bounding box, axes markers, dataset (database)
information, color map information, a triad, and your user information.

You can reset the view by pressing the camera button with four inward-pointing
arrows.  You can click the camera with a floppy disk button to save a specific
view point. This will create a new camera button with a number. Click that at
any time to return to the saved view point. Click the red crossed-out camera
with floppy to clear all saved views.

The black/white square with two arrows flips the foreground and background
colors.  By default the foreground (annotation color) is black and the
background is white.  The button to the left of this, the cylinder with an
arrow around it, toggles autorotation.  With autorotation enabled, you can
provide an initial rotation to the plot with a click+drag, and the "intertia"
will cause it to keep spinning. This option is great for showing off those
fancy surfaces, especially when specular lighting is enabled.

## Window layout

## Scripting VisIt

## Plots and operators

## Keyframing and animation

## Running a parallel engine

## Running a remote engine

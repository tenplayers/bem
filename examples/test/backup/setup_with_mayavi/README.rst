BEM
===


License
-------

see COPYING

Introduction
------------

Wraps the fastlap BEM (boundary element method aka BIE boundary integral
equations) + FMM (fast multipole method) + Laplace kernel implementation
from http://www.rle.mit.edu/cpg/research_codes.htm The original code is
in doc/fastlap_fl-2.0-22oct96 for reference.

Python Environment Setup
------------------------

04/2022 Wenhao He

For users:

Python version <= 3.9 is required. 

Anaconda environment tool is recommended:

    $ conda create -n ele39 python=3.9
    
    $ conda activate ele39

To set up (in folder of setup.py, setup.cfg, pyproject.toml and MANIFEST.in):

    $ pip install .

The workflow of our package is explained in examples/SimpleTrap/SimpleTrap_0305.py

4/23/2022: tested compatible with latest vtk==9.0.3, latest mayavi==4.7.4, and old PyQt5==5.15.6 on python==3.9.12 on both Linux(Ubuntu 20.04.3) and macOS(Monterey 12.1 intel). Newer python versions do not supports mayavi, more tests are needed on Windows. CentOs supports all parts except for mayavi.

(When using Ubuntu, you may have to make your PyQt5 5.15.6 work first: set export QT_DEBUG_PLUGINS=1 in ~/.bashrc and install libxcb-xinerama0)


General Notes
-------------

* The revised workflow for 3D electrostatics is currently in the
  "big_rewrite" branch in svn. It may move to trunk later.

* Most of the bugs and problems mentioned in "BEM software -
  Development" on the wiki are solved.

* The old .cpx "init" style configuration file is gone. Replaced with
  Python code. More flexible, less coding overhead when implementing new
  features. Compare the old cpx with the new python example.

* The input geometry is not the inventor-meshed .cpy file anymore but
  the face loops .ele or .trap or inventor exported .stl. You need to
  re-export your files or write a cpy-to-trap convertor (should not be too
  difficult).

* The meshing is now done preferably with Jonathan Shewchuk's triangle
  code. It is integrated into the Python package and compiled into a
  python module, see triangle and pytriangle.pyx

* The Fastlap code is also integrated and compiled into a python module
  (see fastlap/ and fastlap.pyx and called directly without exporting and
  importing intermediate file formats.

* The meshing can be adaptive: for a given potential configuration, the
  charge distribution for a small initial mesh is solved and the mesh is
  refined such that each triangle in the final mesh contributes equally to
  the field at the observation point (since the triangles are also
  constrained by the geometry, min angle etc, the final number of
  triangles is typically larger than the desired one).

* Triangle areas can also be constrained via differently shaped constraint
  fields (Box, Sphere, Cylinder or BYO).


STL
---

Ryan Bowler, 2014

If you wish to generate your own STL for the SimpleTrap, there are some
important features. The code scales the trap as if it is in units if
microns, so when exporting to STL in Inventor, be sure to choose microns
for the Units. There are no curved surfaces, so Surface and Normal
deviations are not important. Set max edge length to something
reasonable or else triangles are too small or large (40 microns, the ion
height from the surface, is a good choice). Choose a low aspect ratio
and make sure to export the colors.


Notes
-----

Old notes regarding multipole expansion and jumps in potentials/fields:

    The 'slfcc - Precise.exe' version is meant to solve the following
    problem. It can happen that the center of your trapping region is
    right on the boundary between two "cells" of the tree structure
    built by FastLap for the multipole-accelerated algorithm. In this
    case the calculated potentials and fields will show tiny "jumps" in
    their values when going across this boundary. This has usually no
    noticeable effect on potentials, but can be noticeable on the field
    and hence pseudopotential.

    One way to solve this problem is to add dummy electrodes on the side
    of your real electrodes, so that the spatial structure of the tree
    is shifted a bit. This would displace the cell boundary out of the
    center of your trap.

    The other way is using 'slfcc - Precise.exe', which skips the
    multipole acceleration procedure when calculating potentials and
    fields. In other words, it does an exact calculation based on the
    solved charge distribution, without using any tree structure. This
    increases the computing time and memory requirements, but yields a
    slightly more precise result. Note that the charge solving part of
    the algorithm is not modified (= it uses multipole acceleration,
    with a depth set in script 'runBEM.py').

    -> In the new python code this is achieved by passing "num_lev=1" to
    Job.simulate().


Some File descriptions
----------------------

The cpx&cpy.reg file assumes a root directory C:\BEMcode
The vtk.reg file assumes a directory C:\Program Files\ParaView\

Examples\TesSphere_1mm\       1mm tessellated sphere
Examples\SimpleTrap\          Simple Signe style trap
Examples\Skull trap\          Skull trap outline to test Inventor import macros


Notes for software developers:
------------------------

* exact dependent package version is listed in 'environment/' for reference. 
  If mayavi can't be built, you should try to restrict the version of vtk, PyQt5 , mayavi in setup.py first.
  
* mayavi with its dependence vtk, PyQt5 is very sensitive to environment. For this version, keep vtk<9.1 is important.Their future versions should be handled carefully.

* we use setup.py, setup.cfg, pyproject.toml and MANIFEST.in instead of a single setup.py file. setup.py is almost unchanged, setup.cfg fixes the argument build_ext --inplace, and pyproject.toml installs numpy and Cypython before setup.py runs.

* When debugging, you may 1. set pmap = map (serial map) instead of parallel computation 2. create an environment without running setup.py and add father folder of BEM to the system path, so that you can set break point in BEM

* When testing on Centos, our codes can run except for the part of mayavi. Maybe set python=3.6 is better for CentOS.

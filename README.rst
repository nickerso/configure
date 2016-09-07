==============
GET Configured
==============

A CMake project to configure (some of) my projects and their dependencies and build them from source.

Usage
=====

The following is a list of requirements required for building on all platforms

#. Toolchain (Visual Studio, GNU/GCC, Clang)
#. Git distributed version control system
#. CMake version 3.3 or greater
#. Python, including development header files (needed for LLVM+Clang build)

``get-configured`` uses CMake `external projects <https://cmake.org/cmake/help/v3.5/module/ExternalProject.html>`_ to download, configure, and build the full range of dependencies and some of the GET projects. Currently you can turn the various dependencies and projects on or off, depending on whether you want to build a given project from source or if you have an alternate method to provide that requirement (i.e., system package manager).

By default, all dependencies and projects are downloaded, built, and installed. The resulting install tree will provide you what you need to make use of the GET project libraries and applications. You will also have suitably configured build trees for each of the projects.

Ninja Generator
+++++++++++++++

`Ninja <https://ninja-build.org/>`_ is a nice and fast build tool available for many platforms. A default ``get-configured`` session would be something like as follows::

    $> git clone https://github.com/nickerso/get-configured
    $> mkdir GET_ROOT; cd GET_ROOT
    $> cmake -G Ninja ../get-configured -DCMAKE_INSTALL_PREFIX=/full/path/to/install/root
    $> cmake --build .

It may be useful to include some part of the install prefix that is specific to the build being configured. This should result in tree under ``GET_ROOT`` something like:

* ``.``: various CMake generated files
* ``Dependencies``: this is the root for all the non-GET projects that are required. Following a successful build this folder can be deleted.

    * ``Build``: the build trees for each of the dependencies
    * ``Source``: the source trees for each fo the dependencies

* ``GET``: the top-level folder for the GET projects.

    * ``csim``: the source folder for CSim...
    * ``build/CMAKE_BUILD_TYPE``: the build trees for each of the GET projects

And the ``/full/path/to/install/root`` will contain the built projects and the requried CMake config files that are needed to make use of these projects. The GET projects will be configured to make use of the installed dependencies, so once you have built the dependencies for your target configuration you should be able to remove the ``Dependencies`` folder from within ``GET_ROOT``.

On successful completion, you should be able to develop the GET projects configured in ``GET_ROOT/GET``. If you want to configure a new build of a GET project without needing to build/install the dependencies, just configure as usual with CMake and be sure to point ``CMAKE_PREFIX_PATH`` at the ``cmake`` folder in the appropriate installed location (e.g., ``/full/path/to/install/root/cmake``) or other location where you have the required dependencies installed.

Windows
*******

When choosing to use the Ninja generator on Windows make sure you set the environment correctly using the ``vcvarsall.bat`` script, for example to set a 64 bit build for Visual Studio 2015 in a cmd window::

"C:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\vcvarsall.bat" x64

or use the appropriate command console from the Visual Studio development tools.

Visual Studio
+++++++++++++

Follow these instructions for building 64 bit GET projects from the command line using Visual Studio.

$> git clone https://github.com/nickerso/get-configured
$> mkdir GET_ROOT; cd GET_ROOT
$> cmake -G "Visual Studio 14 2015 Win64" ../get-configured -DCMAKE_INSTALL_PREFIX=/full/path/to/install/root
$> cmake --build . --config Release

It is also possible to build from the Visual Studio solution in the ``GET_ROOT`` folder.


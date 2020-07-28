Linux 
======


Find a library on the system
--------------------------------

There seems to be multiple ways to do this, and sometimes one command works over another, not sure why.

.. code-block:: bash

        $ ldconfig -p | grep "name-of-lib"

.. code-block:: bash

        $ dpkg -L "name-of-lib"

Requires installing apt-file

.. code-block:: bash

        $ apt-file search "name-of-lib


ldd - print shared object dependencies. Very useful for debugging missing shared libraries.

.. code-block:: bash

    $ ldd $(which curl)

Can also try grep with ls -R

.. code-block:: bash

    $ ls -R | grep file

Then there is find

.. code-block:: bash

    $ find . -name "*sql*"

Building on linux
==================

Linking static libraries into shared
---------------------------------------

When passing arguments to the linker you need to ensure you use the `-Wl,--whole-archive` and
`-Wl,--no-whole-archive` option. Wrap these around static libraries that you are tyring
to pull into a shared library.

.. code-block:: bash

    -Wl,--whole-archive
    -lxml2
    -Wl,--no-whole-archive

This is necessary to tell the linker to pull all the functions from the library into the shared library you are
building. Otherwise, only some will be pulled in and you will get a linker error.

It seems there is also another way `here <https://stackoverflow.com/questions/6578484/telling-gcc-directly-to-link-a-library-statically>`_

Use -l: instead of -l. For example -l:libXYZ.a to link with libXYZ.a. Notice the lib written out, as opposed to
-lXYZ which would auto expand to libXYZ.

Note, these commands can be embedded into a CMake script by passing to `TARGET_LINK_LIBRARIES`

.. code-block:: cmake

    TARGET_LINK_LIBRARIES(target SHARED -W,l--whole-archive l:xml2 -Wl,no-whole-archive)

Inspecting broken builds
------------------------

List all the shared object libraries that libx depends on

.. code-block:: bash

    ldd libx.so

List the symbols in a library, along with their status (found, undefined etc.)

.. code-block:: bash

    nm libx.so

Use the -D option to inspect dynamic symbols only

.. code-block:: bash

    nm libx.so

Pipe output of nm into grep to search for specific function

.. code-block:: bash

    nm libx.so | grep somefunction

You can examine the Rpath on Linux thus:

.. code-block:: bash

    readelf -d libsemsim.so



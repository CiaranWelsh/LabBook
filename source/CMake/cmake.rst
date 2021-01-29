CMake
======


Cross platform CMake
----------------------

https://gitlab.kitware.com/cmake/community/-/wikis/doc/tutorials/How-To-Write-Platform-Checks


Note: cmake is now on pip version 3.17. pip install cmake.

Copy or install a file
------------------------

Copy during configuration stage

.. code-block:: cmake

    file(COPY ${LIBXML2_LIBRARY} DESTINATION ${PYSEMSIM_DIR})

Copy at install time

.. code-block:: cmake

        install(FILES ${LIBXML2_LIBRARY}
        DESTINATION ${PYSEMSIM_DIR})



API Control
-----------

We should consider both what IS in our API and what isn't. Public header files are okay, but its possible
for develops to still use things you don't want them to. Instead we can use symbol visibility. Heres a class

.. code-block:: C++

    class MyGenerator {
    public:
        int nextValue();
    };


With visual studio DLLs, this class would be hidden by default. However, on GCC and Clang, this class
is visible by default.

On visual studio :ref:`_declspec` you use `__declspec(export)` to change visibility from hidden to visible.



Watch this video: https://www.youtube.com/watch?v=m0DwB4OvDXk
And make notes here!.
































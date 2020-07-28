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

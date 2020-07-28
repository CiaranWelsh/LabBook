DLLs
=====

Creating a DLL and loading functions from it
--------------------------------------------

Here's a little library that can be compiled as a dll:

.. code-block:: c++

    // Hello.cpp
    extern "C" char const * __cdecl GetGreeting()
        {
            return "Hello, C++ Programmers!";
        }

.. code-block:: c++

    // PrintGreeting.cpp
    #include <stdio.h>
    #include <Windows.h>

    int main(){
        HMODULE const HelloDll = LoadLibraryExW(L"test.dll", nullptr, 0);

        /*
         * GetGreetingType is a function pointer for the type we want to load from Hello.dll
         */
        using GetGreetingType = char const* (__cdecl*)();

        // then we load get greeting, casting to the type we loaded.
        GetGreetingType const GetGreeting = reinterpret_cast<GetGreetingType>(
            GetProcAddress(
                HelloDll, "GetGreeting"));

        puts(GetGreeting());

        FreeLibrary(HelloDll);
    }

You can compile this using visual studio developer command prompt. The /c flag tells cl only to compile and not also link Hello.cpp

.. code-block:: cmd

    > cl.exe /c Hello.cpp

We have just created Hello.obj. Now we can link into a dll:

.. code-block:: cmd

    > link.exe Hello.obj /DLL /NOENTRY /EXPORT:GetGreeting

The DLL flag specifies to create a DLL. The NOENTRY flag tells the linker that the dll
does not have an entry point and the /EXPORT:GetGreeting tells the linker which functions from the DLL
are going to be exported into another library.


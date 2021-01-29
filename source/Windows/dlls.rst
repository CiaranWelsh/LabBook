=====
DLLs
=====

Lots of information here is from watching a `lecture on YouTube <https://www.youtube.com/watch?v=JPQWQfDhICA>`_

Explicit Linking
=================

Creating a DLL and loading functions from it
--------------------------------------------
.. _dlls:

Here's a little library that can be compiled as a dll:

.. code-block:: c++

    // Hello.cpp
    extern "C" char const * __cdecl GetGreeting()
        {
            return "Hello, C++ Programmers!";
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

Now, since this is a dll, we need another program, the client program to load `GetGreeting()`
and use it.

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

We can compile, link and run this program:

.. code-block:: powershell

    cl PrintGreeting.cpp
    .\PrintGreeting.exe

Which prints out:

.. code-block::

    Hello, C++ Programmers!



Using dumpbin.exe
-----------------

Dumpbin is a program for parsing windows binaries. Note, on windows you can
you "/" or "-" to indicate that what follows is an option. Additionally,
the commands are case insensitive.

There are a bunch of headers or metadata inside the dll that can be
interrogated using:

DLL Headers
-------------

.. code-block:: powershell

    dumpbin /HEADERS Hello.

DLLs have a predefined structure. First, a bunch of header sections follewed by
a number of sections, which contain actual code, data and resources in the dll.

The section headers told us where to find the data in the file. We can look at
whats actually inside of a section using the `-rawdata` flag.


DLL Raw data
------------

.. code-block:: PowerShellLexer

    dumpbin -rawdata -section:.text Hello.dll

.. code-block::

    D:\TestStaticIntoSharedLinking\cmake-build-release-visual-studio\dynamic_lib\test>dumpbin -rawdata -section:.text test.dll
    Microsoft (R) COFF/PE Dumper Version 14.26.28806.0
    Copyright (C) Microsoft Corporation.  All rights reserved.


    Dump of file test.dll

    File Type: DLL

    SECTION HEADER #1
       .text name
           A virtual size
        1000 virtual address (10001000 to 10001009)
         200 size of raw data
         400 file pointer to raw data (00000400 to 000005FF)
           0 file pointer to relocation table
           0 file pointer to line numbers
           0 number of relocations
           0 number of line numbers
    60000020 flags
             Code
             Execute Read

    RAW DATA #1
      10001000: 55 8B EC B8 00 20 00 10 5D C3                    U.ì¸. ..]Ã

      Summary

            1000 .text


So it contains some bytes. We can also disassemble the bytes:

Disassembley
------------


.. code-block:: powershell

    D:\TestStaticIntoSharedLinking\cmake-build-release-visual-studio\dynamic_lib\test>dumpbin /disasm -section:.text Hello.dll
    Microsoft (R) COFF/PE Dumper Version 14.26.28806.0
    Copyright (C) Microsoft Corporation.  All rights reserved.


    Dump of file Hello.dll

    File Type: DLL

    SECTION HEADER #1
       .text name
           A virtual size
        1000 virtual address (10001000 to 10001009)
         200 size of raw data
         400 file pointer to raw data (00000400 to 000005FF)
           0 file pointer to relocation table
           0 file pointer to line numbers
           0 number of relocations
           0 number of line numbers
    60000020 flags
             Code
             Execute Read

      10001000: 55                 push        ebp
      10001001: 8B EC              mov         ebp,esp
      10001003: B8 00 20 00 10     mov         eax,10002000h
      10001008: 5D                 pop         ebp
      10001009: C3                 ret

      Summary

            1000 .text



RData
------

.. code-block:: powershell

    D:\TestStaticIntoSharedLinking\cmake-build-release-visual-studio\dynamic_lib\test>dumpbin /rawdata -section:.rdata test.dll
    Microsoft (R) COFF/PE Dumper Version 14.26.28806.0
    Copyright (C) Microsoft Corporation.  All rights reserved.


    Dump of file test.dll

    File Type: DLL

    SECTION HEADER #2
      .rdata name
          D8 virtual size
        2000 virtual address (10002000 to 100020D7)
         200 size of raw data
         600 file pointer to raw data (00000600 to 000007FF)
           0 file pointer to relocation table
           0 file pointer to line numbers
           0 number of relocations
           0 number of line numbers
    40000040 flags
             Initialized Data
             Read Only

    RAW DATA #2
      10002000: 48 65 6C 6C 6F 2C 20 43 2B 2B 20 50 72 6F 67 72  Hello, C++ Progr
      10002010: 61 6D 6D 65 72 73 21 00 00 00 00 00 3B 0A 20 5F  ammers!.....;. _
      10002020: 00 00 00 00 0D 00 00 00 50 00 00 00 88 20 00 00  ........P.... ..
      10002030: 88 06 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
      10002040: 00 00 00 00 FF FF FF FF 00 00 00 00 72 20 00 00  ....ÿÿÿÿ....r ..
      10002050: 01 00 00 00 01 00 00 00 01 00 00 00 68 20 00 00  ............h ..
      10002060: 6C 20 00 00 70 20 00 00 00 10 00 00 7B 20 00 00  l ..p ......{ ..
      10002070: 00 00 74 65 73 74 2E 64 6C 6C 00 47 65 74 47 72  ..test.dll.GetGr
      10002080: 65 65 74 69 6E 67 00 00 00 00 00 00 00 10 00 00  eeting..........
      10002090: 0A 00 00 00 2E 74 65 78 74 24 6D 6E 00 00 00 00  .....text$mn....
      100020A0: 00 20 00 00 40 00 00 00 2E 72 64 61 74 61 00 00  . ..@....rdata..
      100020B0: 40 20 00 00 48 00 00 00 2E 65 64 61 74 61 00 00  @ ..H....edata..
      100020C0: 88 20 00 00 50 00 00 00 2E 72 64 61 74 61 24 7A  . ..P....rdata$z
      100020D0: 7A 7A 64 62 67 00 00 00                          zzdbg...

      Summary

            1000 .rdata

Note that we can see where our string is stored. Moreover, the locations of the Export
and Debug directories are also located in here.

DLL Exports
-------------

The export directory defines the public service of the dll, all the things
that other dlls or exes can use from this dll. We can look at these with:

.. code-block:: powershell

    D:\TestStaticIntoSharedLinking\cmake-build-release-visual-studio\dynamic_lib\test>dumpbin -exports test.dll
    Microsoft (R) COFF/PE Dumper Version 14.26.28806.0
    Copyright (C) Microsoft Corporation.  All rights reserved.


    Dump of file test.dll

    File Type: DLL

      Section contains the following exports for test.dll

        00000000 characteristics
        FFFFFFFF time date stamp
            0.00 version
               1 ordinal base
               1 number of functions
               1 number of names

        ordinal hint RVA      name

              1    0 00001000 GetGreeting

      Summary

            1000 .rdata
            1000 .reloc
            1000 .text

    D:\TestStaticIntoSharedLinking\cmake-build-release-visual-studio\dynamic_lib\test>


To reiterate, this command lists the functions that other dlls can import into
their program for use using `LoadLibrary`

DLL Depencencies
----------------

.. code-block:: powershell

    D:\TestStaticIntoSharedLinking\cmake-build-release-visual-studio\dynamic_lib\test> dumpbin -dependents PrintGreeting.exe
    Microsoft (R) COFF/PE Dumper Version 14.26.28806.0
    Copyright (C) Microsoft Corporation.  All rights reserved.


    Dump of file PrintGreeting.exe

    File Type: EXECUTABLE IMAGE

      Image has the following dependencies:

        KERNEL32.dll

      Summary

            2000 .data
            6000 .rdata
            1000 .reloc
            D000 .text



DLL Imports
------------


.. code-block:: powershell

    D:\TestStaticIntoSharedLinking\cmake-build-release-visual-studio\dynamic_lib\test>dumpbin -imports PrintGreeting.exe
    Microsoft (R) COFF/PE Dumper Version 14.26.28806.0
    Copyright (C) Microsoft Corporation.  All rights reserved.


    Dump of file PrintGreeting.exe

    File Type: EXECUTABLE IMAGE

      Section contains the following imports:

        KERNEL32.dll
                    40E000 Import Address Table
                    4133AC Import Name Table
                         0 time date stamp
                         0 Index of first forwarder reference

                      1AB FreeLibrary
                      2AE GetProcAddress
                      3C3 LoadLibraryExW
                      44D QueryPerformanceCounter
                      218 GetCurrentProcessId
                      21C GetCurrentThreadId
                      2E9 GetSystemTimeAsFileTime
                      363 InitializeSListHead
                      37F IsDebuggerPresent
                      5AD UnhandledExceptionFilter
                      56D SetUnhandledExceptionFilter
                      2D0 GetStartupInfoW
                      386 IsProcessorFeaturePresent
                      278 GetModuleHandleW
                      217 GetCurrentProcess
                      58C TerminateProcess
                      611 WriteConsoleW
                      4D3 RtlUnwind
                      261 GetLastError
                      532 SetLastError
                      131 EnterCriticalSection
                      3BD LeaveCriticalSection
                      110 DeleteCriticalSection
                      35F InitializeCriticalSectionAndSpinCount
                      59E TlsAlloc
                      5A0 TlsGetValue
                      5A1 TlsSetValue
                      59F TlsFree
                      462 RaiseException
                      2D2 GetStdHandle
                      612 WriteFile
                      274 GetModuleFileNameW
                      15E ExitProcess
                      277 GetModuleHandleExW
                      1D6 GetCommandLineA
                      1D7 GetCommandLineW
                      24E GetFileType
                      345 HeapAlloc
                      349 HeapFree
                      175 FindClose
                      17B FindFirstFileExW
                      18C FindNextFileW
                      38B IsValidCodePage
                      1B2 GetACP
                      297 GetOEMCP
                      1C1 GetCPInfo
                      3EF MultiByteToWideChar
                      5FE WideCharToMultiByte
                      237 GetEnvironmentStringsW
                      1AA FreeEnvironmentStringsW
                      514 SetEnvironmentVariableW
                      54A SetStdHandle
                      2D7 GetStringTypeW
                       9B CompareStringW
                      3B1 LCMapStringW
                      2B4 GetProcessHeap
                      24C GetFileSizeEx
                      523 SetFilePointerEx
                      1EA GetConsoleCP
                      1FC GetConsoleMode
                      34E HeapSize
                      34C HeapReAlloc
                      19F FlushFileBuffers
                       86 CloseHandle
                       CB CreateFileW
                      109 DecodePointer

      Summary

            2000 .data
            6000 .rdata
            1000 .reloc
            D000 .text




Implicit Linking
================

Before, we use explicit linking to LoadLibrary and GetProcAddress
for specific functions from the library we were using. Now we look at implicit
linking.

Where explicit linking means you physically load the library in your program,
with implicit linking you are providing a *.lib file, which contains the
information needed for a program to implicitely link. Remember that this .lib
is not the same as that produced when building a static library. Instead, it
is a stub file that gets used to create function pointers automatically.

We want this to work:

.. code-block:: C++

    // PrintGreetingImplicityLinking.cpp
    #include <stdio.h>

    extern "C" const char* __cdecl GetGreeting();

    int main(){
        puts(GetGreeting());
    }

You can use

.. code-block:: powershell

    dumpbin -all Hello.lib

To look in detail at the *lib file. It gives us information such as
which functions are available for linking, where they live etc.

We can compile and link:

.. code-block:: powershell

    D:\TestStaticIntoSharedLinking\cmake-build-release-visual-studio\dynamic_lib\test>cl -c PrintGreetingImplicityLinking.cpp
    Microsoft (R) C/C++ Optimizing Compiler Version 19.26.28806 for x86
    Copyright (C) Microsoft Corporation.  All rights reserved.

    PrintGreetingImplicityLinking.cpp

    D:\TestStaticIntoSharedLinking\cmake-build-release-visual-studio\dynamic_lib\test>link PrintGreetingImplicityLinking.obj Hello.lib
    Microsoft (R) Incremental Linker Version 14.26.28806.0
    Copyright (C) Microsoft Corporation.  All rights reserved.

    D:\TestStaticIntoSharedLinking\cmake-build-release-visual-studio\dynamic_lib\test>PrintGreetingImplicityLinking.exe
    Hello, C++ Programmers!


We can look at its dependents:

.. code-block:: powershell

    D:\TestStaticIntoSharedLinking\cmake-build-release-visual-studio\dynamic_lib\test>dumpbin /dependents PrintGreetingImplicityLinking.exe
    Microsoft (R) COFF/PE Dumper Version 14.26.28806.0
    Copyright (C) Microsoft Corporation.  All rights reserved.


    Dump of file PrintGreetingImplicityLinking.exe

    File Type: EXECUTABLE IMAGE

      Image has the following dependencies:

        Hello.dll
        KERNEL32.dll

      Summary

            2000 .data
            6000 .rdata
            1000 .reloc
            D000 .text


Relealing that our PrintGreetingImplicitlLinking.exe depends on both Hello.dll and
KERNEL32.dll, where our explicitely linked program only depended on KERNEL32.dll.


We can check our imports:

.. code-block:: powershell

    D:\TestStaticIntoSharedLinking\cmake-build-release-visual-studio\dynamic_lib\test>dumpbin /imports PrintGreetingImplicityLinking.exe
    Microsoft (R) COFF/PE Dumper Version 14.26.28806.0
    Copyright (C) Microsoft Corporation.  All rights reserved.


    Dump of file PrintGreetingImplicityLinking.exe

    File Type: EXECUTABLE IMAGE

      Section contains the following imports:

        Hello.dll
                    40E000 Import Address Table
                    4133A0 Import Name Table
                         0 time date stamp
                         0 Index of first forwarder reference

                        0 GetGreeting

        KERNEL32.dll
                    40E008 Import Address Table
                    4133A8 Import Name Table
                         0 time date stamp
                         0 Index of first forwarder reference

                      44D QueryPerformanceCounter
                      218 GetCurrentProcessId
                      21C GetCurrentThreadId
                      2E9 GetSystemTimeAsFileTime
                      363 InitializeSListHead
                      37F IsDebuggerPresent
                      5AD UnhandledExceptionFilter
                      56D SetUnhandledExceptionFilter
                      2D0 GetStartupInfoW
                      386 IsProcessorFeaturePresent
                      278 GetModuleHandleW
                      217 GetCurrentProcess
                      58C TerminateProcess
                      611 WriteConsoleW
                      4D3 RtlUnwind
                      261 GetLastError
                      532 SetLastError
                      131 EnterCriticalSection
                      3BD LeaveCriticalSection
                      110 DeleteCriticalSection
                      35F InitializeCriticalSectionAndSpinCount
                      59E TlsAlloc
                      5A0 TlsGetValue
                      5A1 TlsSetValue
                      59F TlsFree
                      1AB FreeLibrary
                      2AE GetProcAddress
                      3C3 LoadLibraryExW
                      462 RaiseException
                      2D2 GetStdHandle
                      612 WriteFile
                      274 GetModuleFileNameW
                      15E ExitProcess
                      277 GetModuleHandleExW
                      1D6 GetCommandLineA
                      1D7 GetCommandLineW
                      24E GetFileType
                      345 HeapAlloc
                      349 HeapFree
                      175 FindClose
                      17B FindFirstFileExW
                      18C FindNextFileW
                      38B IsValidCodePage
                      1B2 GetACP
                      297 GetOEMCP
                      1C1 GetCPInfo
                      3EF MultiByteToWideChar
                      5FE WideCharToMultiByte
                      237 GetEnvironmentStringsW
                      1AA FreeEnvironmentStringsW
                      514 SetEnvironmentVariableW
                      54A SetStdHandle
                      2D7 GetStringTypeW
                       9B CompareStringW
                      3B1 LCMapStringW
                      2B4 GetProcessHeap
                      24C GetFileSizeEx
                      523 SetFilePointerEx
                      1EA GetConsoleCP
                      1FC GetConsoleMode
                      34E HeapSize
                      34C HeapReAlloc
                      19F FlushFileBuffers
                       86 CloseHandle
                       CB CreateFileW
                      109 DecodePointer

      Summary

            2000 .data
            6000 .rdata
            1000 .reloc
            D000 .text

Which indicates that we import our GetGreeting function from Hello.lib/Hello.dll.

Exporting from a DLL
====================

We create a new example to work with.

.. code-block:: C++

    // Numbers.cpp
    extern "C" int GetOne() {return 1;}
    extern "C" int GetTwo() {return 2;}
    extern "C" int GetThree() {return 3;}

Lets compile:

.. code-block:: powershell

    cl -c Numbers.cpp

We have 4 options for exporting these function to make them available for

Export flag command line
-------------------------

So far we've been using Export.

.. code-block:: powershell

    D:\TestStaticIntoSharedLinking\cmake-build-release-visual-studio\dynamic_lib\test>link Numbers.obj /NOENTRY /DLL /EXPORT:GetOne /EXPORT:GetTwo /EXPORT:GetThree
    Microsoft (R) Incremental Linker Version 14.26.28806.0
    Copyright (C) Microsoft Corporation.  All rights reserved.

       Creating library Numbers.lib and object Numbers.exp

    D:\TestStaticIntoSharedLinking\cmake-build-release-visual-studio\dynamic_lib\test>dumpbin /exports Numbers.dll
    Microsoft (R) COFF/PE Dumper Version 14.26.28806.0
    Copyright (C) Microsoft Corporation.  All rights reserved.


    Dump of file Numbers.dll

    File Type: DLL

      Section contains the following exports for Numbers.dll

        00000000 characteristics
        FFFFFFFF time date stamp
            0.00 version
               1 ordinal base
               3 number of functions
               3 number of names

        ordinal hint RVA      name

              1    0 00001000 GetOne
              2    1 00001020 GetThree
              3    2 00001010 GetTwo

      Summary

            1000 .rdata
            1000 .text

We can also export under alias's.

.. code-block:: powershell

    D:\TestStaticIntoSharedLinking\cmake-build-release-visual-studio\dynamic_lib\test>link Numbers.obj /NOENTRY /DLL /EXPORT:GetOne /EXPORT:GetTwo /EXPORT:GetThree /EXPORT:GetOnePlusTwo=GetThree
    Microsoft (R) Incremental Linker Version 14.26.28806.0
    Copyright (C) Microsoft Corporation.  All rights reserved.

       Creating library Numbers.lib and object Numbers.exp

    D:\TestStaticIntoSharedLinking\cmake-build-release-visual-studio\dynamic_lib\test>dumpbin /exports Numbers.dll
    Microsoft (R) COFF/PE Dumper Version 14.26.28806.0
    Copyright (C) Microsoft Corporation.  All rights reserved.


    Dump of file Numbers.dll

    File Type: DLL

      Section contains the following exports for Numbers.dll

        00000000 characteristics
        FFFFFFFF time date stamp
            0.00 version
               1 ordinal base
               4 number of functions
               4 number of names

        ordinal hint RVA      name

              1    0 00001000 GetOne
              2    1 00001020 GetOnePlusTwo
              3    2 00001020 GetThree
              4    3 00001010 GetTwo

      Summary

            1000 .rdata
            1000 .text


.. note::

    GetOnePlusTwo and GetThree are the same function with a different name. They are at the same
    memory address.


Using a def file
-----------------

In a new file, Numbers.def, put the following:

.. code-block:: powershell

    LIBRARY Numbers
    EXPORTS
            GetOne
            GetTwo PRIVATE
            GetOnePlusTwo=GetThree




Now we can link with :

.. code-block:: powershell

    D:\TestStaticIntoSharedLinking\cmake-build-release-visual-studio\dynamic_lib\test>link Numbers.obj /DLL /NOENTRY /DEF:Numbers.def
    Microsoft (R) Incremental Linker Version 14.26.28806.0
    Copyright (C) Microsoft Corporation.  All rights reserved.

       Creating library Numbers.lib and object Numbers.exp

    D:\TestStaticIntoSharedLinking\cmake-build-release-visual-studio\dynamic_lib\test>dumpbin /exports Numbers.lib
    Microsoft (R) COFF/PE Dumper Version 14.26.28806.0
    Copyright (C) Microsoft Corporation.  All rights reserved.


    Dump of file Numbers.lib

    File Type: LIBRARY

         Exports

           ordinal    name

                      _GetOne
                      _GetOnePlusTwo

      Summary

              C3 .debug$S
              14 .idata$2
              14 .idata$3
               4 .idata$4
               4 .idata$5
               C .idata$6




Inside your code
----------------

.. _declspec:

Another option is to declare exports inside your code. Take a look at Numbers2.cpp.

.. code-block:: C++

    extern "C" __declspec(dllexport) int GetOne() { return 1;}
    extern "C" __declspec(dllexport) int GetTwo() { return 2;}
    extern "C" __declspec(dllexport) int GetThree() { return 3;}

We use __declspec(export) to do that same as what we were previously doing on the command line.
The

.. code-block:: powershell

    D:\TestStaticIntoSharedLinking\cmake-build-release-visual-studio\dynamic_lib\test>cl -c Numbers2.cpp
    Microsoft (R) C/C++ Optimizing Compiler Version 19.26.28806 for x86
    Copyright (C) Microsoft Corporation.  All rights reserved.

    Numbers2.cpp

    D:\TestStaticIntoSharedLinking\cmake-build-release-visual-studio\dynamic_lib\test>dumpbin /EXPORTS Numbers2.dll
    Microsoft (R) COFF/PE Dumper Version 14.26.28806.0
    Copyright (C) Microsoft Corporation.  All rights reserved.


    Dump of file Numbers2.dll

    File Type: DLL

      Section contains the following exports for Numbers2.dll

        00000000 characteristics
        FFFFFFFF time date stamp
            0.00 version
               1 ordinal base
               3 number of functions
               3 number of names

        ordinal hint RVA      name

              1    0 00001000 GetOne
              2    1 00001020 GetThree
              3    2 00001010 GetTwo

      Summary

            1000 .rdata
            1000 .text



Declspec export merely tells the compiler to pretend that it got the exports from the
command line. They do the same job but its more convenient.

Pragma
------

Pragma directives can also be used to achieve the same, though this is not often used.
So Numbers3.cpp looks like this.

.. code-block::c++

    extern "C" int GetOne() { return 1;}
    extern "C" int GetTwo() { return 2;}
    extern "C" int GetThree() { return 3;}

    #pragma comment(linker, "/export:GetOne")
    #pragma comment(linker, "/export:GetTwo")
    #pragma comment(linker, "/export:GetThree")



What happens when we load a DLL?
=================================

There are 5 steps, basically:

    1. Find the dll (Hello.dll)
    2. Map Hello.dll into memory
    3. Load any DLLs on which Hello.dll depends
    4. Bind imports from DLLs on which Hello.dll depends
    5. Call the entry point for Hello.dll to let it initialize itself.


Find the DLL
------------

When we do

.. code-block:: cpp

    HMODULE HelloDll = LoadLibraryExW(L"Hello.dll", nullptr, o);

How does the loader know where to find `Hello.dll`?

If we passed an absolute path to `LoadLibraryExW`, this is easy as if its
there it'll be loaded, if not it'll fail. Note, you can load the same library
into the same script from two different drives (C Vs D), but not two
libraries with the same name from the same drive.

If its not an absolute path then the first thing that happens is the loader
will look to see whether the dll is a system dll. These are always loaded
from the same place for security. These are well known to the OS and the same
version of the library will always be loaded. For instance, kernel32.dll or
ole32.dll. This mechanism prevents dll hijacking.

If the dll is not in this small list of libraries, the loader will continue with
the search process. This is the search process:

    1. The directory from which the application is loaded
    2. The system directoy (C:\Windows\System32\ or C\:Windows\SysWOW64\)
    3. The 16-bit system directory (C:\Windows\System\)
    4. The Windows Directory (C:\Windows\)
    5. The current directory
    6. The directories listed in %PATH% environment variable.

Once found, the search stops.

This process is highly customizable. For instance:

    1. DLL Redirection (.local)
    2. Side-by-size components
    3. add to %PATH%
    4. AddDllDirectory
    5. LoadLibraryEx Flags

Do some googling on these.


Map the DLL into Memory
-----------------------

The loader needs to

    1. Open the DLL file and read the image size
    2. Allocate a contiguous, page aligned block of memory of that size
    3. Copy the contents of each section into the appropriate area of that block of memory

Relocation
----------
DLLs have a preferred base address. If the dll does not get loaded into its preferred base
address then the pointers in the dll will be pointing to random slots of memory.
Relocation fixes this.



Load Dependencies and Bind Imports
-----------------------------------
For each DLL dependency:
    1. load the DLL
    2. Get the required imports to fill out the function pointer tables.


Initialize the DLL
---------------------

DLLs have an optional entry point where it can do some initialization. Conventially
this is called `DllMain` but can be called anything.

Here is the signature.

.. code-block:: C++

    BOOL WINAPI DllMain(HINSTANCE instance, DWORD reason, LPVOID reserved);

Where:

    - instance = the DLL handle returned from LoadLibrary
    - reason = indication of why the loaded is calling the entry point
        - DLL_PROCESS_ATTACH = Called once, when DLL is loaded
        - DLL_PROCESS_DETACH = Called once, when DLL is unloaded
        - DLL_THREAD_ATTACH = Called each time a thread starts running
        - DLL_THREAD_DETACH = Called each time a thread stops running
    - reserve = more information for process attach or detach.

Returns True or False depending on load success.

Calls to DllMain are syncronized by a gloval lock called the Loader Lock. So
only 1 thread can be initializing a dll at one time.



Debugging DLL Load Failures
-----------------------------

What if Hello.dll did not exist? Then you would get an error.
How do you debug this?

One way is to use a program called gflags.

Here I deleted Hello.dll. Now when we run a program that uses Hello.dll we get
and error.


.. code-block::powershell

    D:\TestStaticIntoSharedLinking\cmake-build-release-visual-studio\dynamic_lib\test>PrintGreetingImplicityLinking.exe



Importing
-----------

We've already seen `__declspec(dllexport)` which is used inside our source files to
allow other programs access to the public interface. `__declspec(dllimport)` also exists,
and this is used inside programs that `use` a dll.

For instance, see `NumbersCaller.cpp`.

.. code-block::powershell

    extern "C" __declspec(dllimport) int GetOne();
    extern "C" __declspec(dllimport) int GetTwo();
    extern "C" __declspec(dllimport) int GetThree();

The `__declspec(dllimport)` statement tells the compiler than this function
is going to be imported. This is more efficient because the compiler can
do things a little differently.


Exporting Data
--------------

You can export variables as well as functions. When you do this you need
to use __declspec(dllimport).


Exporting C++ classes
-----------------------

This is possible. When you use `__declspec(dllexport)` on a class, rather
than a function, all the members of the class get exported.

However, You are NOT recommended to do exports on classes. You are too dependent on
a compiler. This will be hard to debug and will probably do wrong.













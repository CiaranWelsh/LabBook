=====
DLLs
=====


Explicit Linking
=================

Creating a DLL and loading functions from it
--------------------------------------------

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

.. code-block:: powershell

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














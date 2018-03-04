Building on Windows using MinGW under Cygwin
--------------------------------------------

  Tested on an up-to-date Cygwin 2.5.2 under Windows 10,
  with link-grammar version 5.3.8.


Supported target versions
-------------------------
  The intention is to support versions from Vista on (some WIN32
  functions which are used are not supported in earlier versions.)
  The resulted link-parser executable is able to run under Cygwin too.

  The system compatibility definitions:
  In configure.ac:
     AC_DEFINE(NTDDI_VERSION, NTDDI_VISTA)
     AC_DEFINE(_WIN32_WINNT, _WIN32_WINNT_VISTA)


Configuring
-----------

  $ LDFLAGS=-L$gnuregex/lib CPPFLAGS=-I$gnuregex/include \
      configure -host=x86_64-w64-mingw32 \
      --enable-wordgraph-display \
      --disable-sat-solver \
      --disable-python-bindings \
      --disable-java-bindings

  In the configure command above, $gnuregex points to the directory of the
  POSIX regex for Windows. The library basename must b:e "libregex". The
  -host value is for compiling for 64-bits.

  The SAT solver cannot be enabled for now due to a missing definition in the
  build system. Python bindings fail because it currently tries to use the
  Cygwin Python system. More development work is needed on these.

  (The Java bindings has not been tested in this version. Most probably the
  way described in README.MSYS can be used.)

  $ make

  $ make install

  The dictionaries are installed by default under
  /usr/local/share/link-grammar.


Running
-------

  * From the sources directory, using cmd.exe Windows console:

  Note: ^Z/^D/^C are not supported when running under Cygwin!
  In particular, don't press ^Z - it may crash or stuck the window.
  To exit, use the !exit command.

      > PATH-TO-LG-CONF-DIRECTORY\link-parser\link-parser [arguments]

  * Form the Cygwin shell:

    Before installation:
      $ PATH-TO-LG-CONF-DIRECTORY/link-parser/link-parser [args]

    After "make install" (supposing /usr/local/bin is in the PATH):
      $ link-parser [arguments]

  To run the executable from other location, liblink-grammar-5.dll needs to be
  in a directory in PATH (e.g. in the directory of the executable).

  For more details, see "RUNNING the program" in the main README.

Limitations
-----------

  Since MinGW currently doesn't support locale_t and the isw*() functions
  that use it, the library doesn't support per-dictionary locale setting,
  which just means that if several dictionaries are used, all of them share
  the same global locale, so if they are used from different threads the
  must use languages with the same codeset.  If the program is not
  multi-threaded, dictionaries of several different languages can be created
  and then used one by one, provided that the global locale is switched
  (using setlocale()) to the locale of each dictionary just before using
  this dictionary.

  (To get a complete per-dictionary locale support, the library should be
  compiled using MSVC.)


Implementation Notes
--------------------

  MinGW uses by default a Windows STDIO from an unsupported Windows library
  that just happens to be included even in Windows 10. This STDIO is not C99
  compliant, and in particular doesn't support the %z formats (it crashes
  when it encounters them).

  Hence __USE_MINGW_ANSI_STDIO=1 is defined, so MinGW uses its own C99
  compatible STDIO. However, the *printf() functions of these implementation
  cannot print UTF-8 to the console (to files/pipe they print UTF-8 just
  fine), because they use Windows' putchar(), which cannot write UTF-8 to
  the console. A workaround is implemented in the LG library and in
  link-parser.

  If you write a C/C++ program (to be compiled with MinGW) that uses the
  library and needs to print to the console, and this problem is not fixed
  by then (in Windows or MinGW), then you need to copy this workaround
  implementation. See utilities.c and/or parser_utilities.c.

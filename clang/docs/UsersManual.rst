============================
Clang Compiler User's Manual
============================

.. include:: <isonum.txt>

.. contents::
   :local:

Introduction
============

The Clang Compiler is an open-source compiler for the C family of
programming languages, aiming to be the best in class implementation of
these languages. Clang builds on the LLVM optimizer and code generator,
allowing it to provide high-quality optimization and code generation
support for many targets. For more general information, please see the
`Clang Web Site <https://clang.llvm.org>`_ or the `LLVM Web
Site <https://llvm.org>`_.

This document describes important notes about using Clang as a compiler
for an end-user, documenting the supported features, command line
options, etc. If you are interested in using Clang to build a tool that
processes code, please see :doc:`InternalsManual`. If you are interested in the
`Clang Static Analyzer <https://clang-analyzer.llvm.org>`_, please see its web
page.

Clang is one component in a complete toolchain for C family languages.
A separate document describes the other pieces necessary to
:doc:`assemble a complete toolchain <Toolchain>`.

Clang is designed to support the C family of programming languages,
which includes :ref:`C <c>`, :ref:`Objective-C <objc>`, :ref:`C++ <cxx>`, and
:ref:`Objective-C++ <objcxx>` as well as many dialects of those. For
language-specific information, please see the corresponding language
specific section:

-  :ref:`C Language <c>`: K&R C, ANSI C89, ISO C90, ISO C94 (C89+AMD1), ISO
   C99 (+TC1, TC2, TC3).
-  :ref:`Objective-C Language <objc>`: ObjC 1, ObjC 2, ObjC 2.1, plus
   variants depending on base language.
-  :ref:`C++ Language <cxx>`
-  :ref:`Objective C++ Language <objcxx>`
-  :ref:`OpenCL Kernel Language <opencl>`: OpenCL C v1.0, v1.1, v1.2, v2.0,
   plus C++ for OpenCL.

In addition to these base languages and their dialects, Clang supports a
broad variety of language extensions, which are documented in the
corresponding language section. These extensions are provided to be
compatible with the GCC, Microsoft, and other popular compilers as well
as to improve functionality through Clang-specific features. The Clang
driver and language features are intentionally designed to be as
compatible with the GNU GCC compiler as reasonably possible, easing
migration from GCC to Clang. In most cases, code "just works".
Clang also provides an alternative driver, :ref:`clang-cl`, that is designed
to be compatible with the Visual C++ compiler, cl.exe.

In addition to language specific features, Clang has a variety of
features that depend on what CPU architecture or operating system is
being compiled for. Please see the :ref:`Target-Specific Features and
Limitations <target_features>` section for more details.

The rest of the introduction introduces some basic :ref:`compiler
terminology <terminology>` that is used throughout this manual and
contains a basic :ref:`introduction to using Clang <basicusage>` as a
command line compiler.

.. _terminology:

Terminology
-----------

Front end, parser, backend, preprocessor, undefined behavior,
diagnostic, optimizer

.. _basicusage:

Basic Usage
-----------

Intro to how to use a C compiler for newbies.

compile + link compile then link debug info enabling optimizations
picking a language to use, defaults to C17 by default. Autosenses based
on extension. using a makefile

Command Line Options
====================

This section is generally an index into other sections. It does not go
into depth on the ones that are covered by other sections. However, the
first part introduces the language selection and other high level
options like :option:`-c`, :option:`-g`, etc.

Options to Control Error and Warning Messages
---------------------------------------------

.. option:: -Werror

  Turn warnings into errors.

.. This is in plain monospaced font because it generates the same label as
.. -Werror, and Sphinx complains.

``-Werror=foo``

  Turn warning "foo" into an error.

.. option:: -Wno-error=foo

  Turn warning "foo" into a warning even if :option:`-Werror` is specified.

.. option:: -Wfoo

  Enable warning "foo".
  See the :doc:`diagnostics reference <DiagnosticsReference>` for a complete
  list of the warning flags that can be specified in this way.

.. option:: -Wno-foo

  Disable warning "foo".

.. option:: -w

  Disable all diagnostics.

.. option:: -Weverything

  :ref:`Enable all diagnostics. <diagnostics_enable_everything>`

.. option:: -pedantic

  Warn on language extensions.

.. option:: -pedantic-errors

  Error on language extensions.

.. option:: -Wsystem-headers

  Enable warnings from system headers.

.. option:: -ferror-limit=123

  Stop emitting diagnostics after 123 errors have been produced. The default is
  20, and the error limit can be disabled with `-ferror-limit=0`.

.. option:: -ftemplate-backtrace-limit=123

  Only emit up to 123 template instantiation notes within the template
  instantiation backtrace for a single warning or error. The default is 10, and
  the limit can be disabled with `-ftemplate-backtrace-limit=0`.

.. _cl_diag_formatting:

Formatting of Diagnostics
^^^^^^^^^^^^^^^^^^^^^^^^^

Clang aims to produce beautiful diagnostics by default, particularly for
new users that first come to Clang. However, different people have
different preferences, and sometimes Clang is driven not by a human,
but by a program that wants consistent and easily parsable output. For
these cases, Clang provides a wide range of options to control the exact
output format of the diagnostics that it generates.

.. _opt_fshow-column:

**-f[no-]show-column**
   Print column number in diagnostic.

   This option, which defaults to on, controls whether or not Clang
   prints the column number of a diagnostic. For example, when this is
   enabled, Clang will print something like:

   ::

         test.c:28:8: warning: extra tokens at end of #endif directive [-Wextra-tokens]
         #endif bad
                ^
                //

   When this is disabled, Clang will print "test.c:28: warning..." with
   no column number.

   The printed column numbers count bytes from the beginning of the
   line; take care if your source contains multibyte characters.

.. _opt_fshow-source-location:

**-f[no-]show-source-location**
   Print source file/line/column information in diagnostic.

   This option, which defaults to on, controls whether or not Clang
   prints the filename, line number and column number of a diagnostic.
   For example, when this is enabled, Clang will print something like:

   ::

         test.c:28:8: warning: extra tokens at end of #endif directive [-Wextra-tokens]
         #endif bad
                ^
                //

   When this is disabled, Clang will not print the "test.c:28:8: "
   part.

.. _opt_fcaret-diagnostics:

**-f[no-]caret-diagnostics**
   Print source line and ranges from source code in diagnostic.
   This option, which defaults to on, controls whether or not Clang
   prints the source line, source ranges, and caret when emitting a
   diagnostic. For example, when this is enabled, Clang will print
   something like:

   ::

         test.c:28:8: warning: extra tokens at end of #endif directive [-Wextra-tokens]
         #endif bad
                ^
                //

**-f[no-]color-diagnostics**
   This option, which defaults to on when a color-capable terminal is
   detected, controls whether or not Clang prints diagnostics in color.

   When this option is enabled, Clang will use colors to highlight
   specific parts of the diagnostic, e.g.,

   .. nasty hack to not lose our dignity

   .. raw:: html

       <pre>
         <b><span style="color:black">test.c:28:8: <span style="color:magenta">warning</span>: extra tokens at end of #endif directive [-Wextra-tokens]</span></b>
         #endif bad
                <span style="color:green">^</span>
                <span style="color:green">//</span>
       </pre>

   When this is disabled, Clang will just print:

   ::

         test.c:2:8: warning: extra tokens at end of #endif directive [-Wextra-tokens]
         #endif bad
                ^
                //

**-fansi-escape-codes**
   Controls whether ANSI escape codes are used instead of the Windows Console
   API to output colored diagnostics. This option is only used on Windows and
   defaults to off.

.. option:: -fdiagnostics-format=clang/msvc/vi

   Changes diagnostic output format to better match IDEs and command line tools.

   This option controls the output format of the filename, line number,
   and column printed in diagnostic messages. The options, and their
   affect on formatting a simple conversion diagnostic, follow:

   **clang** (default)
       ::

           t.c:3:11: warning: conversion specifies type 'char *' but the argument has type 'int'

   **msvc**
       ::

           t.c(3,11) : warning: conversion specifies type 'char *' but the argument has type 'int'

   **vi**
       ::

           t.c +3:11: warning: conversion specifies type 'char *' but the argument has type 'int'

.. _opt_fdiagnostics-show-option:

**-f[no-]diagnostics-show-option**
   Enable ``[-Woption]`` information in diagnostic line.

   This option, which defaults to on, controls whether or not Clang
   prints the associated :ref:`warning group <cl_diag_warning_groups>`
   option name when outputting a warning diagnostic. For example, in
   this output:

   ::

         test.c:28:8: warning: extra tokens at end of #endif directive [-Wextra-tokens]
         #endif bad
                ^
                //

   Passing **-fno-diagnostics-show-option** will prevent Clang from
   printing the [:ref:`-Wextra-tokens <opt_Wextra-tokens>`] information in
   the diagnostic. This information tells you the flag needed to enable
   or disable the diagnostic, either from the command line or through
   :ref:`#pragma GCC diagnostic <pragma_GCC_diagnostic>`.

.. _opt_fdiagnostics-show-category:

.. option:: -fdiagnostics-show-category=none/id/name

   Enable printing category information in diagnostic line.

   This option, which defaults to "none", controls whether or not Clang
   prints the category associated with a diagnostic when emitting it.
   Each diagnostic may or many not have an associated category, if it
   has one, it is listed in the diagnostic categorization field of the
   diagnostic line (in the []'s).

   For example, a format string warning will produce these three
   renditions based on the setting of this option:

   ::

         t.c:3:11: warning: conversion specifies type 'char *' but the argument has type 'int' [-Wformat]
         t.c:3:11: warning: conversion specifies type 'char *' but the argument has type 'int' [-Wformat,1]
         t.c:3:11: warning: conversion specifies type 'char *' but the argument has type 'int' [-Wformat,Format String]

   This category can be used by clients that want to group diagnostics
   by category, so it should be a high level category. We want dozens
   of these, not hundreds or thousands of them.

.. _opt_fsave-optimization-record:

.. option:: -f[no-]save-optimization-record[=<format>]

   Enable optimization remarks during compilation and write them to a separate
   file.

   This option, which defaults to off, controls whether Clang writes
   optimization reports to a separate file. By recording diagnostics in a file,
   users can parse or sort the remarks in a convenient way.

   By default, the serialization format is YAML.

   The supported serialization formats are:

   -  .. _opt_fsave_optimization_record_yaml:

      ``-fsave-optimization-record=yaml``: A structured YAML format.

   -  .. _opt_fsave_optimization_record_bitstream:

      ``-fsave-optimization-record=bitstream``: A binary format based on LLVM
      Bitstream.

   The output file is controlled by :ref:`-foptimization-record-file <opt_foptimization-record-file>`.

   In the absence of an explicit output file, the file is chosen using the
   following scheme:

   ``<base>.opt.<format>``

   where ``<base>`` is based on the output file of the compilation (whether
   it's explicitly specified through `-o` or not) when used with `-c` or `-S`.
   For example:

   * ``clang -fsave-optimization-record -c in.c -o out.o`` will generate
     ``out.opt.yaml``

   * ``clang -fsave-optimization-record -c in.c `` will generate
     ``in.opt.yaml``

   When targeting (Thin)LTO, the base is derived from the output filename, and
   the extension is not dropped.

   When targeting ThinLTO, the following scheme is used:

   ``<base>.opt.<format>.thin.<num>.<format>``

   Darwin-only: when used for generating a linked binary from a source file
   (through an intermediate object file), the driver will invoke `cc1` to
   generate a temporary object file. The temporary remark file will be emitted
   next to the object file, which will then be picked up by `dsymutil` and
   emitted in the .dSYM bundle. This is available for all formats except YAML.

   For example:

   ``clang -fsave-optimization-record=bitstream in.c -o out`` will generate

   * ``/var/folders/43/9y164hh52tv_2nrdxrj31nyw0000gn/T/a-9be59b.o``

   * ``/var/folders/43/9y164hh52tv_2nrdxrj31nyw0000gn/T/a-9be59b.opt.bitstream``

   * ``out``

   * ``out.dSYM/Contents/Resources/Remarks/out``

   Darwin-only: compiling for multiple architectures will use the following
   scheme:

   ``<base>-<arch>.opt.<format>``

   Note that this is incompatible with passing the
   :ref:`-foptimization-record-file <opt_foptimization-record-file>` option.

.. _opt_foptimization-record-file:

**-foptimization-record-file**
   Control the file to which optimization reports are written. This implies
   :ref:`-fsave-optimization-record <opt_fsave-optimization-record>`.

    On Darwin platforms, this is incompatible with passing multiple
    ``-arch <arch>`` options.

.. _opt_foptimization-record-passes:

**-foptimization-record-passes**
   Only include passes which match a specified regular expression.

   When optimization reports are being output (see
   :ref:`-fsave-optimization-record <opt_fsave-optimization-record>`), this
   option controls the passes that will be included in the final report.

   If this option is not used, all the passes are included in the optimization
   record.

.. _opt_fdiagnostics-show-hotness:

**-f[no-]diagnostics-show-hotness**
   Enable profile hotness information in diagnostic line.

   This option controls whether Clang prints the profile hotness associated
   with diagnostics in the presence of profile-guided optimization information.
   This is currently supported with optimization remarks (see
   :ref:`Options to Emit Optimization Reports <rpass>`). The hotness information
   allows users to focus on the hot optimization remarks that are likely to be
   more relevant for run-time performance.

   For example, in this output, the block containing the callsite of `foo` was
   executed 3000 times according to the profile data:

   ::

         s.c:7:10: remark: foo inlined into bar (hotness: 3000) [-Rpass-analysis=inline]
           sum += foo(x, x - 2);
                  ^

   This option is implied when
   :ref:`-fsave-optimization-record <opt_fsave-optimization-record>` is used.
   Otherwise, it defaults to off.

.. _opt_fdiagnostics-hotness-threshold:

**-fdiagnostics-hotness-threshold**
   Prevent optimization remarks from being output if they do not have at least
   this hotness value.

   This option, which defaults to zero, controls the minimum hotness an
   optimization remark would need in order to be output by Clang. This is
   currently supported with optimization remarks (see :ref:`Options to Emit
   Optimization Reports <rpass>`) when profile hotness information in
   diagnostics is enabled (see
   :ref:`-fdiagnostics-show-hotness <opt_fdiagnostics-show-hotness>`).

.. _opt_fdiagnostics-fixit-info:

**-f[no-]diagnostics-fixit-info**
   Enable "FixIt" information in the diagnostics output.

   This option, which defaults to on, controls whether or not Clang
   prints the information on how to fix a specific diagnostic
   underneath it when it knows. For example, in this output:

   ::

         test.c:28:8: warning: extra tokens at end of #endif directive [-Wextra-tokens]
         #endif bad
                ^
                //

   Passing **-fno-diagnostics-fixit-info** will prevent Clang from
   printing the "//" line at the end of the message. This information
   is useful for users who may not understand what is wrong, but can be
   confusing for machine parsing.

.. _opt_fdiagnostics-print-source-range-info:

**-fdiagnostics-print-source-range-info**
   Print machine parsable information about source ranges.
   This option makes Clang print information about source ranges in a machine
   parsable format after the file/line/column number information. The
   information is a simple sequence of brace enclosed ranges, where each range
   lists the start and end line/column locations. For example, in this output:

   ::

       exprs.c:47:15:{47:8-47:14}{47:17-47:24}: error: invalid operands to binary expression ('int *' and '_Complex float')
          P = (P-42) + Gamma*4;
              ~~~~~~ ^ ~~~~~~~

   The {}'s are generated by -fdiagnostics-print-source-range-info.

   The printed column numbers count bytes from the beginning of the
   line; take care if your source contains multibyte characters.

.. option:: -fdiagnostics-parseable-fixits

   Print Fix-Its in a machine parseable form.

   This option makes Clang print available Fix-Its in a machine
   parseable format at the end of diagnostics. The following example
   illustrates the format:

   ::

        fix-it:"t.cpp":{7:25-7:29}:"Gamma"

   The range printed is a half-open range, so in this example the
   characters at column 25 up to but not including column 29 on line 7
   in t.cpp should be replaced with the string "Gamma". Either the
   range or the replacement string may be empty (representing strict
   insertions and strict erasures, respectively). Both the file name
   and the insertion string escape backslash (as "\\\\"), tabs (as
   "\\t"), newlines (as "\\n"), double quotes(as "\\"") and
   non-printable characters (as octal "\\xxx").

   The printed column numbers count bytes from the beginning of the
   line; take care if your source contains multibyte characters.

.. option:: -fno-elide-type

   Turns off elision in template type printing.

   The default for template type printing is to elide as many template
   arguments as possible, removing those which are the same in both
   template types, leaving only the differences. Adding this flag will
   print all the template arguments. If supported by the terminal,
   highlighting will still appear on differing arguments.

   Default:

   ::

       t.cc:4:5: note: candidate function not viable: no known conversion from 'vector<map<[...], map<float, [...]>>>' to 'vector<map<[...], map<double, [...]>>>' for 1st argument;

   -fno-elide-type:

   ::

       t.cc:4:5: note: candidate function not viable: no known conversion from 'vector<map<int, map<float, int>>>' to 'vector<map<int, map<double, int>>>' for 1st argument;

.. option:: -fdiagnostics-show-template-tree

   Template type diffing prints a text tree.

   For diffing large templated types, this option will cause Clang to
   display the templates as an indented text tree, one argument per
   line, with differences marked inline. This is compatible with
   -fno-elide-type.

   Default:

   ::

       t.cc:4:5: note: candidate function not viable: no known conversion from 'vector<map<[...], map<float, [...]>>>' to 'vector<map<[...], map<double, [...]>>>' for 1st argument;

   With :option:`-fdiagnostics-show-template-tree`:

   ::

       t.cc:4:5: note: candidate function not viable: no known conversion for 1st argument;
         vector<
           map<
             [...],
             map<
               [float != double],
               [...]>>>

.. _cl_diag_warning_groups:

Individual Warning Groups
^^^^^^^^^^^^^^^^^^^^^^^^^

TODO: Generate this from tblgen. Define one anchor per warning group.

.. _opt_wextra-tokens:

.. option:: -Wextra-tokens

   Warn about excess tokens at the end of a preprocessor directive.

   This option, which defaults to on, enables warnings about extra
   tokens at the end of preprocessor directives. For example:

   ::

         test.c:28:8: warning: extra tokens at end of #endif directive [-Wextra-tokens]
         #endif bad
                ^

   These extra tokens are not strictly conforming, and are usually best
   handled by commenting them out.

.. option:: -Wambiguous-member-template

   Warn about unqualified uses of a member template whose name resolves to
   another template at the location of the use.

   This option, which defaults to on, enables a warning in the
   following code:

   ::

       template<typename T> struct set{};
       template<typename T> struct trait { typedef const T& type; };
       struct Value {
         template<typename T> void set(typename trait<T>::type value) {}
       };
       void foo() {
         Value v;
         v.set<double>(3.2);
       }

   C++ [basic.lookup.classref] requires this to be an error, but,
   because it's hard to work around, Clang downgrades it to a warning
   as an extension.

.. option:: -Wbind-to-temporary-copy

   Warn about an unusable copy constructor when binding a reference to a
   temporary.

   This option enables warnings about binding a
   reference to a temporary when the temporary doesn't have a usable
   copy constructor. For example:

   ::

         struct NonCopyable {
           NonCopyable();
         private:
           NonCopyable(const NonCopyable&);
         };
         void foo(const NonCopyable&);
         void bar() {
           foo(NonCopyable());  // Disallowed in C++98; allowed in C++11.
         }

   ::

         struct NonCopyable2 {
           NonCopyable2();
           NonCopyable2(NonCopyable2&);
         };
         void foo(const NonCopyable2&);
         void bar() {
           foo(NonCopyable2());  // Disallowed in C++98; allowed in C++11.
         }

   Note that if ``NonCopyable2::NonCopyable2()`` has a default argument
   whose instantiation produces a compile error, that error will still
   be a hard error in C++98 mode even if this warning is turned off.

Options to Control Clang Crash Diagnostics
------------------------------------------

As unbelievable as it may sound, Clang does crash from time to time.
Generally, this only occurs to those living on the `bleeding
edge <https://llvm.org/releases/download.html#svn>`_. Clang goes to great
lengths to assist you in filing a bug report. Specifically, Clang
generates preprocessed source file(s) and associated run script(s) upon
a crash. These files should be attached to a bug report to ease
reproducibility of the failure. Below are the command line options to
control the crash diagnostics.

.. option:: -fno-crash-diagnostics

  Disable auto-generation of preprocessed source files during a clang crash.

The -fno-crash-diagnostics flag can be helpful for speeding the process
of generating a delta reduced test case.

.. option:: -fcrash-diagnostics-dir=<dir>

  Specify where to write the crash diagnostics files; defaults to the
  usual location for temporary files.

Clang is also capable of generating preprocessed source file(s) and associated
run script(s) even without a crash. This is specially useful when trying to
generate a reproducer for warnings or errors while using modules.

.. option:: -gen-reproducer

  Generates preprocessed source files, a reproducer script and if relevant, a
  cache containing: built module pcm's and all headers needed to rebuild the
  same modules.

.. _rpass:

Options to Emit Optimization Reports
------------------------------------

Optimization reports trace, at a high-level, all the major decisions
done by compiler transformations. For instance, when the inliner
decides to inline function ``foo()`` into ``bar()``, or the loop unroller
decides to unroll a loop N times, or the vectorizer decides to
vectorize a loop body.

Clang offers a family of flags which the optimizers can use to emit
a diagnostic in three cases:

1. When the pass makes a transformation (`-Rpass`).

2. When the pass fails to make a transformation (`-Rpass-missed`).

3. When the pass determines whether or not to make a transformation
   (`-Rpass-analysis`).

NOTE: Although the discussion below focuses on `-Rpass`, the exact
same options apply to `-Rpass-missed` and `-Rpass-analysis`.

Since there are dozens of passes inside the compiler, each of these flags
take a regular expression that identifies the name of the pass which should
emit the associated diagnostic. For example, to get a report from the inliner,
compile the code with:

.. code-block:: console

   $ clang -O2 -Rpass=inline code.cc -o code
   code.cc:4:25: remark: foo inlined into bar [-Rpass=inline]
   int bar(int j) { return foo(j, j - 2); }
                           ^

Note that remarks from the inliner are identified with `[-Rpass=inline]`.
To request a report from every optimization pass, you should use
`-Rpass=.*` (in fact, you can use any valid POSIX regular
expression). However, do not expect a report from every transformation
made by the compiler. Optimization remarks do not really make sense
outside of the major transformations (e.g., inlining, vectorization,
loop optimizations) and not every optimization pass supports this
feature.

Note that when using profile-guided optimization information, profile hotness
information can be included in the remarks (see
:ref:`-fdiagnostics-show-hotness <opt_fdiagnostics-show-hotness>`).

Current limitations
^^^^^^^^^^^^^^^^^^^

1. Optimization remarks that refer to function names will display the
   mangled name of the function. Since these remarks are emitted by the
   back end of the compiler, it does not know anything about the input
   language, nor its mangling rules.

2. Some source locations are not displayed correctly. The front end has
   a more detailed source location tracking than the locations included
   in the debug info (e.g., the front end can locate code inside macro
   expansions). However, the locations used by `-Rpass` are
   translated from debug annotations. That translation can be lossy,
   which results in some remarks having no location information.

Options to Emit Resource Consumption Reports
--------------------------------------------

These are options that report execution time and consumed memory of different
compilations steps.

.. option:: -fproc-stat-report=

  This option requests driver to print used memory and execution time of each
  compilation step. The ``clang`` driver during execution calls different tools,
  like compiler, assembler, linker etc. With this option the driver reports
  total execution time, the execution time spent in user mode and peak memory
  usage of each the called tool. Value of the option specifies where the report
  is sent to. If it specifies a regular file, the data are saved to this file in
  CSV format:

  .. code-block:: console

    $ clang -fproc-stat-report=abc foo.c
    $ cat abc
    clang-11,"/tmp/foo-123456.o",92000,84000,87536
    ld,"a.out",900,8000,53568

  The data on each row represent:
  
  * file name of the tool executable,
  * output file name in quotes,
  * total execution time in microseconds,
  * execution time in user mode in microseconds,
  * peak memory usage in Kb.
  
  It is possible to specify this option without any value. In this case statistics
  are printed on standard output in human readable format:
  
  .. code-block:: console

    $ clang -fproc-stat-report foo.c
    clang-11: output=/tmp/foo-855a8e.o, total=68.000 ms, user=60.000 ms, mem=86920 Kb
    ld: output=a.out, total=8.000 ms, user=4.000 ms, mem=52320 Kb
  
  The report file specified in the option is locked for write, so this option
  can be used to collect statistics in parallel builds. The report file is not
  cleared, new data is appended to it, thus making posible to accumulate build
  statistics.

  You can also use environment variables to control the process statistics reporting.
  Setting ``CC_PRINT_PROC_STAT`` to ``1`` enables the feature, the report goes to
  stdout in human readable format.
  Setting ``CC_PRINT_PROC_STAT_FILE`` to a fully qualified file path makes it report
  process statistics to the given file in the CSV format. Specifying a relative
  path will likely lead to multiple files with the same name created in different
  directories, since the path is relative to a changing working directory.

  These environment variables are handy when you need to request the statistics
  report without changing your build scripts or alter the existing set of compiler
  options. Note that ``-fproc-stat-report`` take precedence over ``CC_PRINT_PROC_STAT``
  and ``CC_PRINT_PROC_STAT_FILE``.

  .. code-block:: console

    $ export CC_PRINT_PROC_STAT=1
    $ export CC_PRINT_PROC_STAT_FILE=~/project-build-proc-stat.csv
    $ make

Other Options
-------------
Clang options that don't fit neatly into other categories.

.. option:: -fgnuc-version=

  This flag controls the value of ``__GNUC__`` and related macros. This flag
  does not enable or disable any GCC extensions implemented in Clang. Setting
  the version to zero causes Clang to leave ``__GNUC__`` and other
  GNU-namespaced macros, such as ``__GXX_WEAK__``, undefined.

.. option:: -MV

  When emitting a dependency file, use formatting conventions appropriate
  for NMake or Jom. Ignored unless another option causes Clang to emit a
  dependency file.

When Clang emits a dependency file (e.g., you supplied the -M option)
most filenames can be written to the file without any special formatting.
Different Make tools will treat different sets of characters as "special"
and use different conventions for telling the Make tool that the character
is actually part of the filename. Normally Clang uses backslash to "escape"
a special character, which is the convention used by GNU Make. The -MV
option tells Clang to put double-quotes around the entire filename, which
is the convention used by NMake and Jom.

Configuration files
-------------------

Configuration files group command-line options and allow all of them to be
specified just by referencing the configuration file. They may be used, for
example, to collect options required to tune compilation for particular
target, such as -L, -I, -l, --sysroot, codegen options, etc.

The command line option `--config` can be used to specify configuration
file in a Clang invocation. For example:

::

    clang --config /home/user/cfgs/testing.txt
    clang --config debug.cfg

If the provided argument contains a directory separator, it is considered as
a file path, and options are read from that file. Otherwise the argument is
treated as a file name and is searched for sequentially in the directories:

    - user directory,
    - system directory,
    - the directory where Clang executable resides.

Both user and system directories for configuration files are specified during
clang build using CMake parameters, CLANG_CONFIG_FILE_USER_DIR and
CLANG_CONFIG_FILE_SYSTEM_DIR respectively. The first file found is used. It is
an error if the required file cannot be found.

Another way to specify a configuration file is to encode it in executable name.
For example, if the Clang executable is named `armv7l-clang` (it may be a
symbolic link to `clang`), then Clang will search for file `armv7l.cfg` in the
directory where Clang resides.

If a driver mode is specified in invocation, Clang tries to find a file specific
for the specified mode. For example, if the executable file is named
`x86_64-clang-cl`, Clang first looks for `x86_64-cl.cfg` and if it is not found,
looks for `x86_64.cfg`.

If the command line contains options that effectively change target architecture
(these are -m32, -EL, and some others) and the configuration file starts with an
architecture name, Clang tries to load the configuration file for the effective
architecture. For example, invocation:

::

    x86_64-clang -m32 abc.c

causes Clang search for a file `i368.cfg` first, and if no such file is found,
Clang looks for the file `x86_64.cfg`.

The configuration file consists of command-line options specified on one or
more lines. Lines composed of whitespace characters only are ignored as well as
lines in which the first non-blank character is `#`. Long options may be split
between several lines by a trailing backslash. Here is example of a
configuration file:

::

    # Several options on line
    -c --target=x86_64-unknown-linux-gnu

    # Long option split between lines
    -I/usr/lib/gcc/x86_64-linux-gnu/5.4.0/../../../../\
    include/c++/5.4.0

    # other config files may be included
    @linux.options

Files included by `@file` directives in configuration files are resolved
relative to the including file. For example, if a configuration file
`~/.llvm/target.cfg` contains the directive `@os/linux.opts`, the file
`linux.opts` is searched for in the directory `~/.llvm/os`.

Language and Target-Independent Features
========================================

Controlling Errors and Warnings
-------------------------------

Clang provides a number of ways to control which code constructs cause
it to emit errors and warning messages, and how they are displayed to
the console.

Controlling How Clang Displays Diagnostics
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When Clang emits a diagnostic, it includes rich information in the
output, and gives you fine-grain control over which information is
printed. Clang has the ability to print this information, and these are
the options that control it:

#. A file/line/column indicator that shows exactly where the diagnostic
   occurs in your code [:ref:`-fshow-column <opt_fshow-column>`,
   :ref:`-fshow-source-location <opt_fshow-source-location>`].
#. A categorization of the diagnostic as a note, warning, error, or
   fatal error.
#. A text string that describes what the problem is.
#. An option that indicates how to control the diagnostic (for
   diagnostics that support it)
   [:ref:`-fdiagnostics-show-option <opt_fdiagnostics-show-option>`].
#. A :ref:`high-level category <diagnostics_categories>` for the diagnostic
   for clients that want to group diagnostics by class (for diagnostics
   that support it)
   [:ref:`-fdiagnostics-show-category <opt_fdiagnostics-show-category>`].
#. The line of source code that the issue occurs on, along with a caret
   and ranges that indicate the important locations
   [:ref:`-fcaret-diagnostics <opt_fcaret-diagnostics>`].
#. "FixIt" information, which is a concise explanation of how to fix the
   problem (when Clang is certain it knows)
   [:ref:`-fdiagnostics-fixit-info <opt_fdiagnostics-fixit-info>`].
#. A machine-parsable representation of the ranges involved (off by
   default)
   [:ref:`-fdiagnostics-print-source-range-info <opt_fdiagnostics-print-source-range-info>`].

For more information please see :ref:`Formatting of
Diagnostics <cl_diag_formatting>`.

Diagnostic Mappings
^^^^^^^^^^^^^^^^^^^

All diagnostics are mapped into one of these 6 classes:

-  Ignored
-  Note
-  Remark
-  Warning
-  Error
-  Fatal

.. _diagnostics_categories:

Diagnostic Categories
^^^^^^^^^^^^^^^^^^^^^

Though not shown by default, diagnostics may each be associated with a
high-level category. This category is intended to make it possible to
triage builds that produce a large number of errors or warnings in a
grouped way.

Categories are not shown by default, but they can be turned on with the
:ref:`-fdiagnostics-show-category <opt_fdiagnostics-show-category>` option.
When set to "``name``", the category is printed textually in the
diagnostic output. When it is set to "``id``", a category number is
printed. The mapping of category names to category id's can be obtained
by running '``clang   --print-diagnostic-categories``'.

Controlling Diagnostics via Command Line Flags
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

TODO: -W flags, -pedantic, etc

.. _pragma_gcc_diagnostic:

Controlling Diagnostics via Pragmas
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Clang can also control what diagnostics are enabled through the use of
pragmas in the source code. This is useful for turning off specific
warnings in a section of source code. Clang supports GCC's pragma for
compatibility with existing source code, as well as several extensions.

The pragma may control any warning that can be used from the command
line. Warnings may be set to ignored, warning, error, or fatal. The
following example code will tell Clang or GCC to ignore the -Wall
warnings:

.. code-block:: c

  #pragma GCC diagnostic ignored "-Wall"

In addition to all of the functionality provided by GCC's pragma, Clang
also allows you to push and pop the current warning state. This is
particularly useful when writing a header file that will be compiled by
other people, because you don't know what warning flags they build with.

In the below example :option:`-Wextra-tokens` is ignored for only a single line
of code, after which the diagnostics return to whatever state had previously
existed.

.. code-block:: c

  #if foo
  #endif foo // warning: extra tokens at end of #endif directive

  #pragma clang diagnostic push
  #pragma clang diagnostic ignored "-Wextra-tokens"

  #if foo
  #endif foo // no warning

  #pragma clang diagnostic pop

The push and pop pragmas will save and restore the full diagnostic state
of the compiler, regardless of how it was set. That means that it is
possible to use push and pop around GCC compatible diagnostics and Clang
will push and pop them appropriately, while GCC will ignore the pushes
and pops as unknown pragmas. It should be noted that while Clang
supports the GCC pragma, Clang and GCC do not support the exact same set
of warnings, so even when using GCC compatible #pragmas there is no
guarantee that they will have identical behaviour on both compilers.

In addition to controlling warnings and errors generated by the compiler, it is
possible to generate custom warning and error messages through the following
pragmas:

.. code-block:: c

  // The following will produce warning messages
  #pragma message "some diagnostic message"
  #pragma GCC warning "TODO: replace deprecated feature"

  // The following will produce an error message
  #pragma GCC error "Not supported"

These pragmas operate similarly to the ``#warning`` and ``#error`` preprocessor
directives, except that they may also be embedded into preprocessor macros via
the C99 ``_Pragma`` operator, for example:

.. code-block:: c

  #define STR(X) #X
  #define DEFER(M,...) M(__VA_ARGS__)
  #define CUSTOM_ERROR(X) _Pragma(STR(GCC error(X " at line " DEFER(STR,__LINE__))))

  CUSTOM_ERROR("Feature not available");

Controlling Diagnostics in System Headers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Warnings are suppressed when they occur in system headers. By default,
an included file is treated as a system header if it is found in an
include path specified by ``-isystem``, but this can be overridden in
several ways.

The ``system_header`` pragma can be used to mark the current file as
being a system header. No warnings will be produced from the location of
the pragma onwards within the same file.

.. code-block:: c

  #if foo
  #endif foo // warning: extra tokens at end of #endif directive

  #pragma clang system_header

  #if foo
  #endif foo // no warning

The `--system-header-prefix=` and `--no-system-header-prefix=`
command-line arguments can be used to override whether subsets of an include
path are treated as system headers. When the name in a ``#include`` directive
is found within a header search path and starts with a system prefix, the
header is treated as a system header. The last prefix on the
command-line which matches the specified header name takes precedence.
For instance:

.. code-block:: console

  $ clang -Ifoo -isystem bar --system-header-prefix=x/ \
      --no-system-header-prefix=x/y/

Here, ``#include "x/a.h"`` is treated as including a system header, even
if the header is found in ``foo``, and ``#include "x/y/b.h"`` is treated
as not including a system header, even if the header is found in
``bar``.

A ``#include`` directive which finds a file relative to the current
directory is treated as including a system header if the including file
is treated as a system header.

.. _diagnostics_enable_everything:

Enabling All Diagnostics
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In addition to the traditional ``-W`` flags, one can enable **all** diagnostics
by passing :option:`-Weverything`. This works as expected with
:option:`-Werror`, and also includes the warnings from :option:`-pedantic`. Some
diagnostics contradict each other, therefore, users of :option:`-Weverything`
often disable many diagnostics such as `-Wno-c++98-compat` and `-Wno-c++-compat`
because they contradict recent C++ standards.

Since :option:`-Weverything` enables every diagnostic, we generally don't
recommend using it. `-Wall` `-Wextra` are a better choice for most projects.
Using :option:`-Weverything` means that updating your compiler is more difficult
because you're exposed to experimental diagnostics which might be of lower
quality than the default ones. If you do use :option:`-Weverything` then we
advise that you address all new compiler diagnostics as they get added to Clang,
either by fixing everything they find or explicitly disabling that diagnostic
with its corresponding `Wno-` option.

Note that when combined with :option:`-w` (which disables all warnings),
disabling all warnings wins.

Controlling Static Analyzer Diagnostics
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

While not strictly part of the compiler, the diagnostics from Clang's
`static analyzer <https://clang-analyzer.llvm.org>`_ can also be
influenced by the user via changes to the source code. See the available
`annotations <https://clang-analyzer.llvm.org/annotations.html>`_ and the
analyzer's `FAQ
page <https://clang-analyzer.llvm.org/faq.html#exclude_code>`_ for more
information.

.. _usersmanual-precompiled-headers:

Precompiled Headers
-------------------

`Precompiled headers <https://en.wikipedia.org/wiki/Precompiled_header>`_
are a general approach employed by many compilers to reduce compilation
time. The underlying motivation of the approach is that it is common for
the same (and often large) header files to be included by multiple
source files. Consequently, compile times can often be greatly improved
by caching some of the (redundant) work done by a compiler to process
headers. Precompiled header files, which represent one of many ways to
implement this optimization, are literally files that represent an
on-disk cache that contains the vital information necessary to reduce
some of the work needed to process a corresponding header file. While
details of precompiled headers vary between compilers, precompiled
headers have been shown to be highly effective at speeding up program
compilation on systems with very large system headers (e.g., macOS).

Generating a PCH File
^^^^^^^^^^^^^^^^^^^^^

To generate a PCH file using Clang, one invokes Clang with the
`-x <language>-header` option. This mirrors the interface in GCC
for generating PCH files:

.. code-block:: console

  $ gcc -x c-header test.h -o test.h.gch
  $ clang -x c-header test.h -o test.h.pch

Using a PCH File
^^^^^^^^^^^^^^^^

A PCH file can then be used as a prefix header when a :option:`-include`
option is passed to ``clang``:

.. code-block:: console

  $ clang -include test.h test.c -o test

The ``clang`` driver will first check if a PCH file for ``test.h`` is
available; if so, the contents of ``test.h`` (and the files it includes)
will be processed from the PCH file. Otherwise, Clang falls back to
directly processing the content of ``test.h``. This mirrors the behavior
of GCC.

.. note::

  Clang does *not* automatically use PCH files for headers that are directly
  included within a source file. For example:

  .. code-block:: console

    $ clang -x c-header test.h -o test.h.pch
    $ cat test.c
    #include "test.h"
    $ clang test.c -o test

  In this example, ``clang`` will not automatically use the PCH file for
  ``test.h`` since ``test.h`` was included directly in the source file and not
  specified on the command line using :option:`-include`.

Relocatable PCH Files
^^^^^^^^^^^^^^^^^^^^^

It is sometimes necessary to build a precompiled header from headers
that are not yet in their final, installed locations. For example, one
might build a precompiled header within the build tree that is then
meant to be installed alongside the headers. Clang permits the creation
of "relocatable" precompiled headers, which are built with a given path
(into the build directory) and can later be used from an installed
location.

To build a relocatable precompiled header, place your headers into a
subdirectory whose structure mimics the installed location. For example,
if you want to build a precompiled header for the header ``mylib.h``
that will be installed into ``/usr/include``, create a subdirectory
``build/usr/include`` and place the header ``mylib.h`` into that
subdirectory. If ``mylib.h`` depends on other headers, then they can be
stored within ``build/usr/include`` in a way that mimics the installed
location.

Building a relocatable precompiled header requires two additional
arguments. First, pass the ``--relocatable-pch`` flag to indicate that
the resulting PCH file should be relocatable. Second, pass
``-isysroot /path/to/build``, which makes all includes for your library
relative to the build directory. For example:

.. code-block:: console

  # clang -x c-header --relocatable-pch -isysroot /path/to/build /path/to/build/mylib.h mylib.h.pch

When loading the relocatable PCH file, the various headers used in the
PCH file are found from the system header root. For example, ``mylib.h``
can be found in ``/usr/include/mylib.h``. If the headers are installed
in some other system root, the ``-isysroot`` option can be used provide
a different system root from which the headers will be based. For
example, ``-isysroot /Developer/SDKs/MacOSX10.4u.sdk`` will look for
``mylib.h`` in ``/Developer/SDKs/MacOSX10.4u.sdk/usr/include/mylib.h``.

Relocatable precompiled headers are intended to be used in a limited
number of cases where the compilation environment is tightly controlled
and the precompiled header cannot be generated after headers have been
installed.

.. _controlling-fp-behavior:

Controlling Floating Point Behavior
-----------------------------------

Clang provides a number of ways to control floating point behavior. The options
are listed below.

.. option:: -ffast-math

   Enable fast-math mode.  This option lets the
   compiler make aggressive, potentially-lossy assumptions about
   floating-point math.  These include:

   * Floating-point math obeys regular algebraic rules for real numbers (e.g.
     ``+`` and ``*`` are associative, ``x/y == x * (1/y)``, and
     ``(a + b) * c == a * c + b * c``),
   * Operands to floating-point operations are not equal to ``NaN`` and
     ``Inf``, and
   * ``+0`` and ``-0`` are interchangeable.

   ``-ffast-math`` also defines the ``__FAST_MATH__`` preprocessor
   macro. Some math libraries recognize this macro and change their behavior.
   With the exception of ``-ffp-contract=fast``, using any of the options
   below to disable any of the individual optimizations in ``-ffast-math``
   will cause ``__FAST_MATH__`` to no longer be set.

  This option implies:

   * ``-fno-honor-infinities``

   * ``-fno-honor-nans``

   * ``-fno-math-errno``

   * ``-ffinite-math-only``

   * ``-fassociative-math``

   * ``-freciprocal-math``

   * ``-fno-signed-zeros``

   * ``-fno-trapping-math``

   * ``-ffp-contract=fast``

.. option:: -fdenormal-fp-math=<value>

   Select which denormal numbers the code is permitted to require.

   Valid values are: 

   * ``ieee`` - IEEE 754 denormal numbers
   * ``preserve-sign`` - the sign of a flushed-to-zero number is preserved in the sign of 0
   * ``positive-zero`` - denormals are flushed to positive zero

   Defaults to ``ieee``.

.. _opt_fstrict-float-cast-overflow:

**-f[no-]strict-float-cast-overflow**

   When a floating-point value is not representable in a destination integer 
   type, the code has undefined behavior according to the language standard.
   By default, Clang will not guarantee any particular result in that case.
   With the 'no-strict' option, Clang attempts to match the overflowing behavior
   of the target's native float-to-int conversion instructions.

.. _opt_fmath-errno:

**-f[no-]math-errno**

   Require math functions to indicate errors by setting errno.
   The default varies by ToolChain.  ``-fno-math-errno`` allows optimizations
   that might cause standard C math functions to not set ``errno``.
   For example, on some systems, the math function ``sqrt`` is specified
   as setting ``errno`` to ``EDOM`` when the input is negative. On these
   systems, the compiler cannot normally optimize a call to ``sqrt`` to use
   inline code (e.g. the x86 ``sqrtsd`` instruction) without additional
   checking to ensure that ``errno`` is set appropriately.
   ``-fno-math-errno`` permits these transformations.

   On some targets, math library functions never set ``errno``, and so
   ``-fno-math-errno`` is the default. This includes most BSD-derived
   systems, including Darwin.

.. _opt_ftrapping-math:

**-f[no-]trapping-math**

   Control floating point exception behavior. ``-fno-trapping-math`` allows optimizations that assume that floating point operations cannot generate traps such as divide-by-zero, overflow and underflow.

- The option ``-ftrapping-math`` behaves identically to ``-ffp-exception-behavior=strict``.
- The option ``-fno-trapping-math`` behaves identically to ``-ffp-exception-behavior=ignore``.   This is the default.

.. option:: -ffp-contract=<value>

   Specify when the compiler is permitted to form fused floating-point
   operations, such as fused multiply-add (FMA). Fused operations are
   permitted to produce more precise results than performing the same
   operations separately.

   The C standard permits intermediate floating-point results within an
   expression to be computed with more precision than their type would
   normally allow. This permits operation fusing, and Clang takes advantage
   of this by default. This behavior can be controlled with the ``FP_CONTRACT``
   and ``clang fp contract`` pragmas. Please refer to the pragma documentation
   for a description of how the pragmas interact with this option.

   Valid values are:

   * ``fast`` (fuse across statements disregarding pragmas, default for CUDA)
   * ``on`` (fuse in the same statement unless dictated by pragmas, default for languages other than CUDA/HIP)
   * ``off`` (never fuse)
   * ``fast-honor-pragmas`` (fuse across statements unless dictated by pragmas, default for HIP)

.. _opt_fhonor-infinities:

**-f[no-]honor-infinities**

   If both ``-fno-honor-infinities`` and ``-fno-honor-nans`` are used,
   has the same effect as specifying ``-ffinite-math-only``.

.. _opt_fhonor-nans:

**-f[no-]honor-nans**

   If both ``-fno-honor-infinities`` and ``-fno-honor-nans`` are used,
   has the same effect as specifying ``-ffinite-math-only``.

.. _opt_fsigned-zeros:

**-f[no-]signed-zeros**

   Allow optimizations that ignore the sign of floating point zeros.
   Defaults to ``-fno-signed-zeros``.

.. _opt_fassociative-math:

**-f[no-]associative-math**

  Allow floating point operations to be reassociated.
  Defaults to ``-fno-associative-math``.

.. _opt_freciprocal-math:

**-f[no-]reciprocal-math**

  Allow division operations to be transformed into multiplication by a
  reciprocal. This can be significantly faster than an ordinary division
  but can also have significantly less precision. Defaults to
  ``-fno-reciprocal-math``.

.. _opt_funsafe-math-optimizations:

**-f[no-]unsafe-math-optimizations**

   Allow unsafe floating-point optimizations. Also implies:

   * ``-fassociative-math``
   * ``-freciprocal-math``
   * ``-fno-signed-zeroes``
   * ``-fno-trapping-math``.

   Defaults to ``-fno-unsafe-math-optimizations``.

.. _opt_ffinite-math-only:

**-f[no-]finite-math-only**

   Allow floating-point optimizations that assume arguments and results are
   not NaNs or +-Inf.  This defines the ``__FINITE_MATH_ONLY__`` preprocessor macro.
   Also implies:

   * ``-fno-honor-infinities``
   * ``-fno-honor-nans``

   Defaults to ``-fno-finite-math-only``.

.. _opt_frounding-math:

**-f[no-]rounding-math**

Force floating-point operations to honor the dynamically-set rounding mode by default.

The result of a floating-point operation often cannot be exactly represented in the result type and therefore must be rounded.  IEEE 754 describes different rounding modes that control how to perform this rounding, not all of which are supported by all implementations.  C provides interfaces (``fesetround`` and ``fesetenv``) for dynamically controlling the rounding mode, and while it also recommends certain conventions for changing the rounding mode, these conventions are not typically enforced in the ABI.  Since the rounding mode changes the numerical result of operations, the compiler must understand something about it in order to optimize floating point operations.

Note that floating-point operations performed as part of constant initialization are formally performed prior to the start of the program and are therefore not subject to the current rounding mode.  This includes the initialization of global variables and local ``static`` variables.  Floating-point operations in these contexts will be rounded using ``FE_TONEAREST``.

- The option ``-fno-rounding-math`` allows the compiler to assume that the rounding mode is set to ``FE_TONEAREST``.  This is the default.
- The option ``-frounding-math`` forces the compiler to honor the dynamically-set rounding mode.  This prevents optimizations which might affect results if the rounding mode changes or is different from the default; for example, it prevents floating-point operations from being reordered across most calls and prevents constant-folding when the result is not exactly representable.

.. option:: -ffp-model=<value>

   Specify floating point behavior. ``-ffp-model`` is an umbrella
   option that encompasses functionality provided by other, single
   purpose, floating point options.  Valid values are: ``precise``, ``strict``,
   and ``fast``.
   Details:

   * ``precise`` Disables optimizations that are not value-safe on floating-point data, although FP contraction (FMA) is enabled (``-ffp-contract=fast``).  This is the default behavior.
   * ``strict`` Enables ``-frounding-math`` and ``-ffp-exception-behavior=strict``, and disables contractions (FMA).  All of the ``-ffast-math`` enablements are disabled. Enables ``STDC FENV_ACCESS``: by default ``FENV_ACCESS`` is disabled. This option setting behaves as though ``#pragma STDC FENV_ACESS ON`` appeared at the top of the source file.
   * ``fast`` Behaves identically to specifying both ``-ffast-math`` and ``ffp-contract=fast``

   Note: If your command line specifies multiple instances
   of the ``-ffp-model`` option, or if your command line option specifies
   ``-ffp-model`` and later on the command line selects a floating point
   option that has the effect of negating part of the  ``ffp-model`` that
   has been selected, then the compiler will issue a diagnostic warning
   that the override has occurred.

.. option:: -ffp-exception-behavior=<value>

   Specify the floating-point exception behavior.

   Valid values are: ``ignore``, ``maytrap``, and ``strict``.
   The default value is ``ignore``.  Details:

   * ``ignore`` The compiler assumes that the exception status flags will not be read and that floating point exceptions will be masked.
   * ``maytrap`` The compiler avoids transformations that may raise exceptions that would not have been raised by the original code. Constant folding performed by the compiler is exempt from this option.
   * ``strict`` The compiler ensures that all transformations strictly preserve the floating point exception semantics of the original code.

.. option:: -f[no-]protect-parens:

   This option pertains to floating-point types, complex types with
   floating-point components, and vectors of these types. Some arithmetic
   expression transformations that are mathematically correct and permissible
   according to the C and C++ language standards may be incorrect when dealing
   with floating-point types, such as reassociation and distribution. Further,
   the optimizer may ignore parentheses when computing arithmetic expressions
   in circumstances where the parenthesized and unparenthesized expression
   express the same mathematical value. For example (a+b)+c is the same
   mathematical value as a+(b+c), but the optimizer is free to evaluate the 
   additions in any order regardless of the parentheses. When enabled, this
   option forces the optimizer to honor the order of operations with respect
   to parentheses in all circumstances.

   Note that floating-point contraction (option `-ffp-contract=`) is disabled
   when `-fprotect-parens` is enabled.  Also note that in safe floating-point
   modes, such as `-ffp-model=precise` or `-ffp-model=strict`, this option
   has no effect because the optimizer is prohibited from making unsafe
   transformations.

.. _fp-constant-eval:

A note about Floating Point Constant Evaluation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In C, the only place floating point operations are guaranteed to be evaluated
during translation is in the initializers of variables of static storage
duration, which are all notionally initialized before the program begins
executing (and thus before a non-default floating point environment can be
entered).  But C++ has many more contexts where floating point constant
evaluation occurs.  Specifically: for static/thread-local variables,
first try evaluating the initializer in a constant context, including in the
constant floating point environment (just like in C), and then, if that fails,
fall back to emitting runtime code to perform the initialization (which might
in general be in a different floating point environment).

Consider this example when compiled with ``-frounding-math``

   .. code-block:: console

     constexpr float func_01(float x, float y) {
       return x + y;
     }
     float V1 = func_01(1.0F, 0x0.000001p0F);

The C++ rule is that initializers for static storage duration variables are
first evaluated during translation (therefore, in the default rounding mode),
and only evaluated at runtime (and therefore in the runtime rounding mode) if
the compile-time evaluation fails. This is in line with the C rules;
C11 F.8.5 says: *All computation for automatic initialization is done (as if)
at execution time; thus, it is affected by any operative modes and raises
floating-point exceptions as required by IEC 60559 (provided the state for the
FENV_ACCESS pragma is ‘‘on’’). All computation for initialization of objects
that have static or thread storage duration is done (as if) at translation
time.* C++ generalizes this by adding another phase of initialization
(at runtime) if the translation-time initialization fails, but the
translation-time evaluation of the initializer of succeeds, it will be
treated as a constant initializer.


.. _controlling-code-generation:

Controlling Code Generation
---------------------------

Clang provides a number of ways to control code generation. The options
are listed below.

**-f[no-]sanitize=check1,check2,...**
   Turn on runtime checks for various forms of undefined or suspicious
   behavior.

   This option controls whether Clang adds runtime checks for various
   forms of undefined or suspicious behavior, and is disabled by
   default. If a check fails, a diagnostic message is produced at
   runtime explaining the problem. The main checks are:

   -  .. _opt_fsanitize_address:

      ``-fsanitize=address``:
      :doc:`AddressSanitizer`, a memory error
      detector.
   -  .. _opt_fsanitize_thread:

      ``-fsanitize=thread``: :doc:`ThreadSanitizer`, a data race detector.
   -  .. _opt_fsanitize_memory:

      ``-fsanitize=memory``: :doc:`MemorySanitizer`,
      a detector of uninitialized reads. Requires instrumentation of all
      program code.
   -  .. _opt_fsanitize_undefined:

      ``-fsanitize=undefined``: :doc:`UndefinedBehaviorSanitizer`,
      a fast and compatible undefined behavior checker.

   -  ``-fsanitize=dataflow``: :doc:`DataFlowSanitizer`, a general data
      flow analysis.
   -  ``-fsanitize=cfi``: :doc:`control flow integrity <ControlFlowIntegrity>`
      checks. Requires ``-flto``.
   -  ``-fsanitize=safe-stack``: :doc:`safe stack <SafeStack>`
      protection against stack-based memory corruption errors.

   There are more fine-grained checks available: see
   the :ref:`list <ubsan-checks>` of specific kinds of
   undefined behavior that can be detected and the :ref:`list <cfi-schemes>`
   of control flow integrity schemes.

   The ``-fsanitize=`` argument must also be provided when linking, in
   order to link to the appropriate runtime library.

   It is not possible to combine more than one of the ``-fsanitize=address``,
   ``-fsanitize=thread``, and ``-fsanitize=memory`` checkers in the same
   program.

**-f[no-]sanitize-recover=check1,check2,...**

**-f[no-]sanitize-recover[=all]**

   Controls which checks enabled by ``-fsanitize=`` flag are non-fatal.
   If the check is fatal, program will halt after the first error
   of this kind is detected and error report is printed.

   By default, non-fatal checks are those enabled by
   :doc:`UndefinedBehaviorSanitizer`,
   except for ``-fsanitize=return`` and ``-fsanitize=unreachable``. Some
   sanitizers may not support recovery (or not support it by default
   e.g. :doc:`AddressSanitizer`), and always crash the program after the issue
   is detected.

   Note that the ``-fsanitize-trap`` flag has precedence over this flag.
   This means that if a check has been configured to trap elsewhere on the
   command line, or if the check traps by default, this flag will not have
   any effect unless that sanitizer's trapping behavior is disabled with
   ``-fno-sanitize-trap``.

   For example, if a command line contains the flags ``-fsanitize=undefined
   -fsanitize-trap=undefined``, the flag ``-fsanitize-recover=alignment``
   will have no effect on its own; it will need to be accompanied by
   ``-fno-sanitize-trap=alignment``.

**-f[no-]sanitize-trap=check1,check2,...**

**-f[no-]sanitize-trap[=all]**

   Controls which checks enabled by the ``-fsanitize=`` flag trap. This
   option is intended for use in cases where the sanitizer runtime cannot
   be used (for instance, when building libc or a kernel module), or where
   the binary size increase caused by the sanitizer runtime is a concern.

   This flag is only compatible with :doc:`control flow integrity
   <ControlFlowIntegrity>` schemes and :doc:`UndefinedBehaviorSanitizer`
   checks other than ``vptr``.

   This flag is enabled by default for sanitizers in the ``cfi`` group.

.. option:: -fsanitize-ignorelist=/path/to/ignorelist/file

   Disable or modify sanitizer checks for objects (source files, functions,
   variables, types) listed in the file. See
   :doc:`SanitizerSpecialCaseList` for file format description.

.. option:: -fno-sanitize-ignorelist

   Don't use ignorelist file, if it was specified earlier in the command line.

**-f[no-]sanitize-coverage=[type,features,...]**

   Enable simple code coverage in addition to certain sanitizers.
   See :doc:`SanitizerCoverage` for more details.

**-f[no-]sanitize-address-outline-instrumentation**

   Controls how address sanitizer code is generated. If enabled will always use
   a function call instead of inlining the code. Turning this option on could
   reduce the binary size, but might result in a worse run-time performance.

   See :doc: `AddressSanitizer` for more details.

**-f[no-]sanitize-stats**

   Enable simple statistics gathering for the enabled sanitizers.
   See :doc:`SanitizerStats` for more details.

.. option:: -fsanitize-undefined-trap-on-error

   Deprecated alias for ``-fsanitize-trap=undefined``.

.. option:: -fsanitize-cfi-cross-dso

   Enable cross-DSO control flow integrity checks. This flag modifies
   the behavior of sanitizers in the ``cfi`` group to allow checking
   of cross-DSO virtual and indirect calls.

.. option:: -fsanitize-cfi-icall-generalize-pointers

   Generalize pointers in return and argument types in function type signatures
   checked by Control Flow Integrity indirect call checking. See
   :doc:`ControlFlowIntegrity` for more details.

.. option:: -fstrict-vtable-pointers

   Enable optimizations based on the strict rules for overwriting polymorphic
   C++ objects, i.e. the vptr is invariant during an object's lifetime.
   This enables better devirtualization. Turned off by default, because it is
   still experimental.

.. option:: -fwhole-program-vtables

   Enable whole-program vtable optimizations, such as single-implementation
   devirtualization and virtual constant propagation, for classes with
   :doc:`hidden LTO visibility <LTOVisibility>`. Requires ``-flto``.

.. option:: -fforce-emit-vtables

   In order to improve devirtualization, forces emitting of vtables even in
   modules where it isn't necessary. It causes more inline virtual functions
   to be emitted.

.. option:: -fno-assume-sane-operator-new

   Don't assume that the C++'s new operator is sane.

   This option tells the compiler to do not assume that C++'s global
   new operator will always return a pointer that does not alias any
   other pointer when the function returns.

.. option:: -ftrap-function=[name]

   Instruct code generator to emit a function call to the specified
   function name for ``__builtin_trap()``.

   LLVM code generator translates ``__builtin_trap()`` to a trap
   instruction if it is supported by the target ISA. Otherwise, the
   builtin is translated into a call to ``abort``. If this option is
   set, then the code generator will always lower the builtin to a call
   to the specified function regardless of whether the target ISA has a
   trap instruction. This option is useful for environments (e.g.
   deeply embedded) where a trap cannot be properly handled, or when
   some custom behavior is desired.

.. option:: -ftls-model=[model]

   Select which TLS model to use.

   Valid values are: ``global-dynamic``, ``local-dynamic``,
   ``initial-exec`` and ``local-exec``. The default value is
   ``global-dynamic``. The compiler may use a different model if the
   selected model is not supported by the target, or if a more
   efficient model can be used. The TLS model can be overridden per
   variable using the ``tls_model`` attribute.

.. option:: -femulated-tls

   Select emulated TLS model, which overrides all -ftls-model choices.

   In emulated TLS mode, all access to TLS variables are converted to
   calls to __emutls_get_address in the runtime library.

.. option:: -mhwdiv=[values]

   Select the ARM modes (arm or thumb) that support hardware division
   instructions.

   Valid values are: ``arm``, ``thumb`` and ``arm,thumb``.
   This option is used to indicate which mode (arm or thumb) supports
   hardware division instructions. This only applies to the ARM
   architecture.

.. option:: -m[no-]crc

   Enable or disable CRC instructions.

   This option is used to indicate whether CRC instructions are to
   be generated. This only applies to the ARM architecture.

   CRC instructions are enabled by default on ARMv8.

.. option:: -mgeneral-regs-only

   Generate code which only uses the general purpose registers.

   This option restricts the generated code to use general registers
   only. This only applies to the AArch64 architecture.

.. option:: -mcompact-branches=[values]

   Control the usage of compact branches for MIPSR6.

   Valid values are: ``never``, ``optimal`` and ``always``.
   The default value is ``optimal`` which generates compact branches
   when a delay slot cannot be filled. ``never`` disables the usage of
   compact branches and ``always`` generates compact branches whenever
   possible.

**-f[no-]max-type-align=[number]**
   Instruct the code generator to not enforce a higher alignment than the given
   number (of bytes) when accessing memory via an opaque pointer or reference.
   This cap is ignored when directly accessing a variable or when the pointee
   type has an explicit “aligned” attribute.

   The value should usually be determined by the properties of the system allocator.
   Some builtin types, especially vector types, have very high natural alignments;
   when working with values of those types, Clang usually wants to use instructions
   that take advantage of that alignment.  However, many system allocators do
   not promise to return memory that is more than 8-byte or 16-byte-aligned.  Use
   this option to limit the alignment that the compiler can assume for an arbitrary
   pointer, which may point onto the heap.

   This option does not affect the ABI alignment of types; the layout of structs and
   unions and the value returned by the alignof operator remain the same.

   This option can be overridden on a case-by-case basis by putting an explicit
   “aligned” alignment on a struct, union, or typedef.  For example:

   .. code-block:: console

      #include <immintrin.h>
      // Make an aligned typedef of the AVX-512 16-int vector type.
      typedef __v16si __aligned_v16si __attribute__((aligned(64)));

      void initialize_vector(__aligned_v16si *v) {
        // The compiler may assume that ‘v’ is 64-byte aligned, regardless of the
        // value of -fmax-type-align.
      }

.. option:: -faddrsig, -fno-addrsig

   Controls whether Clang emits an address-significance table into the object
   file. Address-significance tables allow linkers to implement `safe ICF
   <https://research.google.com/pubs/archive/36912.pdf>`_ without the false
   positives that can result from other implementation techniques such as
   relocation scanning. Address-significance tables are enabled by default
   on ELF targets when using the integrated assembler. This flag currently
   only has an effect on ELF targets.

**-f[no]-unique-internal-linkage-names**

   Controls whether Clang emits a unique (best-effort) symbol name for internal
   linkage symbols.  When this option is set, compiler hashes the main source
   file path from the command line and appends it to all internal symbols. If a
   program contains multiple objects compiled with the same command-line source
   file path, the symbols are not guaranteed to be unique.  This option is
   particularly useful in attributing profile information to the correct
   function when multiple functions with the same private linkage name exist
   in the binary.

   It should be noted that this option cannot guarantee uniqueness and the
   following is an example where it is not unique when two modules contain
   symbols with the same private linkage name:

   .. code-block:: console

     $ cd $P/foo && clang -c -funique-internal-linkage-names name_conflict.c
     $ cd $P/bar && clang -c -funique-internal-linkage-names name_conflict.c
     $ cd $P && clang foo/name_conflict.o && bar/name_conflict.o

**-fbasic-block-sections=[labels, all, list=<arg>, none]**

  Controls how Clang emits text sections for basic blocks. With values ``all``
  and ``list=<arg>``, each basic block or a subset of basic blocks can be placed
  in its own unique section. With the "labels" value, normal text sections are
  emitted, but a ``.bb_addr_map`` section is emitted which includes address
  offsets for each basic block in the program, relative to the parent function
  address.

  With the ``list=<arg>`` option, a file containing the subset of basic blocks
  that need to placed in unique sections can be specified.  The format of the
  file is as follows.  For example, ``list=spec.txt`` where ``spec.txt`` is the
  following:

  ::

        !foo
        !!2
        !_Z3barv

  will place the machine basic block with ``id 2`` in function ``foo`` in a
  unique section.  It will also place all basic blocks of functions ``bar``
  in unique sections.

  Further, section clusters can also be specified using the ``list=<arg>``
  option.  For example, ``list=spec.txt`` where ``spec.txt`` contains:

  ::

        !foo
        !!1 !!3 !!5
        !!2 !!4 !!6

  will create two unique sections for function ``foo`` with the first
  containing the odd numbered basic blocks and the second containing the
  even numbered basic blocks.

  Basic block sections allow the linker to reorder basic blocks and enables
  link-time optimizations like whole program inter-procedural basic block
  reordering.

Profile Guided Optimization
---------------------------

Profile information enables better optimization. For example, knowing that a
branch is taken very frequently helps the compiler make better decisions when
ordering basic blocks. Knowing that a function ``foo`` is called more
frequently than another function ``bar`` helps the inliner. Optimization
levels ``-O2`` and above are recommended for use of profile guided optimization.

Clang supports profile guided optimization with two different kinds of
profiling. A sampling profiler can generate a profile with very low runtime
overhead, or you can build an instrumented version of the code that collects
more detailed profile information. Both kinds of profiles can provide execution
counts for instructions in the code and information on branches taken and
function invocation.

Regardless of which kind of profiling you use, be careful to collect profiles
by running your code with inputs that are representative of the typical
behavior. Code that is not exercised in the profile will be optimized as if it
is unimportant, and the compiler may make poor optimization choices for code
that is disproportionately used while profiling.

Differences Between Sampling and Instrumentation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Although both techniques are used for similar purposes, there are important
differences between the two:

1. Profile data generated with one cannot be used by the other, and there is no
   conversion tool that can convert one to the other. So, a profile generated
   via ``-fprofile-instr-generate`` must be used with ``-fprofile-instr-use``.
   Similarly, sampling profiles generated by external profilers must be
   converted and used with ``-fprofile-sample-use``.

2. Instrumentation profile data can be used for code coverage analysis and
   optimization.

3. Sampling profiles can only be used for optimization. They cannot be used for
   code coverage analysis. Although it would be technically possible to use
   sampling profiles for code coverage, sample-based profiles are too
   coarse-grained for code coverage purposes; it would yield poor results.

4. Sampling profiles must be generated by an external tool. The profile
   generated by that tool must then be converted into a format that can be read
   by LLVM. The section on sampling profilers describes one of the supported
   sampling profile formats.


Using Sampling Profilers
^^^^^^^^^^^^^^^^^^^^^^^^

Sampling profilers are used to collect runtime information, such as
hardware counters, while your application executes. They are typically
very efficient and do not incur a large runtime overhead. The
sample data collected by the profiler can be used during compilation
to determine what the most executed areas of the code are.

Using the data from a sample profiler requires some changes in the way
a program is built. Before the compiler can use profiling information,
the code needs to execute under the profiler. The following is the
usual build cycle when using sample profilers for optimization:

1. Build the code with source line table information. You can use all the
   usual build flags that you always build your application with. The only
   requirement is that you add ``-gline-tables-only`` or ``-g`` to the
   command line. This is important for the profiler to be able to map
   instructions back to source line locations.

   .. code-block:: console

     $ clang++ -O2 -gline-tables-only code.cc -o code

2. Run the executable under a sampling profiler. The specific profiler
   you use does not really matter, as long as its output can be converted
   into the format that the LLVM optimizer understands. Currently, there
   exists a conversion tool for the Linux Perf profiler
   (https://perf.wiki.kernel.org/), so these examples assume that you
   are using Linux Perf to profile your code.

   .. code-block:: console

     $ perf record -b ./code

   Note the use of the ``-b`` flag. This tells Perf to use the Last Branch
   Record (LBR) to record call chains. While this is not strictly required,
   it provides better call information, which improves the accuracy of
   the profile data.

3. Convert the collected profile data to LLVM's sample profile format.
   This is currently supported via the AutoFDO converter ``create_llvm_prof``.
   It is available at https://github.com/google/autofdo. Once built and
   installed, you can convert the ``perf.data`` file to LLVM using
   the command:

   .. code-block:: console

     $ create_llvm_prof --binary=./code --out=code.prof

   This will read ``perf.data`` and the binary file ``./code`` and emit
   the profile data in ``code.prof``. Note that if you ran ``perf``
   without the ``-b`` flag, you need to use ``--use_lbr=false`` when
   calling ``create_llvm_prof``.

4. Build the code again using the collected profile. This step feeds
   the profile back to the optimizers. This should result in a binary
   that executes faster than the original one. Note that you are not
   required to build the code with the exact same arguments that you
   used in the first step. The only requirement is that you build the code
   with ``-gline-tables-only`` and ``-fprofile-sample-use``.

   .. code-block:: console

     $ clang++ -O2 -gline-tables-only -fprofile-sample-use=code.prof code.cc -o code


Sample Profile Formats
""""""""""""""""""""""

Since external profilers generate profile data in a variety of custom formats,
the data generated by the profiler must be converted into a format that can be
read by the backend. LLVM supports three different sample profile formats:

1. ASCII text. This is the easiest one to generate. The file is divided into
   sections, which correspond to each of the functions with profile
   information. The format is described below. It can also be generated from
   the binary or gcov formats using the ``llvm-profdata`` tool.

2. Binary encoding. This uses a more efficient encoding that yields smaller
   profile files. This is the format generated by the ``create_llvm_prof`` tool
   in https://github.com/google/autofdo.

3. GCC encoding. This is based on the gcov format, which is accepted by GCC. It
   is only interesting in environments where GCC and Clang co-exist. This
   encoding is only generated by the ``create_gcov`` tool in
   https://github.com/google/autofdo. It can be read by LLVM and
   ``llvm-profdata``, but it cannot be generated by either.

If you are using Linux Perf to generate sampling profiles, you can use the
conversion tool ``create_llvm_prof`` described in the previous section.
Otherwise, you will need to write a conversion tool that converts your
profiler's native format into one of these three.


Sample Profile Text Format
""""""""""""""""""""""""""

This section describes the ASCII text format for sampling profiles. It is,
arguably, the easiest one to generate. If you are interested in generating any
of the other two, consult the ``ProfileData`` library in LLVM's source tree
(specifically, ``include/llvm/ProfileData/SampleProfReader.h``).

.. code-block:: console

    function1:total_samples:total_head_samples
     offset1[.discriminator]: number_of_samples [fn1:num fn2:num ... ]
     offset2[.discriminator]: number_of_samples [fn3:num fn4:num ... ]
     ...
     offsetN[.discriminator]: number_of_samples [fn5:num fn6:num ... ]
     offsetA[.discriminator]: fnA:num_of_total_samples
      offsetA1[.discriminator]: number_of_samples [fn7:num fn8:num ... ]
      offsetA1[.discriminator]: number_of_samples [fn9:num fn10:num ... ]
      offsetB[.discriminator]: fnB:num_of_total_samples
       offsetB1[.discriminator]: number_of_samples [fn11:num fn12:num ... ]

This is a nested tree in which the indentation represents the nesting level
of the inline stack. There are no blank lines in the file. And the spacing
within a single line is fixed. Additional spaces will result in an error
while reading the file.

Any line starting with the '#' character is completely ignored.

Inlined calls are represented with indentation. The Inline stack is a
stack of source locations in which the top of the stack represents the
leaf function, and the bottom of the stack represents the actual
symbol to which the instruction belongs.

Function names must be mangled in order for the profile loader to
match them in the current translation unit. The two numbers in the
function header specify how many total samples were accumulated in the
function (first number), and the total number of samples accumulated
in the prologue of the function (second number). This head sample
count provides an indicator of how frequently the function is invoked.

There are two types of lines in the function body.

-  Sampled line represents the profile information of a source location.
   ``offsetN[.discriminator]: number_of_samples [fn5:num fn6:num ... ]``

-  Callsite line represents the profile information of an inlined callsite.
   ``offsetA[.discriminator]: fnA:num_of_total_samples``

Each sampled line may contain several items. Some are optional (marked
below):

a. Source line offset. This number represents the line number
   in the function where the sample was collected. The line number is
   always relative to the line where symbol of the function is
   defined. So, if the function has its header at line 280, the offset
   13 is at line 293 in the file.

   Note that this offset should never be a negative number. This could
   happen in cases like macros. The debug machinery will register the
   line number at the point of macro expansion. So, if the macro was
   expanded in a line before the start of the function, the profile
   converter should emit a 0 as the offset (this means that the optimizers
   will not be able to associate a meaningful weight to the instructions
   in the macro).

b. [OPTIONAL] Discriminator. This is used if the sampled program
   was compiled with DWARF discriminator support
   (http://wiki.dwarfstd.org/index.php?title=Path_Discriminators).
   DWARF discriminators are unsigned integer values that allow the
   compiler to distinguish between multiple execution paths on the
   same source line location.

   For example, consider the line of code ``if (cond) foo(); else bar();``.
   If the predicate ``cond`` is true 80% of the time, then the edge
   into function ``foo`` should be considered to be taken most of the
   time. But both calls to ``foo`` and ``bar`` are at the same source
   line, so a sample count at that line is not sufficient. The
   compiler needs to know which part of that line is taken more
   frequently.

   This is what discriminators provide. In this case, the calls to
   ``foo`` and ``bar`` will be at the same line, but will have
   different discriminator values. This allows the compiler to correctly
   set edge weights into ``foo`` and ``bar``.

c. Number of samples. This is an integer quantity representing the
   number of samples collected by the profiler at this source
   location.

d. [OPTIONAL] Potential call targets and samples. If present, this
   line contains a call instruction. This models both direct and
   number of samples. For example,

   .. code-block:: console

     130: 7  foo:3  bar:2  baz:7

   The above means that at relative line offset 130 there is a call
   instruction that calls one of ``foo()``, ``bar()`` and ``baz()``,
   with ``baz()`` being the relatively more frequently called target.

As an example, consider a program with the call chain ``main -> foo -> bar``.
When built with optimizations enabled, the compiler may inline the
calls to ``bar`` and ``foo`` inside ``main``. The generated profile
could then be something like this:

.. code-block:: console

    main:35504:0
    1: _Z3foov:35504
      2: _Z32bari:31977
      1.1: 31977
    2: 0

This profile indicates that there were a total of 35,504 samples
collected in main. All of those were at line 1 (the call to ``foo``).
Of those, 31,977 were spent inside the body of ``bar``. The last line
of the profile (``2: 0``) corresponds to line 2 inside ``main``. No
samples were collected there.

Profiling with Instrumentation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Clang also supports profiling via instrumentation. This requires building a
special instrumented version of the code and has some runtime
overhead during the profiling, but it provides more detailed results than a
sampling profiler. It also provides reproducible results, at least to the
extent that the code behaves consistently across runs.

Here are the steps for using profile guided optimization with
instrumentation:

1. Build an instrumented version of the code by compiling and linking with the
   ``-fprofile-instr-generate`` option.

   .. code-block:: console

     $ clang++ -O2 -fprofile-instr-generate code.cc -o code

2. Run the instrumented executable with inputs that reflect the typical usage.
   By default, the profile data will be written to a ``default.profraw`` file
   in the current directory. You can override that default by using option
   ``-fprofile-instr-generate=`` or by setting the ``LLVM_PROFILE_FILE`` 
   environment variable to specify an alternate file. If non-default file name
   is specified by both the environment variable and the command line option,
   the environment variable takes precedence. The file name pattern specified
   can include different modifiers: ``%p``, ``%h``, and ``%m``.

   Any instance of ``%p`` in that file name will be replaced by the process
   ID, so that you can easily distinguish the profile output from multiple
   runs.

   .. code-block:: console

     $ LLVM_PROFILE_FILE="code-%p.profraw" ./code

   The modifier ``%h`` can be used in scenarios where the same instrumented
   binary is run in multiple different host machines dumping profile data
   to a shared network based storage. The ``%h`` specifier will be substituted
   with the hostname so that profiles collected from different hosts do not
   clobber each other.

   While the use of ``%p`` specifier can reduce the likelihood for the profiles
   dumped from different processes to clobber each other, such clobbering can still
   happen because of the ``pid`` re-use by the OS. Another side-effect of using
   ``%p`` is that the storage requirement for raw profile data files is greatly
   increased.  To avoid issues like this, the ``%m`` specifier can used in the profile
   name.  When this specifier is used, the profiler runtime will substitute ``%m``
   with a unique integer identifier associated with the instrumented binary. Additionally,
   multiple raw profiles dumped from different processes that share a file system (can be
   on different hosts) will be automatically merged by the profiler runtime during the
   dumping. If the program links in multiple instrumented shared libraries, each library
   will dump the profile data into its own profile data file (with its unique integer
   id embedded in the profile name). Note that the merging enabled by ``%m`` is for raw
   profile data generated by profiler runtime. The resulting merged "raw" profile data
   file still needs to be converted to a different format expected by the compiler (
   see step 3 below).

   .. code-block:: console

     $ LLVM_PROFILE_FILE="code-%m.profraw" ./code


3. Combine profiles from multiple runs and convert the "raw" profile format to
   the input expected by clang. Use the ``merge`` command of the
   ``llvm-profdata`` tool to do this.

   .. code-block:: console

     $ llvm-profdata merge -output=code.profdata code-*.profraw

   Note that this step is necessary even when there is only one "raw" profile,
   since the merge operation also changes the file format.

4. Build the code again using the ``-fprofile-instr-use`` option to specify the
   collected profile data.

   .. code-block:: console

     $ clang++ -O2 -fprofile-instr-use=code.profdata code.cc -o code

   You can repeat step 4 as often as you like without regenerating the
   profile. As you make changes to your code, clang may no longer be able to
   use the profile data. It will warn you when this happens.

Profile generation using an alternative instrumentation method can be
controlled by the GCC-compatible flags ``-fprofile-generate`` and
``-fprofile-use``. Although these flags are semantically equivalent to
their GCC counterparts, they *do not* handle GCC-compatible profiles.
They are only meant to implement GCC's semantics with respect to
profile creation and use. Flag ``-fcs-profile-generate`` also instruments
programs using the same instrumentation method as ``-fprofile-generate``.

.. option:: -fprofile-generate[=<dirname>]

  The ``-fprofile-generate`` and ``-fprofile-generate=`` flags will use
  an alternative instrumentation method for profile generation. When
  given a directory name, it generates the profile file
  ``default_%m.profraw`` in the directory named ``dirname`` if specified.
  If ``dirname`` does not exist, it will be created at runtime. ``%m`` specifier
  will be substituted with a unique id documented in step 2 above. In other words,
  with ``-fprofile-generate[=<dirname>]`` option, the "raw" profile data automatic
  merging is turned on by default, so there will no longer any risk of profile
  clobbering from different running processes.  For example,

  .. code-block:: console

    $ clang++ -O2 -fprofile-generate=yyy/zzz code.cc -o code

  When ``code`` is executed, the profile will be written to the file
  ``yyy/zzz/default_xxxx.profraw``.

  To generate the profile data file with the compiler readable format, the 
  ``llvm-profdata`` tool can be used with the profile directory as the input:

   .. code-block:: console

     $ llvm-profdata merge -output=code.profdata yyy/zzz/

 If the user wants to turn off the auto-merging feature, or simply override the
 the profile dumping path specified at command line, the environment variable
 ``LLVM_PROFILE_FILE`` can still be used to override
 the directory and filename for the profile file at runtime.

.. option:: -fcs-profile-generate[=<dirname>]

  The ``-fcs-profile-generate`` and ``-fcs-profile-generate=`` flags will use
  the same instrumentation method, and generate the same profile as in the
  ``-fprofile-generate`` and ``-fprofile-generate=`` flags. The difference is
  that the instrumentation is performed after inlining so that the resulted
  profile has a better context sensitive information. They cannot be used
  together with ``-fprofile-generate`` and ``-fprofile-generate=`` flags.
  They are typically used in conjunction with ``-fprofile-use`` flag.
  The profile generated by ``-fcs-profile-generate`` and ``-fprofile-generate``
  can be merged by llvm-profdata. A use example:

  .. code-block:: console

    $ clang++ -O2 -fprofile-generate=yyy/zzz code.cc -o code
    $ ./code
    $ llvm-profdata merge -output=code.profdata yyy/zzz/

  The first few steps are the same as that in ``-fprofile-generate``
  compilation. Then perform a second round of instrumentation.

  .. code-block:: console

    $ clang++ -O2 -fprofile-use=code.profdata -fcs-profile-generate=sss/ttt \
      -o cs_code
    $ ./cs_code
    $ llvm-profdata merge -output=cs_code.profdata sss/ttt code.profdata

  The resulted ``cs_code.prodata`` combines ``code.profdata`` and the profile
  generated from binary ``cs_code``. Profile ``cs_code.profata`` can be used by
  ``-fprofile-use`` compilation.

  .. code-block:: console

    $ clang++ -O2 -fprofile-use=cs_code.profdata

  The above command will read both profiles to the compiler at the identical
  point of instrumentations.

.. option:: -fprofile-use[=<pathname>]

  Without any other arguments, ``-fprofile-use`` behaves identically to
  ``-fprofile-instr-use``. Otherwise, if ``pathname`` is the full path to a
  profile file, it reads from that file. If ``pathname`` is a directory name,
  it reads from ``pathname/default.profdata``.

.. option:: -fprofile-update[=<method>]

  Unless ``-fsanitize=thread`` is specified, the default is ``single``, which
  uses non-atomic increments. The counters can be inaccurate under thread
  contention. ``atomic`` uses atomic increments which is accurate but has
  overhead. ``prefer-atomic`` will be transformed to ``atomic`` when supported
  by the target, or ``single`` otherwise.

  This option currently works with ``-fprofile-arcs`` and ``-fprofile-instr-generate``,
  but not with ``-fprofile-generate``.

Disabling Instrumentation
^^^^^^^^^^^^^^^^^^^^^^^^^

In certain situations, it may be useful to disable profile generation or use
for specific files in a build, without affecting the main compilation flags
used for the other files in the project.

In these cases, you can use the flag ``-fno-profile-instr-generate`` (or
``-fno-profile-generate``) to disable profile generation, and
``-fno-profile-instr-use`` (or ``-fno-profile-use``) to disable profile use.

Note that these flags should appear after the corresponding profile
flags to have an effect.

Instrumenting only selected files or functions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Sometimes it's useful to only instrument certain files or functions.  For
example in automated testing infrastructure, it may be desirable to only
instrument files or functions that were modified by a patch to reduce the
overhead of instrumenting a full system.

This can be done using the ``-fprofile-list`` option.

.. option:: -fprofile-list=<pathname>

  This option can be used to apply profile instrumentation only to selected
  files or functions. ``pathname`` should point to a file in the
  :doc:`SanitizerSpecialCaseList` format which selects which files and
  functions to instrument.

  .. code-block:: console

    $ echo "fun:test" > fun.list
    $ clang++ -O2 -fprofile-instr-generate -fprofile-list=fun.list code.cc -o code

The option can be specified multiple times to pass multiple files.

.. code-block:: console

    $ echo "!fun:*test*" > fun.list
    $ echo "src:code.cc" > src.list
    % clang++ -O2 -fprofile-instr-generate -fcoverage-mapping -fprofile-list=fun.list -fprofile-list=code.list code.cc -o code

To filter individual functions or entire source files using ``fun:<name>`` or
``src:<file>`` respectively. To exclude a function or a source file, use
``!fun:<name>`` or ``!src:<file>`` respectively. The format also supports
wildcard expansion. The compiler generated functions are assumed to be located
in the main source file.  It is also possible to restrict the filter to a
particular instrumentation type by using a named section.

.. code-block:: none

  # all functions whose name starts with foo will be instrumented.
  fun:foo*

  # except for foo1 which will be excluded from instrumentation.
  !fun:foo1

  # every function in path/to/foo.cc will be instrumented.
  src:path/to/foo.cc

  # bar will be instrumented only when using backend instrumentation.
  # Recognized section names are clang, llvm and csllvm.
  [llvm]
  fun:bar

When the file contains only excludes, all files and functions except for the
excluded ones will be instrumented. Otherwise, only the files and functions
specified will be instrumented.

Profile remapping
^^^^^^^^^^^^^^^^^

When the program is compiled after a change that affects many symbol names,
pre-existing profile data may no longer match the program. For example:

 * switching from libstdc++ to libc++ will result in the mangled names of all
   functions taking standard library types to change
 * renaming a widely-used type in C++ will result in the mangled names of all
   functions that have parameters involving that type to change
 * moving from a 32-bit compilation to a 64-bit compilation may change the
   underlying type of ``size_t`` and similar types, resulting in changes to
   manglings

Clang allows use of a profile remapping file to specify that such differences
in mangled names should be ignored when matching the profile data against the
program.

.. option:: -fprofile-remapping-file=<file>

  Specifies a file containing profile remapping information, that will be
  used to match mangled names in the profile data to mangled names in the
  program.

The profile remapping file is a text file containing lines of the form

.. code-block:: text

  fragmentkind fragment1 fragment2

where ``fragmentkind`` is one of ``name``, ``type``, or ``encoding``,
indicating whether the following mangled name fragments are
<`name <https://itanium-cxx-abi.github.io/cxx-abi/abi.html#mangle.name>`_>s,
<`type <https://itanium-cxx-abi.github.io/cxx-abi/abi.html#mangle.type>`_>s, or
<`encoding <https://itanium-cxx-abi.github.io/cxx-abi/abi.html#mangle.encoding>`_>s,
respectively.
Blank lines and lines starting with ``#`` are ignored.

For convenience, built-in <substitution>s such as ``St`` and ``Ss``
are accepted as <name>s (even though they technically are not <name>s).

For example, to specify that ``absl::string_view`` and ``std::string_view``
should be treated as equivalent when matching profile data, the following
remapping file could be used:

.. code-block:: text

  # absl::string_view is considered equivalent to std::string_view
  type N4absl11string_viewE St17basic_string_viewIcSt11char_traitsIcEE

  # std:: might be std::__1:: in libc++ or std::__cxx11:: in libstdc++
  name 3std St3__1
  name 3std St7__cxx11

Matching profile data using a profile remapping file is supported on a
best-effort basis. For example, information regarding indirect call targets is
currently not remapped. For best results, you are encouraged to generate new
profile data matching the updated program, or to remap the profile data
using the ``llvm-cxxmap`` and ``llvm-profdata merge`` tools.

.. note::

  Profile data remapping support is currently only implemented for LLVM's
  new pass manager, which can be enabled with
  ``-fexperimental-new-pass-manager``.

.. note::

  Profile data remapping is currently only supported for C++ mangled names
  following the Itanium C++ ABI mangling scheme. This covers all C++ targets
  supported by Clang other than Windows.

GCOV-based Profiling
--------------------

GCOV is a test coverage program, it helps to know how often a line of code
is executed. When instrumenting the code with ``--coverage`` option, some
counters are added for each edge linking basic blocks.

At compile time, gcno files are generated containing information about
blocks and edges between them. At runtime the counters are incremented and at
exit the counters are dumped in gcda files.

The tool ``llvm-cov gcov`` will parse gcno, gcda and source files to generate
a report ``.c.gcov``.

.. option:: -fprofile-filter-files=[regexes]

  Define a list of regexes separated by a semi-colon.
  If a file name matches any of the regexes then the file is instrumented.

   .. code-block:: console

     $ clang --coverage -fprofile-filter-files=".*\.c$" foo.c

  For example, this will only instrument files finishing with ``.c``, skipping ``.h`` files.

.. option:: -fprofile-exclude-files=[regexes]

  Define a list of regexes separated by a semi-colon.
  If a file name doesn't match all the regexes then the file is instrumented.

  .. code-block:: console

     $ clang --coverage -fprofile-exclude-files="^/usr/include/.*$" foo.c

  For example, this will instrument all the files except the ones in ``/usr/include``.

If both options are used then a file is instrumented if its name matches any
of the regexes from ``-fprofile-filter-list`` and doesn't match all the regexes
from ``-fprofile-exclude-list``.

.. code-block:: console

   $ clang --coverage -fprofile-exclude-files="^/usr/include/.*$" \
           -fprofile-filter-files="^/usr/.*$"
          
In that case ``/usr/foo/oof.h`` is instrumented since it matches the filter regex and
doesn't match the exclude regex, but ``/usr/include/foo.h`` doesn't since it matches
the exclude regex.

Controlling Debug Information
-----------------------------

Controlling Size of Debug Information
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Debug info kind generated by Clang can be set by one of the flags listed
below. If multiple flags are present, the last one is used.

.. option:: -g0

  Don't generate any debug info (default).

.. option:: -gline-tables-only

  Generate line number tables only.

  This kind of debug info allows to obtain stack traces with function names,
  file names and line numbers (by such tools as ``gdb`` or ``addr2line``).  It
  doesn't contain any other data (e.g. description of local variables or
  function parameters).

.. option:: -fstandalone-debug

  Clang supports a number of optimizations to reduce the size of debug
  information in the binary. They work based on the assumption that
  the debug type information can be spread out over multiple
  compilation units.  For instance, Clang will not emit type
  definitions for types that are not needed by a module and could be
  replaced with a forward declaration.  Further, Clang will only emit
  type info for a dynamic C++ class in the module that contains the
  vtable for the class.

  The **-fstandalone-debug** option turns off these optimizations.
  This is useful when working with 3rd-party libraries that don't come
  with debug information.  Note that Clang will never emit type
  information for types that are not referenced at all by the program.

.. option:: -fno-standalone-debug

   On Darwin **-fstandalone-debug** is enabled by default. The
   **-fno-standalone-debug** option can be used to get to turn on the
   vtable-based optimization described above.

.. option:: -fuse-ctor-homing

   This optimization is similar to the optimizations that are enabled as part
   of -fno-standalone-debug. Here, Clang only emits type info for a
   non-trivial, non-aggregate C++ class in the modules that contain a
   definition of one of its constructors. This relies on the additional
   assumption that all classes that are not trivially constructible have a
   non-trivial constructor that is used somewhere.

   This flag is not enabled by default, and needs to be used with -cc1 or
   -Xclang.

.. option:: -g

  Generate complete debug info.

.. option:: -feliminate-unused-debug-types

  By default, Clang does not emit type information for types that are defined
  but not used in a program. To retain the debug info for these unused types,
  the negation **-fno-eliminate-unused-debug-types** can be used.

Controlling Macro Debug Info Generation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Debug info for C preprocessor macros increases the size of debug information in
the binary. Macro debug info generated by Clang can be controlled by the flags
listed below.

.. option:: -fdebug-macro

  Generate debug info for preprocessor macros. This flag is discarded when
  **-g0** is enabled.

.. option:: -fno-debug-macro

  Do not generate debug info for preprocessor macros (default).

Controlling Debugger "Tuning"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

While Clang generally emits standard DWARF debug info (http://dwarfstd.org),
different debuggers may know how to take advantage of different specific DWARF
features. You can "tune" the debug info for one of several different debuggers.

.. option:: -ggdb, -glldb, -gsce, -gdbx

  Tune the debug info for the ``gdb``, ``lldb``, Sony PlayStation\ |reg|
  debugger, or ``dbx``, respectively. Each of these options implies **-g**.
  (Therefore, if you want both **-gline-tables-only** and debugger tuning, the
  tuning option must come first.)

Controlling LLVM IR Output
--------------------------

Controlling Value Names in LLVM IR
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Emitting value names in LLVM IR increases the size and verbosity of the IR.
By default, value names are only emitted in assertion-enabled builds of Clang.
However, when reading IR it can be useful to re-enable the emission of value
names to improve readability.

.. option:: -fdiscard-value-names

  Discard value names when generating LLVM IR.

.. option:: -fno-discard-value-names

  Do not discard value names when generating LLVM IR. This option can be used
  to re-enable names for release builds of Clang.


Comment Parsing Options
-----------------------

Clang parses Doxygen and non-Doxygen style documentation comments and attaches
them to the appropriate declaration nodes.  By default, it only parses
Doxygen-style comments and ignores ordinary comments starting with ``//`` and
``/*``.

.. option:: -Wdocumentation

  Emit warnings about use of documentation comments.  This warning group is off
  by default.

  This includes checking that ``\param`` commands name parameters that actually
  present in the function signature, checking that ``\returns`` is used only on
  functions that actually return a value etc.

.. option:: -Wno-documentation-unknown-command

  Don't warn when encountering an unknown Doxygen command.

.. option:: -fparse-all-comments

  Parse all comments as documentation comments (including ordinary comments
  starting with ``//`` and ``/*``).

.. option:: -fcomment-block-commands=[commands]

  Define custom documentation commands as block commands.  This allows Clang to
  construct the correct AST for these custom commands, and silences warnings
  about unknown commands.  Several commands must be separated by a comma
  *without trailing space*; e.g. ``-fcomment-block-commands=foo,bar`` defines
  custom commands ``\foo`` and ``\bar``.

  It is also possible to use ``-fcomment-block-commands`` several times; e.g.
  ``-fcomment-block-commands=foo -fcomment-block-commands=bar`` does the same
  as above.

.. _c:

C Language Features
===================

The support for standard C in clang is feature-complete except for the
C99 floating-point pragmas.

Extensions supported by clang
-----------------------------

See :doc:`LanguageExtensions`.

Differences between various standard modes
------------------------------------------

clang supports the -std option, which changes what language mode clang uses.
The supported modes for C are c89, gnu89, c94, c99, gnu99, c11, gnu11, c17,
gnu17, c2x, gnu2x, and various aliases for those modes. If no -std option is
specified, clang defaults to gnu17 mode. Many C99 and C11 features are
supported in earlier modes as a conforming extension, with a warning. Use
``-pedantic-errors`` to request an error if a feature from a later standard
revision is used in an earlier mode.

Differences between all ``c*`` and ``gnu*`` modes:

-  ``c*`` modes define "``__STRICT_ANSI__``".
-  Target-specific defines not prefixed by underscores, like ``linux``,
   are defined in ``gnu*`` modes.
-  Trigraphs default to being off in ``gnu*`` modes; they can be enabled
   by the ``-trigraphs`` option.
-  The parser recognizes ``asm`` and ``typeof`` as keywords in ``gnu*`` modes;
   the variants ``__asm__`` and ``__typeof__`` are recognized in all modes.
-  The parser recognizes ``inline`` as a keyword in ``gnu*`` mode, in
   addition to recognizing it in the ``*99`` and later modes for which it is
   part of the ISO C standard. The variant ``__inline__`` is recognized in all
   modes.
-  The Apple "blocks" extension is recognized by default in ``gnu*`` modes
   on some platforms; it can be enabled in any mode with the ``-fblocks``
   option.

Differences between ``*89`` and ``*94`` modes:

-  Digraphs are not recognized in c89 mode.

Differences between ``*94`` and ``*99`` modes:

-  The ``*99`` modes default to implementing ``inline`` / ``__inline__``
   as specified in C99, while the ``*89`` modes implement the GNU version.
   This can be overridden for individual functions with the ``__gnu_inline__``
   attribute.
-  The scope of names defined inside a ``for``, ``if``, ``switch``, ``while``,
   or ``do`` statement is different. (example: ``if ((struct x {int x;}*)0) {}``.)
-  ``__STDC_VERSION__`` is not defined in ``*89`` modes.
-  ``inline`` is not recognized as a keyword in ``c89`` mode.
-  ``restrict`` is not recognized as a keyword in ``*89`` modes.
-  Commas are allowed in integer constant expressions in ``*99`` modes.
-  Arrays which are not lvalues are not implicitly promoted to pointers
   in ``*89`` modes.
-  Some warnings are different.

Differences between ``*99`` and ``*11`` modes:

-  Warnings for use of C11 features are disabled.
-  ``__STDC_VERSION__`` is defined to ``201112L`` rather than ``199901L``.

Differences between ``*11`` and ``*17`` modes:

-  ``__STDC_VERSION__`` is defined to ``201710L`` rather than ``201112L``.

GCC extensions not implemented yet
----------------------------------

clang tries to be compatible with gcc as much as possible, but some gcc
extensions are not implemented yet:

-  clang does not support decimal floating point types (``_Decimal32`` and
   friends) yet.
-  clang does not support nested functions; this is a complex feature
   which is infrequently used, so it is unlikely to be implemented
   anytime soon. In C++11 it can be emulated by assigning lambda
   functions to local variables, e.g:

   .. code-block:: cpp

     auto const local_function = [&](int parameter) {
       // Do something
     };
     ...
     local_function(1);

-  clang only supports global register variables when the register specified
   is non-allocatable (e.g. the stack pointer). Support for general global
   register variables is unlikely to be implemented soon because it requires
   additional LLVM backend support.
-  clang does not support static initialization of flexible array
   members. This appears to be a rarely used extension, but could be
   implemented pending user demand.
-  clang does not support
   ``__builtin_va_arg_pack``/``__builtin_va_arg_pack_len``. This is
   used rarely, but in some potentially interesting places, like the
   glibc headers, so it may be implemented pending user demand. Note
   that because clang pretends to be like GCC 4.2, and this extension
   was introduced in 4.3, the glibc headers will not try to use this
   extension with clang at the moment.
-  clang does not support the gcc extension for forward-declaring
   function parameters; this has not shown up in any real-world code
   yet, though, so it might never be implemented.

This is not a complete list; if you find an unsupported extension
missing from this list, please send an e-mail to cfe-dev. This list
currently excludes C++; see :ref:`C++ Language Features <cxx>`. Also, this
list does not include bugs in mostly-implemented features; please see
the `bug
tracker <https://bugs.llvm.org/buglist.cgi?quicksearch=product%3Aclang+component%3A-New%2BBugs%2CAST%2CBasic%2CDriver%2CHeaders%2CLLVM%2BCodeGen%2Cparser%2Cpreprocessor%2CSemantic%2BAnalyzer>`_
for known existing bugs (FIXME: Is there a section for bug-reporting
guidelines somewhere?).

Intentionally unsupported GCC extensions
----------------------------------------

-  clang does not support the gcc extension that allows variable-length
   arrays in structures. This is for a few reasons: one, it is tricky to
   implement, two, the extension is completely undocumented, and three,
   the extension appears to be rarely used. Note that clang *does*
   support flexible array members (arrays with a zero or unspecified
   size at the end of a structure).
-  GCC accepts many expression forms that are not valid integer constant
   expressions in bit-field widths, enumerator constants, case labels,
   and in array bounds at global scope. Clang also accepts additional
   expression forms in these contexts, but constructs that GCC accepts due to
   simplifications GCC performs while parsing, such as ``x - x`` (where ``x`` is a
   variable) will likely never be accepted by Clang.
-  clang does not support ``__builtin_apply`` and friends; this extension
   is extremely obscure and difficult to implement reliably.

.. _c_ms:

Microsoft extensions
--------------------

clang has support for many extensions from Microsoft Visual C++. To enable these
extensions, use the ``-fms-extensions`` command-line option. This is the default
for Windows targets. Clang does not implement every pragma or declspec provided
by MSVC, but the popular ones, such as ``__declspec(dllexport)`` and ``#pragma
comment(lib)`` are well supported.

clang has a ``-fms-compatibility`` flag that makes clang accept enough
invalid C++ to be able to parse most Microsoft headers. For example, it
allows `unqualified lookup of dependent base class members
<https://clang.llvm.org/compatibility.html#dep_lookup_bases>`_, which is
a common compatibility issue with clang. This flag is enabled by default
for Windows targets.

``-fdelayed-template-parsing`` lets clang delay parsing of function template
definitions until the end of a translation unit. This flag is enabled by
default for Windows targets.

For compatibility with existing code that compiles with MSVC, clang defines the
``_MSC_VER`` and ``_MSC_FULL_VER`` macros. These default to the values of 1800
and 180000000 respectively, making clang look like an early release of Visual
C++ 2013. The ``-fms-compatibility-version=`` flag overrides these values.  It
accepts a dotted version tuple, such as 19.00.23506. Changing the MSVC
compatibility version makes clang behave more like that version of MSVC. For
example, ``-fms-compatibility-version=19`` will enable C++14 features and define
``char16_t`` and ``char32_t`` as builtin types.

.. _cxx:

C++ Language Features
=====================

clang fully implements all of standard C++98 except for exported
templates (which were removed in C++11), all of standard C++11,
C++14, and C++17, and most of C++20.

See the `C++ support in Clang <https://clang.llvm.org/cxx_status.html>` page
for detailed information on C++ feature support across Clang versions.

Controlling implementation limits
---------------------------------

.. option:: -fbracket-depth=N

  Sets the limit for nested parentheses, brackets, and braces to N.  The
  default is 256.

.. option:: -fconstexpr-depth=N

  Sets the limit for recursive constexpr function invocations to N.  The
  default is 512.

.. option:: -fconstexpr-steps=N

  Sets the limit for the number of full-expressions evaluated in a single
  constant expression evaluation.  The default is 1048576.

.. option:: -ftemplate-depth=N

  Sets the limit for recursively nested template instantiations to N.  The
  default is 1024.

.. option:: -foperator-arrow-depth=N

  Sets the limit for iterative calls to 'operator->' functions to N.  The
  default is 256.

.. _objc:

Objective-C Language Features
=============================

.. _objcxx:

Objective-C++ Language Features
===============================

.. _openmp:

OpenMP Features
===============

Clang supports all OpenMP 4.5 directives and clauses. See :doc:`OpenMPSupport`
for additional details.

Use `-fopenmp` to enable OpenMP. Support for OpenMP can be disabled with
`-fno-openmp`.

Use `-fopenmp-simd` to enable OpenMP simd features only, without linking
the runtime library; for combined constructs
(e.g. ``#pragma omp parallel for simd``) the non-simd directives and clauses
will be ignored. This can be disabled with `-fno-openmp-simd`.

Controlling implementation limits
---------------------------------

.. option:: -fopenmp-use-tls

 Controls code generation for OpenMP threadprivate variables. In presence of
 this option all threadprivate variables are generated the same way as thread
 local variables, using TLS support. If `-fno-openmp-use-tls`
 is provided or target does not support TLS, code generation for threadprivate
 variables relies on OpenMP runtime library.

.. _opencl:

OpenCL Features
===============

Clang can be used to compile OpenCL kernels for execution on a device
(e.g. GPU). It is possible to compile the kernel into a binary (e.g. for AMDGPU)
that can be uploaded to run directly on a device (e.g. using
`clCreateProgramWithBinary
<https://www.khronos.org/registry/OpenCL/specs/opencl-1.1.pdf#111>`_) or
into generic bitcode files loadable into other toolchains.

Compiling to a binary using the default target from the installation can be done
as follows:

   .. code-block:: console

     $ echo "kernel void k(){}" > test.cl
     $ clang test.cl

Compiling for a specific target can be done by specifying the triple corresponding
to the target, for example:

   .. code-block:: console

     $ clang -target nvptx64-unknown-unknown test.cl
     $ clang -target amdgcn-amd-amdhsa -mcpu=gfx900 test.cl

Compiling to bitcode can be done as follows:

   .. code-block:: console

     $ clang -c -emit-llvm test.cl

This will produce a file `test.bc` that can be used in vendor toolchains
to perform machine code generation.

Note that if compiled to bitcode for generic targets such as SPIR,
portable IR is produced that can be used with various vendor
tools as well as open source tools such as `SPIRV-LLVM Translator
<https://github.com/KhronosGroup/SPIRV-LLVM-Translator>`_
to produce SPIR-V binary. More details are provided in `the offline
compilation from OpenCL kernel sources into SPIR-V using open source
tools
<https://github.com/KhronosGroup/OpenCL-Guide/blob/main/chapters/os_tooling.md>`_.

Clang currently supports OpenCL C language standards up to v2.0. Clang mainly
supports full profile. There is only very limited support of the embedded
profile. 
Starting from clang 9 a C++ mode is available for OpenCL (see
:ref:`C++ for OpenCL <cxx_for_opencl>`).

There is ongoing support for OpenCL v3.0 that is documented along with other
experimental functionality and features in development on :doc:`OpenCLSupport`
page.

OpenCL Specific Options
-----------------------

Most of the OpenCL build options from `the specification v2.0 section 5.8.4
<https://www.khronos.org/registry/cl/specs/opencl-2.0.pdf#200>`_ are available.

Examples:

   .. code-block:: console

     $ clang -cl-std=CL2.0 -cl-single-precision-constant test.cl


Many flags used for the compilation for C sources can also be passed while
compiling for OpenCL, examples: ``-c``, ``-O<1-4|s>``, ``-o``, ``-emit-llvm``, etc.

Some extra options are available to support special OpenCL features.

.. _opencl_cl_no_stdinc:

.. option:: -cl-no-stdinc

Allows to disable all extra types and functions that are not native to the compiler.
This might reduce the compilation speed marginally but many declarations from the
OpenCL standard will not be accessible. For example, the following will fail to
compile.

   .. code-block:: console

     $ echo "bool is_wg_uniform(int i){return get_enqueued_local_size(i)==get_local_size(i);}" > test.cl
     $ clang -cl-std=CL2.0 -cl-no-stdinc test.cl
     error: use of undeclared identifier 'get_enqueued_local_size'
     error: use of undeclared identifier 'get_local_size'

More information about the standard types and functions is provided in :ref:`the
section on the OpenCL Header <opencl_header>`.

OpenCL Targets
--------------

OpenCL targets are derived from the regular Clang target classes. The OpenCL
specific parts of the target representation provide address space mapping as
well as a set of supported extensions.

Specific Targets
^^^^^^^^^^^^^^^^

There is a set of concrete HW architectures that OpenCL can be compiled for.

- For AMD target:

   .. code-block:: console

     $ clang -target amdgcn-amd-amdhsa -mcpu=gfx900 test.cl

- For Nvidia architectures:

   .. code-block:: console

     $ clang -target nvptx64-unknown-unknown test.cl


Generic Targets
^^^^^^^^^^^^^^^

- SPIR is available as a generic target to allow portable bitcode to be produced
  that can be used across GPU toolchains. The implementation follows `the SPIR
  specification <https://www.khronos.org/spir>`_. There are two flavors
  available for 32 and 64 bits.

   .. code-block:: console

    $ clang -target spir test.cl -emit-llvm -c
    $ clang -target spir64 test.cl -emit-llvm -c

  All known OpenCL extensions are supported in the SPIR targets. Clang will
  generate SPIR v1.2 compatible IR for OpenCL versions up to 2.0 and SPIR v2.0
  for OpenCL v2.0 or C++ for OpenCL.

- x86 is used by some implementations that are x86 compatible and currently
  remains for backwards compatibility (with older implementations prior to
  SPIR target support). For "non-SPMD" targets which cannot spawn multiple
  work-items on the fly using hardware, which covers practically all non-GPU
  devices such as CPUs and DSPs, additional processing is needed for the kernels
  to support multiple work-item execution. For this, a 3rd party toolchain,
  such as for example `POCL <http://portablecl.org/>`_, can be used.

  This target does not support multiple memory segments and, therefore, the fake
  address space map can be added using the :ref:`-ffake-address-space-map
  <opencl_fake_address_space_map>` flag.

.. _opencl_header:

OpenCL Header
-------------

By default Clang will include standard headers and therefore most of OpenCL
builtin functions and types are available during compilation. The
default declarations of non-native compiler types and functions can be disabled
by using flag :ref:`-cl-no-stdinc <opencl_cl_no_stdinc>`.

The following example demonstrates that OpenCL kernel sources with various
standard builtin functions can be compiled without the need for an explicit
includes or compiler flags.

   .. code-block:: console

     $ echo "bool is_wg_uniform(int i){return get_enqueued_local_size(i)==get_local_size(i);}" > test.cl
     $ clang -cl-std=CL2.0 test.cl

More information about the default headers is provided in :doc:`OpenCLSupport`.

OpenCL Extensions
-----------------

Most of the ``cl_khr_*`` extensions to OpenCL C from `the official OpenCL
registry <https://www.khronos.org/registry/OpenCL/>`_ are available and
configured per target depending on the support available in the specific
architecture.

It is possible to alter the default extensions setting per target using
``-cl-ext`` flag. (See :ref:`flags description <opencl_cl_ext>` for more details).

Vendor extensions can be added flexibly by declaring the list of types and
functions associated with each extensions enclosed within the following
compiler pragma directives:

  .. code-block:: c

       #pragma OPENCL EXTENSION the_new_extension_name : begin
       // declare types and functions associated with the extension here
       #pragma OPENCL EXTENSION the_new_extension_name : end

For example, parsing the following code adds ``my_t`` type and ``my_func``
function to the custom ``my_ext`` extension.

  .. code-block:: c

       #pragma OPENCL EXTENSION my_ext : begin
       typedef struct{
         int a;
       }my_t;
       void my_func(my_t);
       #pragma OPENCL EXTENSION my_ext : end

There is no conflict resolution for identifier clashes among extensions.
It is therefore recommended that the identifiers are prefixed with a
double underscore to avoid clashing with user space identifiers. Vendor
extension should use reserved identifier prefix e.g. amd, arm, intel.

Clang also supports language extensions documented in `The OpenCL C Language
Extensions Documentation
<https://github.com/KhronosGroup/Khronosdotorg/blob/master/api/opencl/assets/OpenCL_LangExt.pdf>`_.

OpenCL-Specific Attributes
--------------------------

OpenCL support in Clang contains a set of attribute taken directly from the
specification as well as additional attributes.

See also :doc:`AttributeReference`.

nosvm
^^^^^

Clang supports this attribute to comply to OpenCL v2.0 conformance, but it
does not have any effect on the IR. For more details reffer to the specification
`section 6.7.2
<https://www.khronos.org/registry/cl/specs/opencl-2.0-openclc.pdf#49>`_


opencl_unroll_hint
^^^^^^^^^^^^^^^^^^

The implementation of this feature mirrors the unroll hint for C.
More details on the syntax can be found in the specification
`section 6.11.5
<https://www.khronos.org/registry/cl/specs/opencl-2.0-openclc.pdf#61>`_

convergent
^^^^^^^^^^

To make sure no invalid optimizations occur for single program multiple data
(SPMD) / single instruction multiple thread (SIMT) Clang provides attributes that
can be used for special functions that have cross work item semantics.
An example is the subgroup operations such as `intel_sub_group_shuffle 
<https://www.khronos.org/registry/cl/extensions/intel/cl_intel_subgroups.txt>`_

   .. code-block:: c

     // Define custom my_sub_group_shuffle(data, c)
     // that makes use of intel_sub_group_shuffle
     r1 = ... 
     if (r0) r1 = computeA();
     // Shuffle data from r1 into r3
     // of threads id r2.
     r3 = my_sub_group_shuffle(r1, r2);
     if (r0) r3 = computeB();

with non-SPMD semantics this is optimized to the following equivalent code:

   .. code-block:: c

     r1 = ...
     if (!r0)
       // Incorrect functionality! The data in r1
       // have not been computed by all threads yet.
       r3 = my_sub_group_shuffle(r1, r2);
     else {
       r1 = computeA();
       r3 = my_sub_group_shuffle(r1, r2);
       r3 = computeB();
     }

Declaring the function ``my_sub_group_shuffle`` with the convergent attribute
would prevent this:

   .. code-block:: c

     my_sub_group_shuffle() __attribute__((convergent));

Using ``convergent`` guarantees correct execution by keeping CFG equivalence
wrt operations marked as ``convergent``. CFG ``G´`` is equivalent to ``G`` wrt
node ``Ni`` : ``iff ∀ Nj (i≠j)`` domination and post-domination relations with
respect to ``Ni`` remain the same in both ``G`` and ``G´``. 

noduplicate
^^^^^^^^^^^

``noduplicate`` is more restrictive with respect to optimizations than
``convergent`` because a convergent function only preserves CFG equivalence.
This allows some optimizations to happen as long as the control flow remains
unmodified.

   .. code-block:: c

     for (int i=0; i<4; i++)
       my_sub_group_shuffle()

can be modified to:

   .. code-block:: c

     my_sub_group_shuffle();
     my_sub_group_shuffle();
     my_sub_group_shuffle();
     my_sub_group_shuffle();

while using ``noduplicate`` would disallow this. Also ``noduplicate`` doesn't
have the same safe semantics of CFG as ``convergent`` and can cause changes in
CFG that modify semantics of the original program.

``noduplicate`` is kept for backwards compatibility only and it considered to be
deprecated for future uses.

.. _cxx_for_opencl:

C++ for OpenCL
--------------

Starting from clang 9 kernel code can contain C++17 features: classes, templates,
function overloading, type deduction, etc. Please note that this is not an
implementation of `OpenCL C++
<https://www.khronos.org/registry/OpenCL/specs/2.2/pdf/OpenCL_Cxx.pdf>`_ and
there is no plan to support it in clang in any new releases in the near future.


Clang currently supports C++ for OpenCL v1.0.
For detailed information about this language refer to the C++ for OpenCL
Programming Language Documentation available
in `the latest build
<https://www.khronos.org/opencl/assets/CXX_for_OpenCL.html>`_
or in `the official release
<https://github.com/KhronosGroup/OpenCL-Docs/releases/tag/cxxforopencl-v1.0-r2>`_.

To enable the C++ for OpenCL mode, pass one of following command line options when
compiling ``.cl`` file ``-cl-std=clc++``, ``-cl-std=CLC++``, ``-std=clc++`` or
``-std=CLC++``.

   .. code-block:: c++

     template<class T> T add( T x, T y )
     {
       return x + y;
     }

     __kernel void test( __global float* a, __global float* b)
     {
       auto index = get_global_id(0);
       a[index] = add(b[index], b[index+1]);
     }


   .. code-block:: console

     clang -cl-std=clc++ test.cl

Alternatively, files with ``.clcpp`` extension are compiled with the C++ for OpenCL
mode.

   .. code-block:: console

     clang test.clcpp

C++ for OpenCL kernel sources can also be compiled online in drivers supporting 
`cl_ext_cxx_for_opencl
<https://www.khronos.org/registry/OpenCL/extensions/ext/cl_ext_cxx_for_opencl.html>`_
extension.

Constructing and destroying global objects
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Global objects with non-trivial constructors require the constructors to be run
before the first kernel using the global objects is executed. Similarly global
objects with non-trivial destructors require destructor invocation just after
the last kernel using the program objects is executed.
In OpenCL versions earlier than v2.2 there is no support for invoking global
constructors. However, an easy workaround is to manually enqueue the
constructor initialization kernel that has the following name scheme
``_GLOBAL__sub_I_<compiled file name>``.
This kernel is only present if there are global objects with non-trivial
constructors present in the compiled binary. One way to check this is by 
passing ``CL_PROGRAM_KERNEL_NAMES`` to ``clGetProgramInfo`` (OpenCL v2.0
s5.8.7) and then checking whether any kernel name matches the naming scheme of
global constructor initialization kernel above.

Note that if multiple files are compiled and linked into libraries, multiple
kernels that initialize global objects for multiple modules would have to be
invoked.

Applications are currently required to run initialization of global objects
manually before running any kernels in which the objects are used.

   .. code-block:: console

     clang -cl-std=clc++ test.cl

If there are any global objects to be initialized, the final binary will
contain the ``_GLOBAL__sub_I_test.cl`` kernel to be enqueued.

Note that the manual workaround only applies to objects declared at the
program scope. There is no manual workaround for the construction of static
objects with non-trivial constructors inside functions.

Global destructors can not be invoked manually in the OpenCL v2.0 drivers.
However, all memory used for program scope objects should be released on
``clReleaseProgram``.

Libraries
^^^^^^^^^
Limited experimental support of C++ standard libraries for OpenCL is
described in :doc:`OpenCLSupport` page.

.. _target_features:

Target-Specific Features and Limitations
========================================

CPU Architectures Features and Limitations
------------------------------------------

X86
^^^

The support for X86 (both 32-bit and 64-bit) is considered stable on
Darwin (macOS), Linux, FreeBSD, and Dragonfly BSD: it has been tested
to correctly compile many large C, C++, Objective-C, and Objective-C++
codebases.

On ``x86_64-mingw32``, passing i128(by value) is incompatible with the
Microsoft x64 calling convention. You might need to tweak
``WinX86_64ABIInfo::classify()`` in lib/CodeGen/TargetInfo.cpp.

For the X86 target, clang supports the `-m16` command line
argument which enables 16-bit code output. This is broadly similar to
using ``asm(".code16gcc")`` with the GNU toolchain. The generated code
and the ABI remains 32-bit but the assembler emits instructions
appropriate for a CPU running in 16-bit mode, with address-size and
operand-size prefixes to enable 32-bit addressing and operations.

Several micro-architecture levels as specified by the x86-64 psABI are defined.
They are cumulative in the sense that features from previous levels are
implicitly included in later levels.

- ``-march=x86-64``: CMOV, CMPXCHG8B, FPU, FXSR, MMX, FXSR, SCE, SSE, SSE2
- ``-march=x86-64-v2``: (close to Nehalem) CMPXCHG16B, LAHF-SAHF, POPCNT, SSE3, SSE4.1, SSE4.2, SSSE3
- ``-march=x86-64-v3``: (close to Haswell) AVX, AVX2, BMI1, BMI2, F16C, FMA, LZCNT, MOVBE, XSAVE
- ``-march=x86-64-v4``: AVX512F, AVX512BW, AVX512CD, AVX512DQ, AVX512VL

ARM
^^^

The support for ARM (specifically ARMv6 and ARMv7) is considered stable
on Darwin (iOS): it has been tested to correctly compile many large C,
C++, Objective-C, and Objective-C++ codebases. Clang only supports a
limited number of ARM architectures. It does not yet fully support
ARMv5, for example.

PowerPC
^^^^^^^

The support for PowerPC (especially PowerPC64) is considered stable
on Linux and FreeBSD: it has been tested to correctly compile many
large C and C++ codebases. PowerPC (32bit) is still missing certain
features (e.g. PIC code on ELF platforms).

Other platforms
^^^^^^^^^^^^^^^

clang currently contains some support for other architectures (e.g. Sparc);
however, significant pieces of code generation are still missing, and they
haven't undergone significant testing.

clang contains limited support for the MSP430 embedded processor, but
both the clang support and the LLVM backend support are highly
experimental.

Other platforms are completely unsupported at the moment. Adding the
minimal support needed for parsing and semantic analysis on a new
platform is quite easy; see ``lib/Basic/Targets.cpp`` in the clang source
tree. This level of support is also sufficient for conversion to LLVM IR
for simple programs. Proper support for conversion to LLVM IR requires
adding code to ``lib/CodeGen/CGCall.cpp`` at the moment; this is likely to
change soon, though. Generating assembly requires a suitable LLVM
backend.

Operating System Features and Limitations
-----------------------------------------

Windows
^^^^^^^

Clang has experimental support for targeting "Cygming" (Cygwin / MinGW)
platforms.

See also :ref:`Microsoft Extensions <c_ms>`.

Cygwin
""""""

Clang works on Cygwin-1.7.

MinGW32
"""""""

Clang works on some mingw32 distributions. Clang assumes directories as
below;

-  ``C:/mingw/include``
-  ``C:/mingw/lib``
-  ``C:/mingw/lib/gcc/mingw32/4.[3-5].0/include/c++``

On MSYS, a few tests might fail.

MinGW-w64
"""""""""

For 32-bit (i686-w64-mingw32), and 64-bit (x86\_64-w64-mingw32), Clang
assumes as below;

-  ``GCC versions 4.5.0 to 4.5.3, 4.6.0 to 4.6.2, or 4.7.0 (for the C++ header search path)``
-  ``some_directory/bin/gcc.exe``
-  ``some_directory/bin/clang.exe``
-  ``some_directory/bin/clang++.exe``
-  ``some_directory/bin/../include/c++/GCC_version``
-  ``some_directory/bin/../include/c++/GCC_version/x86_64-w64-mingw32``
-  ``some_directory/bin/../include/c++/GCC_version/i686-w64-mingw32``
-  ``some_directory/bin/../include/c++/GCC_version/backward``
-  ``some_directory/bin/../x86_64-w64-mingw32/include``
-  ``some_directory/bin/../i686-w64-mingw32/include``
-  ``some_directory/bin/../include``

This directory layout is standard for any toolchain you will find on the
official `MinGW-w64 website <http://mingw-w64.sourceforge.net>`_.

Clang expects the GCC executable "gcc.exe" compiled for
``i686-w64-mingw32`` (or ``x86_64-w64-mingw32``) to be present on PATH.

`Some tests might fail <https://bugs.llvm.org/show_bug.cgi?id=9072>`_ on
``x86_64-w64-mingw32``.

.. _clang-cl:

clang-cl
========

clang-cl is an alternative command-line interface to Clang, designed for
compatibility with the Visual C++ compiler, cl.exe.

To enable clang-cl to find system headers, libraries, and the linker when run
from the command-line, it should be executed inside a Visual Studio Native Tools
Command Prompt or a regular Command Prompt where the environment has been set
up using e.g. `vcvarsall.bat <https://msdn.microsoft.com/en-us/library/f2ccy3wt.aspx>`_.

clang-cl can also be used from inside Visual Studio by selecting the LLVM
Platform Toolset. The toolset is not part of the installer, but may be installed
separately from the
`Visual Studio Marketplace <https://marketplace.visualstudio.com/items?itemName=LLVMExtensions.llvm-toolchain>`_.
To use the toolset, select a project in Solution Explorer, open its Property
Page (Alt+F7), and in the "General" section of "Configuration Properties"
change "Platform Toolset" to LLVM.  Doing so enables an additional Property
Page for selecting the clang-cl executable to use for builds.

To use the toolset with MSBuild directly, invoke it with e.g.
``/p:PlatformToolset=LLVM``. This allows trying out the clang-cl toolchain
without modifying your project files.

It's also possible to point MSBuild at clang-cl without changing toolset by
passing ``/p:CLToolPath=c:\llvm\bin /p:CLToolExe=clang-cl.exe``.

When using CMake and the Visual Studio generators, the toolset can be set with the ``-T`` flag:

  ::

    cmake -G"Visual Studio 15 2017" -T LLVM ..

When using CMake with the Ninja generator, set the ``CMAKE_C_COMPILER`` and
``CMAKE_CXX_COMPILER`` variables to clang-cl:

  ::

    cmake -GNinja -DCMAKE_C_COMPILER="c:/Program Files (x86)/LLVM/bin/clang-cl.exe"
        -DCMAKE_CXX_COMPILER="c:/Program Files (x86)/LLVM/bin/clang-cl.exe" ..


Command-Line Options
--------------------

To be compatible with cl.exe, clang-cl supports most of the same command-line
options. Those options can start with either ``/`` or ``-``. It also supports
some of Clang's core options, such as the ``-W`` options.

Options that are known to clang-cl, but not currently supported, are ignored
with a warning. For example:

  ::

    clang-cl.exe: warning: argument unused during compilation: '/AI'

To suppress warnings about unused arguments, use the ``-Qunused-arguments`` option.

Options that are not known to clang-cl will be ignored by default. Use the
``-Werror=unknown-argument`` option in order to treat them as errors. If these
options are spelled with a leading ``/``, they will be mistaken for a filename:

  ::

    clang-cl.exe: error: no such file or directory: '/foobar'

Please `file a bug <https://bugs.llvm.org/enter_bug.cgi?product=clang&component=Driver>`_
for any valid cl.exe flags that clang-cl does not understand.

Execute ``clang-cl /?`` to see a list of supported options:

  ::

    CL.EXE COMPATIBILITY OPTIONS:
      /?                      Display available options
      /arch:<value>           Set architecture for code generation
      /Brepro-                Emit an object file which cannot be reproduced over time
      /Brepro                 Emit an object file which can be reproduced over time
      /clang:<arg>            Pass <arg> to the clang driver
      /C                      Don't discard comments when preprocessing
      /c                      Compile only
      /d1PP                   Retain macro definitions in /E mode
      /d1reportAllClassLayout Dump record layout information
      /diagnostics:caret      Enable caret and column diagnostics (on by default)
      /diagnostics:classic    Disable column and caret diagnostics
      /diagnostics:column     Disable caret diagnostics but keep column info
      /D <macro[=value]>      Define macro
      /EH<value>              Exception handling model
      /EP                     Disable linemarker output and preprocess to stdout
      /execution-charset:<value>
                              Runtime encoding, supports only UTF-8
      /E                      Preprocess to stdout
      /FA                     Output assembly code file during compilation
      /Fa<file or directory>  Output assembly code to this file during compilation (with /FA)
      /Fe<file or directory>  Set output executable file or directory (ends in / or \)
      /FI <value>             Include file before parsing
      /Fi<file>               Set preprocess output file name (with /P)
      /Fo<file or directory>  Set output object file, or directory (ends in / or \) (with /c)
      /fp:except-
      /fp:except
      /fp:fast
      /fp:precise
      /fp:strict
      /Fp<filename>           Set pch filename (with /Yc and /Yu)
      /GA                     Assume thread-local variables are defined in the executable
      /Gd                     Set __cdecl as a default calling convention
      /GF-                    Disable string pooling
      /GF                     Enable string pooling (default)
      /GR-                    Disable emission of RTTI data
      /Gregcall               Set __regcall as a default calling convention
      /GR                     Enable emission of RTTI data
      /Gr                     Set __fastcall as a default calling convention
      /GS-                    Disable buffer security check
      /GS                     Enable buffer security check (default)
      /Gs                     Use stack probes (default)
      /Gs<value>              Set stack probe size (default 4096)
      /guard:<value>          Enable Control Flow Guard with /guard:cf,
                              or only the table with /guard:cf,nochecks.
                              Enable EH Continuation Guard with /guard:ehcont
      /Gv                     Set __vectorcall as a default calling convention
      /Gw-                    Don't put each data item in its own section
      /Gw                     Put each data item in its own section
      /GX-                    Disable exception handling
      /GX                     Enable exception handling
      /Gy-                    Don't put each function in its own section (default)
      /Gy                     Put each function in its own section
      /Gz                     Set __stdcall as a default calling convention
      /help                   Display available options
      /imsvc <dir>            Add directory to system include search path, as if part of %INCLUDE%
      /I <dir>                Add directory to include search path
      /J                      Make char type unsigned
      /LDd                    Create debug DLL
      /LD                     Create DLL
      /link <options>         Forward options to the linker
      /MDd                    Use DLL debug run-time
      /MD                     Use DLL run-time
      /MTd                    Use static debug run-time
      /MT                     Use static run-time
      /O0                     Disable optimization
      /O1                     Optimize for size  (same as /Og     /Os /Oy /Ob2 /GF /Gy)
      /O2                     Optimize for speed (same as /Og /Oi /Ot /Oy /Ob2 /GF /Gy)
      /Ob0                    Disable function inlining
      /Ob1                    Only inline functions which are (explicitly or implicitly) marked inline
      /Ob2                    Inline functions as deemed beneficial by the compiler
      /Od                     Disable optimization
      /Og                     No effect
      /Oi-                    Disable use of builtin functions
      /Oi                     Enable use of builtin functions
      /Os                     Optimize for size
      /Ot                     Optimize for speed
      /Ox                     Deprecated (same as /Og /Oi /Ot /Oy /Ob2); use /O2 instead
      /Oy-                    Disable frame pointer omission (x86 only, default)
      /Oy                     Enable frame pointer omission (x86 only)
      /O<flags>               Set multiple /O flags at once; e.g. '/O2y-' for '/O2 /Oy-'
      /o <file or directory>  Set output file or directory (ends in / or \)
      /P                      Preprocess to file
      /Qvec-                  Disable the loop vectorization passes
      /Qvec                   Enable the loop vectorization passes
      /showFilenames-         Don't print the name of each compiled file (default)
      /showFilenames          Print the name of each compiled file
      /showIncludes           Print info about included files to stderr
      /source-charset:<value> Source encoding, supports only UTF-8
      /std:<value>            Language standard to compile for
      /TC                     Treat all source files as C
      /Tc <filename>          Specify a C source file
      /TP                     Treat all source files as C++
      /Tp <filename>          Specify a C++ source file
      /utf-8                  Set source and runtime encoding to UTF-8 (default)
      /U <macro>              Undefine macro
      /vd<value>              Control vtordisp placement
      /vmb                    Use a best-case representation method for member pointers
      /vmg                    Use a most-general representation for member pointers
      /vmm                    Set the default most-general representation to multiple inheritance
      /vms                    Set the default most-general representation to single inheritance
      /vmv                    Set the default most-general representation to virtual inheritance
      /volatile:iso           Volatile loads and stores have standard semantics
      /volatile:ms            Volatile loads and stores have acquire and release semantics
      /W0                     Disable all warnings
      /W1                     Enable -Wall
      /W2                     Enable -Wall
      /W3                     Enable -Wall
      /W4                     Enable -Wall and -Wextra
      /Wall                   Enable -Weverything
      /WX-                    Do not treat warnings as errors
      /WX                     Treat warnings as errors
      /w                      Disable all warnings
      /X                      Don't add %INCLUDE% to the include search path
      /Y-                     Disable precompiled headers, overrides /Yc and /Yu
      /Yc<filename>           Generate a pch file for all code up to and including <filename>
      /Yu<filename>           Load a pch file and use it instead of all code up to and including <filename>
      /Z7                     Enable CodeView debug information in object files
      /Zc:char8_t             Enable C++2a char8_t type
      /Zc:char8_t-            Disable C++2a char8_t type
      /Zc:dllexportInlines-   Don't dllexport/dllimport inline member functions of dllexport/import classes
      /Zc:dllexportInlines    dllexport/dllimport inline member functions of dllexport/import classes (default)
      /Zc:sizedDealloc-       Disable C++14 sized global deallocation functions
      /Zc:sizedDealloc        Enable C++14 sized global deallocation functions
      /Zc:strictStrings       Treat string literals as const
      /Zc:threadSafeInit-     Disable thread-safe initialization of static variables
      /Zc:threadSafeInit      Enable thread-safe initialization of static variables
      /Zc:trigraphs-          Disable trigraphs (default)
      /Zc:trigraphs           Enable trigraphs
      /Zc:twoPhase-           Disable two-phase name lookup in templates
      /Zc:twoPhase            Enable two-phase name lookup in templates
      /Zd                     Emit debug line number tables only
      /Zi                     Alias for /Z7. Does not produce PDBs.
      /Zl                     Don't mention any default libraries in the object file
      /Zp                     Set the default maximum struct packing alignment to 1
      /Zp<value>              Specify the default maximum struct packing alignment
      /Zs                     Syntax-check only

    OPTIONS:
      -###                    Print (but do not run) the commands to run for this compilation
      --analyze               Run the static analyzer
      -faddrsig               Emit an address-significance table
      -fansi-escape-codes     Use ANSI escape codes for diagnostics
      -fblocks                Enable the 'blocks' language feature
      -fcf-protection=<value> Instrument control-flow architecture protection. Options: return, branch, full, none.
      -fcf-protection         Enable cf-protection in 'full' mode
      -fcolor-diagnostics     Use colors in diagnostics
      -fcomplete-member-pointers
                              Require member pointer base types to be complete if they would be significant under the Microsoft ABI
      -fcoverage-mapping      Generate coverage mapping to enable code coverage analysis
      -fcrash-diagnostics-dir=<dir>
                              Put crash-report files in <dir>
      -fdebug-macro           Emit macro debug information
      -fdelayed-template-parsing
                              Parse templated function definitions at the end of the translation unit
      -fdiagnostics-absolute-paths
                              Print absolute paths in diagnostics
      -fdiagnostics-parseable-fixits
                              Print fix-its in machine parseable form
      -flto=<value>           Set LTO mode to either 'full' or 'thin'
      -flto                   Enable LTO in 'full' mode
      -fmerge-all-constants   Allow merging of constants
      -fms-compatibility-version=<value>
                              Dot-separated value representing the Microsoft compiler version
                              number to report in _MSC_VER (0 = don't define it (default))
      -fms-compatibility      Enable full Microsoft Visual C++ compatibility
      -fms-extensions         Accept some non-standard constructs supported by the Microsoft compiler
      -fmsc-version=<value>   Microsoft compiler version number to report in _MSC_VER
                              (0 = don't define it (default))
      -fno-addrsig            Don't emit an address-significance table
      -fno-builtin-<value>    Disable implicit builtin knowledge of a specific function
      -fno-builtin            Disable implicit builtin knowledge of functions
      -fno-complete-member-pointers
                              Do not require member pointer base types to be complete if they would be significant under the Microsoft ABI
      -fno-coverage-mapping   Disable code coverage analysis
      -fno-crash-diagnostics  Disable auto-generation of preprocessed source files and a script for reproduction during a clang crash
      -fno-debug-macro        Do not emit macro debug information
      -fno-delayed-template-parsing
                              Disable delayed template parsing
      -fno-sanitize-address-poison-custom-array-cookie
                              Disable poisoning array cookies when using custom operator new[] in AddressSanitizer
      -fno-sanitize-address-use-after-scope
                              Disable use-after-scope detection in AddressSanitizer
      -fno-sanitize-address-use-odr-indicator
                               Disable ODR indicator globals
      -fno-sanitize-ignorelist Don't use ignorelist file for sanitizers
      -fno-sanitize-cfi-cross-dso
                              Disable control flow integrity (CFI) checks for cross-DSO calls.
      -fno-sanitize-coverage=<value>
                              Disable specified features of coverage instrumentation for Sanitizers
      -fno-sanitize-memory-track-origins
                              Disable origins tracking in MemorySanitizer
      -fno-sanitize-memory-use-after-dtor
                              Disable use-after-destroy detection in MemorySanitizer
      -fno-sanitize-recover=<value>
                              Disable recovery for specified sanitizers
      -fno-sanitize-stats     Disable sanitizer statistics gathering.
      -fno-sanitize-thread-atomics
                              Disable atomic operations instrumentation in ThreadSanitizer
      -fno-sanitize-thread-func-entry-exit
                              Disable function entry/exit instrumentation in ThreadSanitizer
      -fno-sanitize-thread-memory-access
                              Disable memory access instrumentation in ThreadSanitizer
      -fno-sanitize-trap=<value>
                              Disable trapping for specified sanitizers
      -fno-standalone-debug   Limit debug information produced to reduce size of debug binary
      -fobjc-runtime=<value>  Specify the target Objective-C runtime kind and version
      -fprofile-exclude-files=<value>
                              Instrument only functions from files where names don't match all the regexes separated by a semi-colon
      -fprofile-filter-files=<value>
                              Instrument only functions from files where names match any regex separated by a semi-colon
      -fprofile-instr-generate=<file>
                              Generate instrumented code to collect execution counts into <file>
                              (overridden by LLVM_PROFILE_FILE env var)
      -fprofile-instr-generate
                              Generate instrumented code to collect execution counts into default.profraw file
                              (overridden by '=' form of option or LLVM_PROFILE_FILE env var)
      -fprofile-instr-use=<value>
                              Use instrumentation data for profile-guided optimization
      -fprofile-remapping-file=<file>
                              Use the remappings described in <file> to match the profile data against names in the program
      -fprofile-list=<file>
                              Filename defining the list of functions/files to instrument
      -fsanitize-address-field-padding=<value>
                              Level of field padding for AddressSanitizer
      -fsanitize-address-globals-dead-stripping
                              Enable linker dead stripping of globals in AddressSanitizer
      -fsanitize-address-poison-custom-array-cookie
                              Enable poisoning array cookies when using custom operator new[] in AddressSanitizer
      -fsanitize-address-use-after-return=<mode>
                              Select the mode of detecting stack use-after-return in AddressSanitizer: never | runtime (default) | always
      -fsanitize-address-use-after-scope
                              Enable use-after-scope detection in AddressSanitizer
      -fsanitize-address-use-odr-indicator
                              Enable ODR indicator globals to avoid false ODR violation reports in partially sanitized programs at the cost of an increase in binary size
      -fsanitize-ignorelist=<value>
                              Path to ignorelist file for sanitizers
      -fsanitize-cfi-cross-dso
                              Enable control flow integrity (CFI) checks for cross-DSO calls.
      -fsanitize-cfi-icall-generalize-pointers
                              Generalize pointers in CFI indirect call type signature checks
      -fsanitize-coverage=<value>
                              Specify the type of coverage instrumentation for Sanitizers
      -fsanitize-hwaddress-abi=<value>
                              Select the HWAddressSanitizer ABI to target (interceptor or platform, default interceptor)
      -fsanitize-memory-track-origins=<value>
                              Enable origins tracking in MemorySanitizer
      -fsanitize-memory-track-origins
                              Enable origins tracking in MemorySanitizer
      -fsanitize-memory-use-after-dtor
                              Enable use-after-destroy detection in MemorySanitizer
      -fsanitize-recover=<value>
                              Enable recovery for specified sanitizers
      -fsanitize-stats        Enable sanitizer statistics gathering.
      -fsanitize-thread-atomics
                              Enable atomic operations instrumentation in ThreadSanitizer (default)
      -fsanitize-thread-func-entry-exit
                              Enable function entry/exit instrumentation in ThreadSanitizer (default)
      -fsanitize-thread-memory-access
                              Enable memory access instrumentation in ThreadSanitizer (default)
      -fsanitize-trap=<value> Enable trapping for specified sanitizers
      -fsanitize-undefined-strip-path-components=<number>
                              Strip (or keep only, if negative) a given number of path components when emitting check metadata.
      -fsanitize=<check>      Turn on runtime checks for various forms of undefined or suspicious
                              behavior. See user manual for available checks
      -fsplit-lto-unit        Enables splitting of the LTO unit.
      -fstandalone-debug      Emit full debug info for all types used by the program
      -fwhole-program-vtables Enables whole-program vtable optimization. Requires -flto
      -gcodeview-ghash        Emit type record hashes in a .debug$H section
      -gcodeview              Generate CodeView debug information
      -gline-directives-only  Emit debug line info directives only
      -gline-tables-only      Emit debug line number tables only
      -miamcu                 Use Intel MCU ABI
      -mllvm <value>          Additional arguments to forward to LLVM's option processing
      -nobuiltininc           Disable builtin #include directories
      -Qunused-arguments      Don't emit warning for unused driver arguments
      -R<remark>              Enable the specified remark
      --target=<value>        Generate code for the given target
      --version               Print version information
      -v                      Show commands to run and use verbose output
      -W<warning>             Enable the specified warning
      -Xclang <arg>           Pass <arg> to the clang compiler

The /clang: Option
^^^^^^^^^^^^^^^^^^

When clang-cl is run with a set of ``/clang:<arg>`` options, it will gather all
of the ``<arg>`` arguments and process them as if they were passed to the clang
driver. This mechanism allows you to pass flags that are not exposed in the
clang-cl options or flags that have a different meaning when passed to the clang
driver. Regardless of where they appear in the command line, the ``/clang:``
arguments are treated as if they were passed at the end of the clang-cl command
line.

The /Zc:dllexportInlines- Option
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This causes the class-level `dllexport` and `dllimport` attributes to not apply
to inline member functions, as they otherwise would. For example, in the code
below `S::foo()` would normally be defined and exported by the DLL, but when
using the ``/Zc:dllexportInlines-`` flag it is not:

.. code-block:: c

  struct __declspec(dllexport) S {
    void foo() {}
  }

This has the benefit that the compiler doesn't need to emit a definition of
`S::foo()` in every translation unit where the declaration is included, as it
would otherwise do to ensure there's a definition in the DLL even if it's not
used there. If the declaration occurs in a header file that's widely used, this
can save significant compilation time and output size. It also reduces the
number of functions exported by the DLL similarly to what
``-fvisibility-inlines-hidden`` does for shared objects on ELF and Mach-O.
Since the function declaration comes with an inline definition, users of the
library can use that definition directly instead of importing it from the DLL.

Note that the Microsoft Visual C++ compiler does not support this option, and
if code in a DLL is compiled with ``/Zc:dllexportInlines-``, the code using the
DLL must be compiled in the same way so that it doesn't attempt to dllimport
the inline member functions. The reverse scenario should generally work though:
a DLL compiled without this flag (such as a system library compiled with Visual
C++) can be referenced from code compiled using the flag, meaning that the
referencing code will use the inline definitions instead of importing them from
the DLL.

Also note that like when using ``-fvisibility-inlines-hidden``, the address of
`S::foo()` will be different inside and outside the DLL, breaking the C/C++
standard requirement that functions have a unique address.

The flag does not apply to explicit class template instantiation definitions or
declarations, as those are typically used to explicitly provide a single
definition in a DLL, (dllexported instantiation definition) or to signal that
the definition is available elsewhere (dllimport instantiation declaration). It
also doesn't apply to inline members with static local variables, to ensure
that the same instance of the variable is used inside and outside the DLL.

Using this flag can cause problems when inline functions that would otherwise
be dllexported refer to internal symbols of a DLL. For example:

.. code-block:: c

  void internal();

  struct __declspec(dllimport) S {
    void foo() { internal(); }
  }

Normally, references to `S::foo()` would use the definition in the DLL from
which it was exported, and which presumably also has the definition of
`internal()`. However, when using ``/Zc:dllexportInlines-``, the inline
definition of `S::foo()` is used directly, resulting in a link error since
`internal()` is not available. Even worse, if there is an inline definition of
`internal()` containing a static local variable, we will now refer to a
different instance of that variable than in the DLL:

.. code-block:: c

  inline int internal() { static int x; return x++; }

  struct __declspec(dllimport) S {
    int foo() { return internal(); }
  }

This could lead to very subtle bugs. Using ``-fvisibility-inlines-hidden`` can
lead to the same issue. To avoid it in this case, make `S::foo()` or
`internal()` non-inline, or mark them `dllimport/dllexport` explicitly.

Finding Clang runtime libraries
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

clang-cl supports several features that require runtime library support:

- Address Sanitizer (ASan): ``-fsanitize=address``
- Undefined Behavior Sanitizer (UBSan): ``-fsanitize=undefined``
- Code coverage: ``-fprofile-instr-generate -fcoverage-mapping``
- Profile Guided Optimization (PGO): ``-fprofile-instr-generate``
- Certain math operations (int128 division) require the builtins library

In order to use these features, the user must link the right runtime libraries
into their program. These libraries are distributed alongside Clang in the
library resource directory. Clang searches for the resource directory by
searching relative to the Clang executable. For example, if LLVM is installed
in ``C:\Program Files\LLVM``, then the profile runtime library will be located
at the path
``C:\Program Files\LLVM\lib\clang\11.0.0\lib\windows\clang_rt.profile-x86_64.lib``.

For UBSan, PGO, and coverage, Clang will emit object files that auto-link the
appropriate runtime library, but the user generally needs to help the linker
(whether it is ``lld-link.exe`` or MSVC ``link.exe``) find the library resource
directory. Using the example installation above, this would mean passing
``/LIBPATH:C:\Program Files\LLVM\lib\clang\11.0.0\lib\windows`` to the linker.
If the user links the program with the ``clang`` or ``clang-cl`` drivers, the
driver will pass this flag for them.

If the linker cannot find the appropriate library, it will emit an error like
this::

  $ clang-cl -c -fsanitize=undefined t.cpp

  $ lld-link t.obj -dll
  lld-link: error: could not open 'clang_rt.ubsan_standalone-x86_64.lib': no such file or directory
  lld-link: error: could not open 'clang_rt.ubsan_standalone_cxx-x86_64.lib': no such file or directory

  $ link t.obj -dll -nologo
  LINK : fatal error LNK1104: cannot open file 'clang_rt.ubsan_standalone-x86_64.lib'

To fix the error, add the appropriate ``/libpath:`` flag to the link line.

For ASan, as of this writing, the user is also responsible for linking against
the correct ASan libraries.

If the user is using the dynamic CRT (``/MD``), then they should add
``clang_rt.asan_dynamic-x86_64.lib`` to the link line as a regular input. For
other architectures, replace x86_64 with the appropriate name here and below.

If the user is using the static CRT (``/MT``), then different runtimes are used
to produce DLLs and EXEs. To link a DLL, pass
``clang_rt.asan_dll_thunk-x86_64.lib``. To link an EXE, pass
``-wholearchive:clang_rt.asan-x86_64.lib``.

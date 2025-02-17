.. cmake-manual-description: CMake Generator Expressions

cmake-generator-expressions(7)
******************************

.. only:: html

   .. contents::

Introduction
============

Generator expressions are evaluated during build system generation to produce
information specific to each build configuration.

Generator expressions are allowed in the context of many target properties,
such as :prop_tgt:`LINK_LIBRARIES`, :prop_tgt:`INCLUDE_DIRECTORIES`,
:prop_tgt:`COMPILE_DEFINITIONS` and others.  They may also be used when using
commands to populate those properties, such as :command:`target_link_libraries`,
:command:`target_include_directories`, :command:`target_compile_definitions`
and others.

They enable conditional linking, conditional definitions used when compiling,
conditional include directories, and more.  The conditions may be based on
the build configuration, target properties, platform information or any other
queryable information.

Generator expressions have the form ``$<...>``.  To avoid confusion, this page
deviates from most of the CMake documentation in that it omits angular brackets
``<...>`` around placeholders like ``condition``, ``string``, ``target``,
among others.

Generator expressions can be nested, as shown in most of the examples below.

.. _`Boolean Generator Expressions`:

Boolean Generator Expressions
=============================

Boolean expressions evaluate to either ``0`` or ``1``.
They are typically used to construct the condition in a :ref:`conditional
generator expression<Conditional Generator Expressions>`.

Available boolean expressions are:

Logical Operators
-----------------

.. genex:: $<BOOL:string>

  Converts ``string`` to ``0`` or ``1``. Evaluates to ``0`` if any of the
  following is true:

  * ``string`` is empty,
  * ``string`` is a case-insensitive equal of
    ``0``, ``FALSE``, ``OFF``, ``N``, ``NO``, ``IGNORE``, or ``NOTFOUND``, or
  * ``string`` ends in the suffix ``-NOTFOUND`` (case-sensitive).

  Otherwise evaluates to ``1``.

.. genex:: $<AND:conditions>

  where ``conditions`` is a comma-separated list of boolean expressions.
  Evaluates to ``1`` if all conditions are ``1``.
  Otherwise evaluates to ``0``.

.. genex:: $<OR:conditions>

  where ``conditions`` is a comma-separated list of boolean expressions.
  Evaluates to ``1`` if at least one of the conditions is ``1``.
  Otherwise evaluates to ``0``.

.. genex:: $<NOT:condition>

  ``0`` if ``condition`` is ``1``, else ``1``.

String Comparisons
------------------

.. genex:: $<STREQUAL:string1,string2>

  ``1`` if ``string1`` and ``string2`` are equal, else ``0``.
  The comparison is case-sensitive.  For a case-insensitive comparison,
  combine with a :ref:`string transforming generator expression
  <String Transforming Generator Expressions>`,

  .. code-block:: cmake

    $<STREQUAL:$<UPPER_CASE:${foo}>,"BAR"> # "1" if ${foo} is any of "BAR", "Bar", "bar", ...

.. genex:: $<EQUAL:value1,value2>

  ``1`` if ``value1`` and ``value2`` are numerically equal, else ``0``.

.. genex:: $<IN_LIST:string,list>

  .. versionadded:: 3.12

  ``1`` if ``string`` is member of the semicolon-separated ``list``, else ``0``.
  Uses case-sensitive comparisons.

.. genex:: $<VERSION_LESS:v1,v2>

  ``1`` if ``v1`` is a version less than ``v2``, else ``0``.

.. genex:: $<VERSION_GREATER:v1,v2>

  ``1`` if ``v1`` is a version greater than ``v2``, else ``0``.

.. genex:: $<VERSION_EQUAL:v1,v2>

  ``1`` if ``v1`` is the same version as ``v2``, else ``0``.

.. genex:: $<VERSION_LESS_EQUAL:v1,v2>

  .. versionadded:: 3.7

  ``1`` if ``v1`` is a version less than or equal to ``v2``, else ``0``.

.. genex:: $<VERSION_GREATER_EQUAL:v1,v2>

  .. versionadded:: 3.7

  ``1`` if ``v1`` is a version greater than or equal to ``v2``, else ``0``.

Path Comparisons
----------------

.. genex:: $<PATH_EQUAL:path1,path2>

  .. versionadded:: 3.24

  Compares the lexical representations of two paths. No normalization is
  performed on either path. Returns ``1`` if the paths are equal, ``0``
  otherwise.

  See :ref:`cmake_path(COMPARE) <Path COMPARE>` for more details.

.. _GenEx Path Queries:

Path Queries
------------

The ``$<PATH>`` generator expression offers the same capabilities as the
:command:`cmake_path` command, for the :ref:`Query <Path Query>` options.

For all ``$<PATH>`` generator expressions, paths are expected in cmake-style
format. The :ref:`$\<PATH:CMAKE_PATH\> <GenEx PATH-CMAKE_PATH>` generator
expression can be used to convert a native path to a cmake-style one.

The ``$<PATH>`` generator expression can also be used for path
:ref:`Decomposition <GenEx Path Decomposition>` and
:ref:`Transformations <GenEx Path Transformations>`.

.. genex:: $<PATH:HAS_*,path>

  .. versionadded:: 3.24

  The following operations return ``1`` if the particular path component is
  present, ``0`` otherwise. See :ref:`Path Structure And Terminology` for the
  meaning of each path component.

  ::

    $<PATH:HAS_ROOT_NAME,path>
    $<PATH:HAS_ROOT_DIRECTORY,path>
    $<PATH:HAS_ROOT_PATH,path>
    $<PATH:HAS_FILENAME,path>
    $<PATH:HAS_EXTENSION,path>
    $<PATH:HAS_STEM,path>
    $<PATH:HAS_RELATIVE_PART,path>
    $<PATH:HAS_PARENT_PATH,path>

  Note the following special cases:

  * For ``HAS_ROOT_PATH``, a true result will only be returned if at least one
    of ``root-name`` or ``root-directory`` is non-empty.

  * For ``HAS_PARENT_PATH``, the root directory is also considered to have a
    parent, which will be itself.  The result is true except if the path
    consists of just a :ref:`filename <FILENAME_DEF>`.

.. genex:: $<PATH:IS_ABSOLUTE,path>

  .. versionadded:: 3.24

  Returns ``1`` if the path is :ref:`absolute <IS_ABSOLUTE>`, ``0`` otherwise.

.. genex:: $<PATH:IS_RELATIVE,path>

  .. versionadded:: 3.24

  This will return the opposite of ``IS_ABSOLUTE``.

.. genex:: $<PATH:IS_PREFIX[,NORMALIZE],path,input>

  .. versionadded:: 3.24

  Returns ``1`` if ``path`` is the prefix of ``input``,``0`` otherwise.

  When the ``NORMALIZE`` option is specified, ``path`` and ``input`` are
  :ref:`normalized <Normalization>` before the check.

Variable Queries
----------------

.. genex:: $<TARGET_EXISTS:target>

  .. versionadded:: 3.12

  ``1`` if ``target`` exists, else ``0``.

.. genex:: $<CONFIG:cfgs>

  ``1`` if config is any one of the entries in comma-separated list
  ``cfgs``, else ``0``. This is a case-insensitive comparison. The mapping in
  :prop_tgt:`MAP_IMPORTED_CONFIG_<CONFIG>` is also considered by this
  expression when it is evaluated on a property on an :prop_tgt:`IMPORTED`
  target.

.. genex:: $<PLATFORM_ID:platform_ids>

  where ``platform_ids`` is a comma-separated list.
  ``1`` if the CMake's platform id matches any one of the entries in
  ``platform_ids``, otherwise ``0``.
  See also the :variable:`CMAKE_SYSTEM_NAME` variable.

.. genex:: $<C_COMPILER_ID:compiler_ids>

  where ``compiler_ids`` is a comma-separated list.
  ``1`` if the CMake's compiler id of the C compiler matches any one
  of the entries in ``compiler_ids``, otherwise ``0``.
  See also the :variable:`CMAKE_<LANG>_COMPILER_ID` variable.

.. genex:: $<CXX_COMPILER_ID:compiler_ids>

  where ``compiler_ids`` is a comma-separated list.
  ``1`` if the CMake's compiler id of the CXX compiler matches any one
  of the entries in ``compiler_ids``, otherwise ``0``.
  See also the :variable:`CMAKE_<LANG>_COMPILER_ID` variable.

.. genex:: $<CUDA_COMPILER_ID:compiler_ids>

  .. versionadded:: 3.15

  where ``compiler_ids`` is a comma-separated list.
  ``1`` if the CMake's compiler id of the CUDA compiler matches any one
  of the entries in ``compiler_ids``, otherwise ``0``.
  See also the :variable:`CMAKE_<LANG>_COMPILER_ID` variable.

.. genex:: $<OBJC_COMPILER_ID:compiler_ids>

  .. versionadded:: 3.16

  where ``compiler_ids`` is a comma-separated list.
  ``1`` if the CMake's compiler id of the Objective-C compiler matches any one
  of the entries in ``compiler_ids``, otherwise ``0``.
  See also the :variable:`CMAKE_<LANG>_COMPILER_ID` variable.

.. genex:: $<OBJCXX_COMPILER_ID:compiler_ids>

  .. versionadded:: 3.16

  where ``compiler_ids`` is a comma-separated list.
  ``1`` if the CMake's compiler id of the Objective-C++ compiler matches any one
  of the entries in ``compiler_ids``, otherwise ``0``.
  See also the :variable:`CMAKE_<LANG>_COMPILER_ID` variable.

.. genex:: $<Fortran_COMPILER_ID:compiler_ids>

  where ``compiler_ids`` is a comma-separated list.
  ``1`` if the CMake's compiler id of the Fortran compiler matches any one
  of the entries in ``compiler_ids``, otherwise ``0``.
  See also the :variable:`CMAKE_<LANG>_COMPILER_ID` variable.

.. genex:: $<HIP_COMPILER_ID:compiler_ids>

  .. versionadded:: 3.21

  where ``compiler_ids`` is a comma-separated list.
  ``1`` if the CMake's compiler id of the HIP compiler matches any one
  of the entries in ``compiler_ids``, otherwise ``0``.
  See also the :variable:`CMAKE_<LANG>_COMPILER_ID` variable.

.. genex:: $<ISPC_COMPILER_ID:compiler_ids>

  .. versionadded:: 3.19

  where ``compiler_ids`` is a comma-separated list.
  ``1`` if the CMake's compiler id of the ISPC compiler matches any one
  of the entries in ``compiler_ids``, otherwise ``0``.
  See also the :variable:`CMAKE_<LANG>_COMPILER_ID` variable.

.. genex:: $<C_COMPILER_VERSION:version>

  ``1`` if the version of the C compiler matches ``version``, otherwise ``0``.
  See also the :variable:`CMAKE_<LANG>_COMPILER_VERSION` variable.

.. genex:: $<CXX_COMPILER_VERSION:version>

  ``1`` if the version of the CXX compiler matches ``version``, otherwise ``0``.
  See also the :variable:`CMAKE_<LANG>_COMPILER_VERSION` variable.

.. genex:: $<CUDA_COMPILER_VERSION:version>

  .. versionadded:: 3.15

  ``1`` if the version of the CXX compiler matches ``version``, otherwise ``0``.
  See also the :variable:`CMAKE_<LANG>_COMPILER_VERSION` variable.

.. genex:: $<OBJC_COMPILER_VERSION:version>

  .. versionadded:: 3.16

  ``1`` if the version of the OBJC compiler matches ``version``, otherwise ``0``.
  See also the :variable:`CMAKE_<LANG>_COMPILER_VERSION` variable.

.. genex:: $<OBJCXX_COMPILER_VERSION:version>

  .. versionadded:: 3.16

  ``1`` if the version of the OBJCXX compiler matches ``version``, otherwise ``0``.
  See also the :variable:`CMAKE_<LANG>_COMPILER_VERSION` variable.

.. genex:: $<Fortran_COMPILER_VERSION:version>

  ``1`` if the version of the Fortran compiler matches ``version``, otherwise ``0``.
  See also the :variable:`CMAKE_<LANG>_COMPILER_VERSION` variable.

.. genex:: $<HIP_COMPILER_VERSION:version>

  .. versionadded:: 3.21

  ``1`` if the version of the HIP compiler matches ``version``, otherwise ``0``.
  See also the :variable:`CMAKE_<LANG>_COMPILER_VERSION` variable.

.. genex:: $<ISPC_COMPILER_VERSION:version>

  .. versionadded:: 3.19

  ``1`` if the version of the ISPC compiler matches ``version``, otherwise ``0``.
  See also the :variable:`CMAKE_<LANG>_COMPILER_VERSION` variable.

.. genex:: $<TARGET_POLICY:policy>

  ``1`` if the ``policy`` was NEW when the 'head' target was created,
  else ``0``.  If the ``policy`` was not set, the warning message for the policy
  will be emitted. This generator expression only works for a subset of
  policies.

.. genex:: $<COMPILE_FEATURES:features>

  .. versionadded:: 3.1

  where ``features`` is a comma-spearated list.
  Evaluates to ``1`` if all of the ``features`` are available for the 'head'
  target, and ``0`` otherwise. If this expression is used while evaluating
  the link implementation of a target and if any dependency transitively
  increases the required :prop_tgt:`C_STANDARD` or :prop_tgt:`CXX_STANDARD`
  for the 'head' target, an error is reported.  See the
  :manual:`cmake-compile-features(7)` manual for information on
  compile features and a list of supported compilers.

.. _`Boolean COMPILE_LANGUAGE Generator Expression`:

.. genex:: $<COMPILE_LANG_AND_ID:language,compiler_ids>

  .. versionadded:: 3.15

  ``1`` when the language used for compilation unit matches ``language`` and
  the CMake's compiler id of the language compiler matches any one of the
  entries in ``compiler_ids``, otherwise ``0``. This expression is a short form
  for the combination of ``$<COMPILE_LANGUAGE:language>`` and
  ``$<LANG_COMPILER_ID:compiler_ids>``. This expression may be used to specify
  compile options, compile definitions, and include directories for source files of a
  particular language and compiler combination in a target. For example:

  .. code-block:: cmake

    add_executable(myapp main.cpp foo.c bar.cpp zot.cu)
    target_compile_definitions(myapp
      PRIVATE $<$<COMPILE_LANG_AND_ID:CXX,AppleClang,Clang>:COMPILING_CXX_WITH_CLANG>
              $<$<COMPILE_LANG_AND_ID:CXX,Intel>:COMPILING_CXX_WITH_INTEL>
              $<$<COMPILE_LANG_AND_ID:C,Clang>:COMPILING_C_WITH_CLANG>
    )

  This specifies the use of different compile definitions based on both
  the compiler id and compilation language. This example will have a
  ``COMPILING_CXX_WITH_CLANG`` compile definition when Clang is the CXX
  compiler, and ``COMPILING_CXX_WITH_INTEL`` when Intel is the CXX compiler.
  Likewise when the C compiler is Clang it will only see the  ``COMPILING_C_WITH_CLANG``
  definition.

  Without the ``COMPILE_LANG_AND_ID`` generator expression the same logic
  would be expressed as:

  .. code-block:: cmake

    target_compile_definitions(myapp
      PRIVATE $<$<AND:$<COMPILE_LANGUAGE:CXX>,$<CXX_COMPILER_ID:AppleClang,Clang>>:COMPILING_CXX_WITH_CLANG>
              $<$<AND:$<COMPILE_LANGUAGE:CXX>,$<CXX_COMPILER_ID:Intel>>:COMPILING_CXX_WITH_INTEL>
              $<$<AND:$<COMPILE_LANGUAGE:C>,$<C_COMPILER_ID:Clang>>:COMPILING_C_WITH_CLANG>
    )

.. genex:: $<COMPILE_LANGUAGE:languages>

  .. versionadded:: 3.3

  ``1`` when the language used for compilation unit matches any of the entries
  in ``languages``, otherwise ``0``.  This expression may be used to specify
  compile options, compile definitions, and include directories for source files of a
  particular language in a target. For example:

  .. code-block:: cmake

    add_executable(myapp main.cpp foo.c bar.cpp zot.cu)
    target_compile_options(myapp
      PRIVATE $<$<COMPILE_LANGUAGE:CXX>:-fno-exceptions>
    )
    target_compile_definitions(myapp
      PRIVATE $<$<COMPILE_LANGUAGE:CXX>:COMPILING_CXX>
              $<$<COMPILE_LANGUAGE:CUDA>:COMPILING_CUDA>
    )
    target_include_directories(myapp
      PRIVATE $<$<COMPILE_LANGUAGE:CXX,CUDA>:/opt/foo/headers>
    )

  This specifies the use of the ``-fno-exceptions`` compile option,
  ``COMPILING_CXX`` compile definition, and ``cxx_headers`` include
  directory for C++ only (compiler id checks elided).  It also specifies
  a ``COMPILING_CUDA`` compile definition for CUDA.

  Note that with :ref:`Visual Studio Generators` and :generator:`Xcode` there
  is no way to represent target-wide compile definitions or include directories
  separately for ``C`` and ``CXX`` languages.
  Also, with :ref:`Visual Studio Generators` there is no way to represent
  target-wide flags separately for ``C`` and ``CXX`` languages.  Under these
  generators, expressions for both C and C++ sources will be evaluated
  using ``CXX`` if there are any C++ sources and otherwise using ``C``.
  A workaround is to create separate libraries for each source file language
  instead:

  .. code-block:: cmake

    add_library(myapp_c foo.c)
    add_library(myapp_cxx bar.cpp)
    target_compile_options(myapp_cxx PUBLIC -fno-exceptions)
    add_executable(myapp main.cpp)
    target_link_libraries(myapp myapp_c myapp_cxx)

.. _`Boolean LINK_LANGUAGE Generator Expression`:

.. genex:: $<LINK_LANG_AND_ID:language,compiler_ids>

  .. versionadded:: 3.18

  ``1`` when the language used for link step matches ``language`` and the
  CMake's compiler id of the language linker matches any one of the entries
  in ``compiler_ids``, otherwise ``0``. This expression is a short form for the
  combination of ``$<LINK_LANGUAGE:language>`` and
  ``$<LANG_COMPILER_ID:compiler_ids>``. This expression may be used to specify
  link libraries, link options, link directories and link dependencies of a
  particular language and linker combination in a target. For example:

  .. code-block:: cmake

    add_library(libC_Clang ...)
    add_library(libCXX_Clang ...)
    add_library(libC_Intel ...)
    add_library(libCXX_Intel ...)

    add_executable(myapp main.c)
    if (CXX_CONFIG)
      target_sources(myapp PRIVATE file.cxx)
    endif()
    target_link_libraries(myapp
      PRIVATE $<$<LINK_LANG_AND_ID:CXX,Clang,AppleClang>:libCXX_Clang>
              $<$<LINK_LANG_AND_ID:C,Clang,AppleClang>:libC_Clang>
              $<$<LINK_LANG_AND_ID:CXX,Intel>:libCXX_Intel>
              $<$<LINK_LANG_AND_ID:C,Intel>:libC_Intel>)

  This specifies the use of different link libraries based on both the
  compiler id and link language. This example will have target ``libCXX_Clang``
  as link dependency when ``Clang`` or ``AppleClang`` is the ``CXX``
  linker, and ``libCXX_Intel`` when ``Intel`` is the ``CXX`` linker.
  Likewise when the ``C`` linker is ``Clang`` or ``AppleClang``, target
  ``libC_Clang`` will be added as link dependency and ``libC_Intel`` when
  ``Intel`` is the ``C`` linker.

  See :ref:`the note related to
  <Constraints LINK_LANGUAGE Generator Expression>`
  ``$<LINK_LANGUAGE:language>`` for constraints about the usage of this
  generator expression.

.. genex:: $<LINK_LANGUAGE:languages>

  .. versionadded:: 3.18

  ``1`` when the language used for link step matches any of the entries
  in ``languages``, otherwise ``0``.  This expression may be used to specify
  link libraries, link options, link directories and link dependencies of a
  particular language in a target. For example:

  .. code-block:: cmake

    add_library(api_C ...)
    add_library(api_CXX ...)
    add_library(api INTERFACE)
    target_link_options(api INTERFACE $<$<LINK_LANGUAGE:C>:-opt_c>
                                        $<$<LINK_LANGUAGE:CXX>:-opt_cxx>)
    target_link_libraries(api INTERFACE $<$<LINK_LANGUAGE:C>:api_C>
                                        $<$<LINK_LANGUAGE:CXX>:api_CXX>)

    add_executable(myapp1 main.c)
    target_link_options(myapp1 PRIVATE api)

    add_executable(myapp2 main.cpp)
    target_link_options(myapp2 PRIVATE api)

  This specifies to use the ``api`` target for linking targets ``myapp1`` and
  ``myapp2``. In practice, ``myapp1`` will link with target ``api_C`` and
  option ``-opt_c`` because it will use ``C`` as link language. And ``myapp2``
  will link with ``api_CXX`` and option ``-opt_cxx`` because ``CXX`` will be
  the link language.

  .. _`Constraints LINK_LANGUAGE Generator Expression`:

  .. note::

    To determine the link language of a target, it is required to collect,
    transitively, all the targets which will be linked to it. So, for link
    libraries properties, a double evaluation will be done. During the first
    evaluation, ``$<LINK_LANGUAGE:..>`` expressions will always return ``0``.
    The link language computed after this first pass will be used to do the
    second pass. To avoid inconsistency, it is required that the second pass
    do not change the link language. Moreover, to avoid unexpected
    side-effects, it is required to specify complete entities as part of the
    ``$<LINK_LANGUAGE:..>`` expression. For example:

    .. code-block:: cmake

      add_library(lib STATIC file.cxx)
      add_library(libother STATIC file.c)

      # bad usage
      add_executable(myapp1 main.c)
      target_link_libraries(myapp1 PRIVATE lib$<$<LINK_LANGUAGE:C>:other>)

      # correct usage
      add_executable(myapp2 main.c)
      target_link_libraries(myapp2 PRIVATE $<$<LINK_LANGUAGE:C>:libother>)

    In this example, for ``myapp1``, the first pass will, unexpectedly,
    determine that the link language is ``CXX`` because the evaluation of the
    generator expression will be an empty string so ``myapp1`` will depends on
    target ``lib`` which is ``C++``. On the contrary, for ``myapp2``, the first
    evaluation will give ``C`` as link language, so the second pass will
    correctly add target ``libother`` as link dependency.

.. genex:: $<DEVICE_LINK:list>

  .. versionadded:: 3.18

  Returns the list if it is the device link step, an empty list otherwise.
  The device link step is controlled by :prop_tgt:`CUDA_SEPARABLE_COMPILATION`
  and :prop_tgt:`CUDA_RESOLVE_DEVICE_SYMBOLS` properties and
  policy :policy:`CMP0105`. This expression can only be used to specify link
  options.

.. genex:: $<HOST_LINK:list>

  .. versionadded:: 3.18

  Returns the list if it is the normal link step, an empty list otherwise.
  This expression is mainly useful when a device link step is also involved
  (see ``$<DEVICE_LINK:list>`` generator expression). This expression can only
  be used to specify link options.

String-Valued Generator Expressions
===================================

These expressions expand to some string.
For example,

.. code-block:: cmake

  include_directories(/usr/include/$<CXX_COMPILER_ID>/)

expands to ``/usr/include/GNU/`` or ``/usr/include/Clang/`` etc, depending on
the compiler identifier.

String-valued expressions may also be combined with other expressions.
Here an example for a string-valued expression within a boolean expressions
within a conditional expression:

.. code-block:: cmake

  $<$<VERSION_LESS:$<CXX_COMPILER_VERSION>,4.2.0>:OLD_COMPILER>

expands to ``OLD_COMPILER`` if the
:variable:`CMAKE_CXX_COMPILER_VERSION <CMAKE_<LANG>_COMPILER_VERSION>` is less
than 4.2.0.

And here two nested string-valued expressions:

.. code-block:: cmake

  -I$<JOIN:$<TARGET_PROPERTY:INCLUDE_DIRECTORIES>, -I>

generates a string of the entries in the :prop_tgt:`INCLUDE_DIRECTORIES` target
property with each entry preceded by ``-I``.

Expanding on the previous example, if one first wants to check if the
``INCLUDE_DIRECTORIES`` property is non-empty, then it is advisable to
introduce a helper variable to keep the code readable:

.. code-block:: cmake

  set(prop "$<TARGET_PROPERTY:INCLUDE_DIRECTORIES>") # helper variable
  $<$<BOOL:${prop}>:-I$<JOIN:${prop}, -I>>

The following string-valued generator expressions are available:

Escaped Characters
------------------

String literals to escape the special meaning a character would otherwise have:

.. genex:: $<ANGLE-R>

  A literal ``>``. Used for example to compare strings that contain a ``>``.

.. genex:: $<COMMA>

  A literal ``,``. Used for example to compare strings which contain a ``,``.

.. genex:: $<SEMICOLON>

  A literal ``;``. Used to prevent list expansion on an argument with ``;``.

.. _`Conditional Generator Expressions`:

Conditional Expressions
-----------------------

Conditional generator expressions depend on a boolean condition
that must be ``0`` or ``1``.

.. genex:: $<condition:true_string>

  Evaluates to ``true_string`` if ``condition`` is ``1``.
  Otherwise evaluates to the empty string.

.. genex:: $<IF:condition,true_string,false_string>

  .. versionadded:: 3.8

  Evaluates to ``true_string`` if ``condition`` is ``1``.
  Otherwise evaluates to ``false_string``.

Typically, the ``condition`` is a :ref:`boolean generator expression
<Boolean Generator Expressions>`.  For instance,

.. code-block:: cmake

  $<$<CONFIG:Debug>:DEBUG_MODE>

expands to ``DEBUG_MODE`` when the ``Debug`` configuration is used, and
otherwise expands to the empty string.

.. _`String Transforming Generator Expressions`:

String Transformations
----------------------

.. genex:: $<JOIN:list,string>

  Joins the list with the content of ``string``.

.. genex:: $<REMOVE_DUPLICATES:list>

  .. versionadded:: 3.15

  Removes duplicated items in the given ``list``. The relative order of items
  is preserved, but if duplicates are encountered, only the first instance is
  preserved.

.. genex:: $<FILTER:list,INCLUDE|EXCLUDE,regex>

  .. versionadded:: 3.15

  Includes or removes items from ``list`` that match the regular expression ``regex``.

.. genex:: $<LOWER_CASE:string>

  Content of ``string`` converted to lower case.

.. genex:: $<UPPER_CASE:string>

  Content of ``string`` converted to upper case.

.. genex:: $<GENEX_EVAL:expr>

  .. versionadded:: 3.12

  Content of ``expr`` evaluated as a generator expression in the current
  context. This enables consumption of generator expressions whose
  evaluation results itself in generator expressions.

.. genex:: $<TARGET_GENEX_EVAL:tgt,expr>

  .. versionadded:: 3.12

  Content of ``expr`` evaluated as a generator expression in the context of
  ``tgt`` target. This enables consumption of custom target properties that
  themselves contain generator expressions.

  Having the capability to evaluate generator expressions is very useful when
  you want to manage custom properties supporting generator expressions.
  For example:

  .. code-block:: cmake

    add_library(foo ...)

    set_property(TARGET foo PROPERTY
      CUSTOM_KEYS $<$<CONFIG:DEBUG>:FOO_EXTRA_THINGS>
    )

    add_custom_target(printFooKeys
      COMMAND ${CMAKE_COMMAND} -E echo $<TARGET_PROPERTY:foo,CUSTOM_KEYS>
    )

  This naive implementation of the ``printFooKeys`` custom command is wrong
  because ``CUSTOM_KEYS`` target property is not evaluated and the content
  is passed as is (i.e. ``$<$<CONFIG:DEBUG>:FOO_EXTRA_THINGS>``).

  To have the expected result (i.e. ``FOO_EXTRA_THINGS`` if config is
  ``Debug``), it is required to evaluate the output of
  ``$<TARGET_PROPERTY:foo,CUSTOM_KEYS>``:

  .. code-block:: cmake

    add_custom_target(printFooKeys
      COMMAND ${CMAKE_COMMAND} -E
        echo $<TARGET_GENEX_EVAL:foo,$<TARGET_PROPERTY:foo,CUSTOM_KEYS>>
    )

.. _GenEx Path Decomposition:

Path Decomposition
------------------

The ``$<PATH>`` generator expression offers the same capabilities as the
:command:`cmake_path` command, for the
:ref:`Decomposition <Path Decomposition>` options.

For all ``$<PATH>`` generator expressions, paths are expected in cmake-style
format. The :ref:`$\<PATH:CMAKE_PATH\> <GenEx PATH-CMAKE_PATH>` generator
expression can be used to convert a native path to a cmake-style one.

The ``$<PATH>`` generator expression can also be used for path
:ref:`Queries <GenEx Path Queries>` and
:ref:`Transformations <GenEx Path Transformations>`.

.. genex:: $<PATH:GET_*,...>

  .. versionadded:: 3.24

  The following operations retrieve a different component or group of
  components from a path. See :ref:`Path Structure And Terminology` for the
  meaning of each path component.

  ::

    $<PATH:GET_ROOT_NAME,path>
    $<PATH:GET_ROOT_DIRECTORY,path>
    $<PATH:GET_ROOT_PATH,path>
    $<PATH:GET_FILENAME,path>
    $<PATH:GET_EXTENSION[,LAST_ONLY],path>
    $<PATH:GET_STEM[,LAST_ONLY],path>
    $<PATH:GET_RELATIVE_PART,path>
    $<PATH:GET_PARENT_PATH,path>

  If a requested component is not present in the path, an empty string is
  returned.

.. _GenEx Path Transformations:

Path Transformations
--------------------

The ``$<PATH>`` generator expression offers the same capabilities as the
:command:`cmake_path` command, for the
:ref:`Modification <Path Modification>` and
:ref:`Generation <Path Generation>` options.

For all ``$<PATH>`` generator expressions, paths are expected in cmake-style
format. The :ref:`$\<PATH:CMAKE_PATH\> <GenEx PATH-CMAKE_PATH>` generator
expression can be used to convert a native path to a cmake-style one.

The ``$<PATH>`` generator expression can also be used for path
:ref:`Queries <GenEx Path Queries>` and
:ref:`Decomposition <GenEx Path Decomposition>`.

.. _GenEx PATH-CMAKE_PATH:

.. genex:: $<PATH:CMAKE_PATH[,NORMALIZE],path>

  .. versionadded:: 3.24

  Returns ``path``. If ``path`` is a native path, it is converted into a
  cmake-style path with forward-slashes (``/``). On Windows, the long filename
  marker is taken into account.

  When the ``NORMALIZE`` option is specified, the path is :ref:`normalized
  <Normalization>` after the conversion.

.. genex:: $<PATH:APPEND,path,input,...>

  .. versionadded:: 3.24

  Returns all the ``input`` arguments appended to ``path`` using ``/`` as the
  ``directory-separator``. Depending on the ``input``, the value of ``path``
  may be discarded.

  See :ref:`cmake_path(APPEND) <APPEND>` for more details.

.. genex:: $<PATH:REMOVE_FILENAME,path>

  .. versionadded:: 3.24

  Returns ``path`` with filename component (as returned by
  ``$<PATH:GET_FILENAME>``) removed. After removal, any trailing
  ``directory-separator`` is left alone, if present.

  See :ref:`cmake_path(REMOVE_FILENAME) <REMOVE_FILENAME>` for more details.

.. genex:: $<PATH:REPLACE_FILENAME,path,input>

  .. versionadded:: 3.24

  Returns ``path`` with the filename component replaced by ``input``. If
  ``path`` has no filename component (i.e. ``$<PATH:HAS_FILENAME>`` returns
  ``0``), ``path`` is unchanged.

  See :ref:`cmake_path(REPLACE_FILENAME) <REPLACE_FILENAME>` for more details.

.. genex:: $<PATH:REMOVE_EXTENSION[,LAST_ONLY],path>

  .. versionadded:: 3.24

  Returns ``path`` with the :ref:`extension <EXTENSION_DEF>` removed, if any.

  See :ref:`cmake_path(REMOVE_EXTENSION) <REMOVE_EXTENSION>` for more details.

.. genex:: $<PATH:REPLACE_EXTENSION[,LAST_ONLY],path>

  .. versionadded:: 3.24

  Returns ``path`` with the :ref:`extension <EXTENSION_DEF>` replaced by
  ``input``, if any.

  See :ref:`cmake_path(REPLACE_EXTENSION) <REPLACE_EXTENSION>` for more details.

.. genex:: $<PATH:NORMAL_PATH,path>

  .. versionadded:: 3.24

  Returns ``path`` normalized according to the steps described in
  :ref:`Normalization`.

.. genex:: $<PATH:RELATIVE_PATH,path,base_directory>

  .. versionadded:: 3.24

  Returns ``path``, modified to make it relative to the ``base_directory``
  argument.

  See :ref:`cmake_path(RELATIVE_PATH) <cmake_path-RELATIVE_PATH>` for more
  details.

.. genex:: $<PATH:ABSOLUTE_PATH[,NORMALIZE],path,base_directory>

  .. versionadded:: 3.24

  Returns ``path`` as absolute. If ``path`` is a relative path
  (``$<PATH:IS_RELATIVE>`` returns ``1``), it is evaluated relative to the
  given base directory specified by ``base_directory`` argument.

  When the ``NORMALIZE`` option is specified, the path is
  :ref:`normalized <Normalization>` after the path computation.

  See :ref:`cmake_path(ABSOLUTE_PATH) <ABSOLUTE_PATH>` for more details.

Variable Queries
----------------

.. genex:: $<CONFIG>

  Configuration name.

.. genex:: $<CONFIGURATION>

  Configuration name. Deprecated since CMake 3.0. Use ``CONFIG`` instead.

.. genex:: $<PLATFORM_ID>

  The current system's CMake platform id.
  See also the :variable:`CMAKE_SYSTEM_NAME` variable.

.. genex:: $<C_COMPILER_ID>

  The CMake's compiler id of the C compiler used.
  See also the :variable:`CMAKE_<LANG>_COMPILER_ID` variable.

.. genex:: $<CXX_COMPILER_ID>

  The CMake's compiler id of the CXX compiler used.
  See also the :variable:`CMAKE_<LANG>_COMPILER_ID` variable.

.. genex:: $<CUDA_COMPILER_ID>

  The CMake's compiler id of the CUDA compiler used.
  See also the :variable:`CMAKE_<LANG>_COMPILER_ID` variable.

.. genex:: $<OBJC_COMPILER_ID>

  .. versionadded:: 3.16

  The CMake's compiler id of the OBJC compiler used.
  See also the :variable:`CMAKE_<LANG>_COMPILER_ID` variable.

.. genex:: $<OBJCXX_COMPILER_ID>

  .. versionadded:: 3.16

  The CMake's compiler id of the OBJCXX compiler used.
  See also the :variable:`CMAKE_<LANG>_COMPILER_ID` variable.

.. genex:: $<Fortran_COMPILER_ID>

  The CMake's compiler id of the Fortran compiler used.
  See also the :variable:`CMAKE_<LANG>_COMPILER_ID` variable.

.. genex:: $<HIP_COMPILER_ID>

  .. versionadded:: 3.21

  The CMake's compiler id of the HIP compiler used.
  See also the :variable:`CMAKE_<LANG>_COMPILER_ID` variable.

.. genex:: $<ISPC_COMPILER_ID>

  .. versionadded:: 3.19

  The CMake's compiler id of the ISPC compiler used.
  See also the :variable:`CMAKE_<LANG>_COMPILER_ID` variable.

.. genex:: $<C_COMPILER_VERSION>

  The version of the C compiler used.
  See also the :variable:`CMAKE_<LANG>_COMPILER_VERSION` variable.

.. genex:: $<CXX_COMPILER_VERSION>

  The version of the CXX compiler used.
  See also the :variable:`CMAKE_<LANG>_COMPILER_VERSION` variable.

.. genex:: $<CUDA_COMPILER_VERSION>

  The version of the CUDA compiler used.
  See also the :variable:`CMAKE_<LANG>_COMPILER_VERSION` variable.

.. genex:: $<OBJC_COMPILER_VERSION>

  .. versionadded:: 3.16

  The version of the OBJC compiler used.
  See also the :variable:`CMAKE_<LANG>_COMPILER_VERSION` variable.

.. genex:: $<OBJCXX_COMPILER_VERSION>

  .. versionadded:: 3.16

  The version of the OBJCXX compiler used.
  See also the :variable:`CMAKE_<LANG>_COMPILER_VERSION` variable.

.. genex:: $<Fortran_COMPILER_VERSION>

  The version of the Fortran compiler used.
  See also the :variable:`CMAKE_<LANG>_COMPILER_VERSION` variable.

.. genex:: $<HIP_COMPILER_VERSION>

  .. versionadded:: 3.21

  The version of the HIP compiler used.
  See also the :variable:`CMAKE_<LANG>_COMPILER_VERSION` variable.

.. genex:: $<ISPC_COMPILER_VERSION>

  .. versionadded:: 3.19

  The version of the ISPC compiler used.
  See also the :variable:`CMAKE_<LANG>_COMPILER_VERSION` variable.

.. genex:: $<COMPILE_LANGUAGE>

  .. versionadded:: 3.3

  The compile language of source files when evaluating compile options.
  See :ref:`the related boolean expression
  <Boolean COMPILE_LANGUAGE Generator Expression>`
  ``$<COMPILE_LANGUAGE:language>``
  for notes about the portability of this generator expression.

.. genex:: $<LINK_LANGUAGE>

  .. versionadded:: 3.18

  The link language of target when evaluating link options.
  See :ref:`the related boolean expression
  <Boolean LINK_LANGUAGE Generator Expression>` ``$<LINK_LANGUAGE:language>``
  for notes about the portability of this generator expression.

  .. note::

    This generator expression is not supported by the link libraries
    properties to avoid side-effects due to the double evaluation of
    these properties.

.. _`Target-Dependent Queries`:

Target-Dependent Queries
------------------------

These queries refer to a target ``tgt``. This can be any runtime artifact,
namely:

* an executable target created by :command:`add_executable`
* a shared library target (``.so``, ``.dll`` but not their ``.lib`` import library)
  created by :command:`add_library`
* a static library target created by :command:`add_library`

In the following, "the ``tgt`` filename" means the name of the ``tgt``
binary file. This has to be distinguished from "the target name",
which is just the string ``tgt``.

.. genex:: $<TARGET_NAME_IF_EXISTS:tgt>

  .. versionadded:: 3.12

  The target name ``tgt`` if the target exists, an empty string otherwise.

  Note that ``tgt`` is not added as a dependency of the target this
  expression is evaluated on.

.. genex:: $<TARGET_FILE:tgt>

  Full path to the ``tgt`` binary file.

.. genex:: $<TARGET_FILE_BASE_NAME:tgt>

  .. versionadded:: 3.15

  Base name of ``tgt``, i.e. ``$<TARGET_FILE_NAME:tgt>`` without prefix and
  suffix.
  For example, if the ``tgt`` filename is ``libbase.so``, the base name is ``base``.

  See also the :prop_tgt:`OUTPUT_NAME`, :prop_tgt:`ARCHIVE_OUTPUT_NAME`,
  :prop_tgt:`LIBRARY_OUTPUT_NAME` and :prop_tgt:`RUNTIME_OUTPUT_NAME`
  target properties and their configuration specific variants
  :prop_tgt:`OUTPUT_NAME_<CONFIG>`, :prop_tgt:`ARCHIVE_OUTPUT_NAME_<CONFIG>`,
  :prop_tgt:`LIBRARY_OUTPUT_NAME_<CONFIG>` and
  :prop_tgt:`RUNTIME_OUTPUT_NAME_<CONFIG>`.

  The :prop_tgt:`<CONFIG>_POSTFIX` and :prop_tgt:`DEBUG_POSTFIX` target
  properties can also be considered.

  Note that ``tgt`` is not added as a dependency of the target this
  expression is evaluated on.

.. genex:: $<TARGET_FILE_PREFIX:tgt>

  .. versionadded:: 3.15

  Prefix of the ``tgt`` filename (such as ``lib``).

  See also the :prop_tgt:`PREFIX` target property.

  Note that ``tgt`` is not added as a dependency of the target this
  expression is evaluated on.

.. genex:: $<TARGET_FILE_SUFFIX:tgt>

  .. versionadded:: 3.15

  Suffix of the ``tgt`` filename (extension such as ``.so`` or ``.exe``).

  See also the :prop_tgt:`SUFFIX` target property.

  Note that ``tgt`` is not added as a dependency of the target this
  expression is evaluated on.

.. genex:: $<TARGET_FILE_NAME:tgt>

  The ``tgt`` filename.

  Note that ``tgt`` is not added as a dependency of the target this
  expression is evaluated on (see policy :policy:`CMP0112`).

.. genex:: $<TARGET_FILE_DIR:tgt>

  Directory of the ``tgt`` binary file.

  Note that ``tgt`` is not added as a dependency of the target this
  expression is evaluated on (see policy :policy:`CMP0112`).

.. genex:: $<TARGET_LINKER_FILE:tgt>

  File used when linking to the ``tgt`` target.  This will usually
  be the library that ``tgt`` represents (``.a``, ``.lib``, ``.so``),
  but for a shared library on DLL platforms, it would be the ``.lib``
  import library associated with the DLL.

.. genex:: $<TARGET_LINKER_FILE_BASE_NAME:tgt>

  .. versionadded:: 3.15

  Base name of file used to link the target ``tgt``, i.e.
  ``$<TARGET_LINKER_FILE_NAME:tgt>`` without prefix and suffix. For example,
  if target file name is ``libbase.a``, the base name is ``base``.

  See also the :prop_tgt:`OUTPUT_NAME`, :prop_tgt:`ARCHIVE_OUTPUT_NAME`,
  and :prop_tgt:`LIBRARY_OUTPUT_NAME` target properties and their configuration
  specific variants :prop_tgt:`OUTPUT_NAME_<CONFIG>`,
  :prop_tgt:`ARCHIVE_OUTPUT_NAME_<CONFIG>` and
  :prop_tgt:`LIBRARY_OUTPUT_NAME_<CONFIG>`.

  The :prop_tgt:`<CONFIG>_POSTFIX` and :prop_tgt:`DEBUG_POSTFIX` target
  properties can also be considered.

  Note that ``tgt`` is not added as a dependency of the target this
  expression is evaluated on.

.. genex:: $<TARGET_LINKER_FILE_PREFIX:tgt>

  .. versionadded:: 3.15

  Prefix of file used to link target ``tgt``.

  See also the :prop_tgt:`PREFIX` and :prop_tgt:`IMPORT_PREFIX` target
  properties.

  Note that ``tgt`` is not added as a dependency of the target this
  expression is evaluated on.

.. genex:: $<TARGET_LINKER_FILE_SUFFIX:tgt>

  .. versionadded:: 3.15

  Suffix of file used to link where ``tgt`` is the name of a target.

  The suffix corresponds to the file extension (such as ".so" or ".lib").

  See also the :prop_tgt:`SUFFIX` and :prop_tgt:`IMPORT_SUFFIX` target
  properties.

  Note that ``tgt`` is not added as a dependency of the target this
  expression is evaluated on.

.. genex:: $<TARGET_LINKER_FILE_NAME:tgt>

  Name of file used to link target ``tgt``.

  Note that ``tgt`` is not added as a dependency of the target this
  expression is evaluated on (see policy :policy:`CMP0112`).

.. genex:: $<TARGET_LINKER_FILE_DIR:tgt>

  Directory of file used to link target ``tgt``.

  Note that ``tgt`` is not added as a dependency of the target this
  expression is evaluated on (see policy :policy:`CMP0112`).

.. genex:: $<TARGET_SONAME_FILE:tgt>

  File with soname (``.so.3``) where ``tgt`` is the name of a target.
.. genex:: $<TARGET_SONAME_FILE_NAME:tgt>

  Name of file with soname (``.so.3``).

  Note that ``tgt`` is not added as a dependency of the target this
  expression is evaluated on (see policy :policy:`CMP0112`).

.. genex:: $<TARGET_SONAME_FILE_DIR:tgt>

  Directory of with soname (``.so.3``).

  Note that ``tgt`` is not added as a dependency of the target this
  expression is evaluated on (see policy :policy:`CMP0112`).

.. genex:: $<TARGET_PDB_FILE:tgt>

  .. versionadded:: 3.1

  Full path to the linker generated program database file (.pdb)
  where ``tgt`` is the name of a target.

  See also the :prop_tgt:`PDB_NAME` and :prop_tgt:`PDB_OUTPUT_DIRECTORY`
  target properties and their configuration specific variants
  :prop_tgt:`PDB_NAME_<CONFIG>` and :prop_tgt:`PDB_OUTPUT_DIRECTORY_<CONFIG>`.

.. genex:: $<TARGET_PDB_FILE_BASE_NAME:tgt>

  .. versionadded:: 3.15

  Base name of the linker generated program database file (.pdb)
  where ``tgt`` is the name of a target.

  The base name corresponds to the target PDB file name (see
  ``$<TARGET_PDB_FILE_NAME:tgt>``) without prefix and suffix. For example,
  if target file name is ``base.pdb``, the base name is ``base``.

  See also the :prop_tgt:`PDB_NAME` target property and its configuration
  specific variant :prop_tgt:`PDB_NAME_<CONFIG>`.

  The :prop_tgt:`<CONFIG>_POSTFIX` and :prop_tgt:`DEBUG_POSTFIX` target
  properties can also be considered.

  Note that ``tgt`` is not added as a dependency of the target this
  expression is evaluated on.

.. genex:: $<TARGET_PDB_FILE_NAME:tgt>

  .. versionadded:: 3.1

  Name of the linker generated program database file (.pdb).

  Note that ``tgt`` is not added as a dependency of the target this
  expression is evaluated on (see policy :policy:`CMP0112`).

.. genex:: $<TARGET_PDB_FILE_DIR:tgt>

  .. versionadded:: 3.1

  Directory of the linker generated program database file (.pdb).

  Note that ``tgt`` is not added as a dependency of the target this
  expression is evaluated on (see policy :policy:`CMP0112`).

.. genex:: $<TARGET_BUNDLE_DIR:tgt>

  .. versionadded:: 3.9

  Full path to the bundle directory (``/path/to/my.app``,
  ``/path/to/my.framework``, or ``/path/to/my.bundle``),
  where ``tgt`` is the name of a target.

  Note that ``tgt`` is not added as a dependency of the target this
  expression is evaluated on (see policy :policy:`CMP0112`).

.. genex:: $<TARGET_BUNDLE_DIR_NAME:tgt>

  .. versionadded:: 3.24

  Name of the bundle directory (``my.app``, ``my.framework``, or
  ``my.bundle``), where ``tgt`` is the name of a target.

  Note that ``tgt`` is not added as a dependency of the target this
  expression is evaluated on (see policy :policy:`CMP0112`).

.. genex:: $<TARGET_BUNDLE_CONTENT_DIR:tgt>

  .. versionadded:: 3.9

  Full path to the bundle content directory where ``tgt`` is the name of a
  target.  For the macOS SDK it leads to ``/path/to/my.app/Contents``,
  ``/path/to/my.framework``, or ``/path/to/my.bundle/Contents``.
  For all other SDKs (e.g. iOS) it leads to ``/path/to/my.app``,
  ``/path/to/my.framework``, or ``/path/to/my.bundle`` due to the flat
  bundle structure.

  Note that ``tgt`` is not added as a dependency of the target this
  expression is evaluated on (see policy :policy:`CMP0112`).

.. genex:: $<TARGET_PROPERTY:tgt,prop>

  Value of the property ``prop`` on the target ``tgt``.

  Note that ``tgt`` is not added as a dependency of the target this
  expression is evaluated on.

.. genex:: $<TARGET_PROPERTY:prop>

  Value of the property ``prop`` on the target for which the expression
  is being evaluated. Note that for generator expressions in
  :ref:`Target Usage Requirements` this is the consuming target rather
  than the target specifying the requirement.

.. genex:: $<TARGET_RUNTIME_DLLS:tgt>

  .. versionadded:: 3.21

  List of DLLs that the target depends on at runtime. This is determined by
  the locations of all the ``SHARED`` targets in the target's transitive
  dependencies. Using this generator expression on targets other than
  executables, ``SHARED`` libraries, and ``MODULE`` libraries is an error. On
  non-DLL platforms, it evaluates to an empty string.

  This generator expression can be used to copy all of the DLLs that a target
  depends on into its output directory in a ``POST_BUILD`` custom command. For
  example:

  .. code-block:: cmake

    find_package(foo CONFIG REQUIRED) # package generated by install(EXPORT)

    add_executable(exe main.c)
    target_link_libraries(exe PRIVATE foo::foo foo::bar)
    add_custom_command(TARGET exe POST_BUILD
      COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_RUNTIME_DLLS:exe> $<TARGET_FILE_DIR:exe>
      COMMAND_EXPAND_LISTS
      )

  .. note::

    :ref:`Imported Targets` are supported only if they know the location
    of their ``.dll`` files.  An imported ``SHARED`` library must have
    :prop_tgt:`IMPORTED_LOCATION` set to its ``.dll`` file.  See the
    :ref:`add_library imported libraries <add_library imported libraries>`
    section for details.  Many :ref:`Find Modules` produce imported targets
    with the ``UNKNOWN`` type and therefore will be ignored.

.. genex:: $<INSTALL_PREFIX>

  Content of the install prefix when the target is exported via
  :command:`install(EXPORT)`, or when evaluated in the
  :prop_tgt:`INSTALL_NAME_DIR` property or the ``INSTALL_NAME_DIR`` argument of
  :command:`install(RUNTIME_DEPENDENCY_SET)`, and empty otherwise.

Output-Related Expressions
--------------------------

.. genex:: $<TARGET_NAME:...>

  Marks ``...`` as being the name of a target.  This is required if exporting
  targets to multiple dependent export sets.  The ``...`` must be a literal
  name of a target- it may not contain generator expressions.

.. genex:: $<LINK_ONLY:...>

  .. versionadded:: 3.1

  Content of ``...``, except while collecting :ref:`Target Usage Requirements`,
  in which case it is the empty string.  This is intended for use in an
  :prop_tgt:`INTERFACE_LINK_LIBRARIES` target property, typically populated
  via the :command:`target_link_libraries` command, to specify private link
  dependencies without other usage requirements.

  .. versionadded:: 3.24
    ``LINK_ONLY`` may also be used in a :prop_tgt:`LINK_LIBRARIES` target
    property.  See policy :policy:`CMP0131`.

.. genex:: $<LINK_LIBRARY:feature,library-list>

  .. versionadded:: 3.24

  Manage how libraries are specified during the link step.
  This expression may be used to specify how to link libraries in a target.
  For example:

  .. code-block:: cmake

    add_library(lib1 STATIC ...)
    add_library(lib2 ...)
    target_link_libraries(lib2 PRIVATE "$<LINK_LIBRARY:load_archive,lib1>")

  This specify to use the ``lib1`` target with feature ``load_archive`` for
  linking target ``lib2``. The feature must have be defined by
  :variable:`CMAKE_<LANG>_LINK_LIBRARY_USING_<FEATURE>` variable or, if
  :variable:`CMAKE_<LANG>_LINK_LIBRARY_USING_<FEATURE>_SUPPORTED` is false,
  by :variable:`CMAKE_LINK_LIBRARY_USING_<FEATURE>` variable.

  .. note::

    The evaluation of this generator expression will use, for the following
    variables, the values defined at the level of the creation of the target:

    * :variable:`CMAKE_<LANG>_LINK_LIBRARY_USING_<FEATURE>_SUPPORTED`
    * :variable:`CMAKE_<LANG>_LINK_LIBRARY_USING_<FEATURE>`
    * :variable:`CMAKE_LINK_LIBRARY_USING_<FEATURE>_SUPPORTED`
    * :variable:`CMAKE_LINK_LIBRARY_USING_<FEATURE>`

  This expression can only be used to specify link libraries (i.e. part of
  :command:`link_libraries` or :command:`target_link_libraries` commands and
  :prop_tgt:`LINK_LIBRARIES` or :prop_tgt:`INTERFACE_LINK_LIBRARIES` target
  properties).

  .. note::

    If this expression appears in the :prop_tgt:`INTERFACE_LINK_LIBRARIES`
    property of a target, it will be included in the imported target generated
    by :command:`install(EXPORT)` command. It is the responsibility of the
    environment consuming this import to define the link feature used by this
    expression.

  The ``library-list`` argument can hold CMake targets or external libraries.
  Any CMake target of type :ref:`OBJECT <Object Libraries>` or
  :ref:`INTERFACE <Interface Libraries>` will be ignored by this expression and
  will be handled in the standard way.

  Each target or external library involved in the link step must have only one
  kind of feature (the absence of feature is also incompatible with any
  feature). For example:

  .. code-block:: cmake

    add_library(lib1 ...)

    add_library(lib2 ...)
    target_link_libraries(lib2 PUBLIC "$<LINK_LIBRARY:feature1,lib1>")

    add_library(lib3 ...)
    target_link_libraries(lib3 PRIVATE lib1 lib2)
    # an error will be raised here because lib1 has two different features

  To resolve such incompatibilities, the :prop_tgt:`LINK_LIBRARY_OVERRIDE`
  and  :prop_tgt:`LINK_LIBRARY_OVERRIDE_<LIBRARY>` target properties can be
  used.

  .. note::

    This expression does not guarantee that the list of specified libraries
    will be kept grouped. So, to manage constructs like ``start-group`` and
    ``end-group``, as supported by ``GNU ld``, the :genex:`LINK_GROUP`
    generator expression can be used.

  CMake pre-defines some features of general interest:

  .. include:: ../variable/LINK_LIBRARY_PREDEFINED_FEATURES.txt

.. genex:: $<LINK_GROUP:feature,library-list>

  .. versionadded:: 3.24

  Manage the grouping of libraries during the link step.
  This expression may be used to specify how to kept groups of libraries during
  the link of a target.
  For example:

  .. code-block:: cmake

    add_library(lib1 STATIC ...)
    add_library(lib2 ...)
    target_link_libraries(lib2 PRIVATE "$<LINK_GROUP:cross_refs,lib1,external>")

  This specify to use the ``lib1`` target and ``external`` library  with the
  group feature ``cross_refs`` for linking target ``lib2``. The feature must
  have be defined by :variable:`CMAKE_<LANG>_LINK_GROUP_USING_<FEATURE>`
  variable or, if :variable:`CMAKE_<LANG>_LINK_GROUP_USING_<FEATURE>_SUPPORTED`
  is false, by :variable:`CMAKE_LINK_GROUP_USING_<FEATURE>` variable.

  .. note::

    The evaluation of this generator expression will use, for the following
    variables, the values defined at the level of the creation of the target:

    * :variable:`CMAKE_<LANG>_LINK_GROUP_USING_<FEATURE>_SUPPORTED`
    * :variable:`CMAKE_<LANG>_LINK_GROUP_USING_<FEATURE>`
    * :variable:`CMAKE_LINK_GROUP_USING_<FEATURE>_SUPPORTED`
    * :variable:`CMAKE_LINK_GROUP_USING_<FEATURE>`

  This expression can only be used to specify link libraries (i.e. part of
  :command:`link_libraries` or :command:`target_link_libraries` commands and
  :prop_tgt:`LINK_LIBRARIES` or :prop_tgt:`INTERFACE_LINK_LIBRARIES` target
  properties).

  .. note::

    If this expression appears in the :prop_tgt:`INTERFACE_LINK_LIBRARIES`
    property of a target, it will be included in the imported target generated
    by :command:`install(EXPORT)` command. It is the responsibility of the
    environment consuming this import to define the link feature used by this
    expression.

  The ``library-list`` argument can hold CMake targets or external libraries.
  Any CMake target of type :ref:`OBJECT <Object Libraries>` or
  :ref:`INTERFACE <Interface Libraries>` will be ignored by this expression and
  will be handled in the standard way.

  .. note::

    This expression is compatible with the :genex:`LINK_LIBRARY` generator
    expression. The libraries involved in a group can be specified using the
    :genex:`LINK_LIBRARY` generator expression.

  Each target or external library involved in the link step can be part of
  different groups as far as these groups use the same feature, so mixing
  different group features for the same target or library is forbidden. The
  different groups will be part of the link step.

  .. code-block:: cmake

    add_library(lib1 ...)
    add_library(lib2 ...)

    add_library(lib3 ...)
    target_link_libraries(lib3 PUBLIC "$<LINK_GROUP:feature1,lib1,lib2>")

    add_library(lib4 ...)
    target_link_libraries(lib4 PRIVATE "$<LINK_GROUP:feature1,lib1,lib3>")
    # lib4 will be linked with the groups {lib1,lib2} and {lib1,lib3}

    add_library(lib5 ...)
    target_link_libraries(lib5 PRIVATE "$<LINK_GROUP:feature2,lib1,lib3>")
    # an error will be raised here because lib1 is part of two groups with
    # different features

  When a target or an external library is involved in the link step as part of
  a group and also as standalone, any occurrence of the standalone link item
  will be replaced by the group or groups it belong to.

  .. code-block:: cmake

    add_library(lib1 ...)
    add_library(lib2 ...)

    add_library(lib3 ...)
    target_link_libraries(lib3 PUBLIC lib1)

    add_library(lib4 ...)
    target_link_libraries(lib4 PRIVATE lib3 "$<LINK_GROUP:feature1,lib1,lib2>")
    # lib4 will only be linked with lib3 and the group {lib1,lib2}

  This example will be "re-written" by CMake in the following form:

  .. code-block:: cmake

    add_library(lib1 ...)
    add_library(lib2 ...)

    add_library(lib3 ...)
    target_link_libraries(lib3 PUBLIC "$<LINK_GROUP:feature1,lib1,lib2>")

    add_library(lib4 ...)
    target_link_libraries(lib4 PRIVATE lib3 "$<LINK_GROUP:feature1,lib1,lib2>")
    # lib4 will only be linked with lib3 and the group {lib1,lib2}

  Be aware that the precedence of the group over the standalone link item can
  result in some circular dependency between groups, which will raise an
  error because circular dependencies are not allowed for groups.

  .. code-block:: cmake

    add_library(lib1A ...)
    add_library(lib1B ...)

    add_library(lib2A ...)
    add_library(lib2B ...)

    target_link_libraries(lib1A PUBLIC lib2A)
    target_link_libraries(lib2B PUBLIC lib1B)

    add_library(lib ...)
    target_link_libraries(lib3 PRIVATE "$<LINK_GROUP:feat,lib1A,lib1B>"
                                       "$<LINK_GROUP:feat,lib2A,lib2B>")

  This example will be "re-written" by CMake in the following form:

  .. code-block:: cmake

    add_library(lib1A ...)
    add_library(lib1B ...)

    add_library(lib2A ...)
    add_library(lib2B ...)

    target_link_libraries(lib1A PUBLIC "$<LINK_GROUP:feat,lib2A,lib2B>")
    target_link_libraries(lib2B PUBLIC "$<LINK_GROUP:feat,lib1A,lib1B>")

    add_library(lib ...)
    target_link_libraries(lib3 PRIVATE "$<LINK_GROUP:feat,lib1A,lib1B>"
                                       "$<LINK_GROUP:feat,lib2A,lib2B>")

  So, we have a circular dependency between groups ``{lib1A,lib1B}`` and
  ``{lib2A,lib2B}``.

  CMake pre-defines some features of general interest:

  .. include:: ../variable/LINK_GROUP_PREDEFINED_FEATURES.txt

.. genex:: $<INSTALL_INTERFACE:...>

  Content of ``...`` when the property is exported using :command:`install(EXPORT)`,
  and empty otherwise.

.. genex:: $<BUILD_INTERFACE:...>

  Content of ``...`` when the property is exported using :command:`export`, or
  when the target is used by another target in the same buildsystem. Expands to
  the empty string otherwise.

.. genex:: $<MAKE_C_IDENTIFIER:...>

  Content of ``...`` converted to a C identifier.  The conversion follows the
  same behavior as :command:`string(MAKE_C_IDENTIFIER)`.

.. genex:: $<TARGET_OBJECTS:objLib>

  .. versionadded:: 3.1

  List of objects resulting from build of ``objLib``.

.. genex:: $<SHELL_PATH:...>

  .. versionadded:: 3.4

  Content of ``...`` converted to shell path style. For example, slashes are
  converted to backslashes in Windows shells and drive letters are converted
  to posix paths in MSYS shells. The ``...`` must be an absolute path.

  .. versionadded:: 3.14
    The ``...`` may be a :ref:`semicolon-separated list <CMake Language Lists>`
    of paths, in which case each path is converted individually and a result
    list is generated using the shell path separator (``:`` on POSIX and
    ``;`` on Windows).  Be sure to enclose the argument containing this genex
    in double quotes in CMake source code so that ``;`` does not split arguments.

.. genex:: $<OUTPUT_CONFIG:...>

  .. versionadded:: 3.20

  Only valid in :command:`add_custom_command` and :command:`add_custom_target`
  as the outer-most generator expression in an argument.
  With the :generator:`Ninja Multi-Config` generator, generator expressions
  in ``...`` are evaluated using the custom command's "output config".
  With other generators, the content of ``...`` is evaluated normally.

.. genex:: $<COMMAND_CONFIG:...>

  .. versionadded:: 3.20

  Only valid in :command:`add_custom_command` and :command:`add_custom_target`
  as the outer-most generator expression in an argument.
  With the :generator:`Ninja Multi-Config` generator, generator expressions
  in ``...`` are evaluated using the custom command's "command config".
  With other generators, the content of ``...`` is evaluated normally.

Debugging
=========

Since generator expressions are evaluated during generation of the buildsystem,
and not during processing of ``CMakeLists.txt`` files, it is not possible to
inspect their result with the :command:`message()` command.

One possible way to generate debug messages is to add a custom target,

.. code-block:: cmake

  add_custom_target(genexdebug COMMAND ${CMAKE_COMMAND} -E echo "$<...>")

The shell command ``make genexdebug`` (invoked after execution of ``cmake``)
would then print the result of ``$<...>``.

Another way is to write debug messages to a file:

.. code-block:: cmake

  file(GENERATE OUTPUT filename CONTENT "$<...>")

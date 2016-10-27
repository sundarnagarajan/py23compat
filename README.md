# py23compat
Module to inject python 2-3 compatible features from python-future

I wanted a way to do the following:
- Make new python source files Py2-3 compatible from the start using python-future
- Consistently import (inject) python-future features into all source files in a package
- Avoid having to do multiple imports at the top of each source file - potentially
leading to mistakes or inconsistencies



Dependencies:
------------
    Uses and requires installation of the excellent 'future' package.
    Install with pip install future

    This module has no other external dependencies. You can place it
    anywhere on your PYTHONPATH

    Does not work with Python2 < 2.6 - importing this module or calling
    inject_compat() should have no effect in such cases

Four capabilities, three variables to set and one method:
--------------------------------------------------------
    Capability                              Variable / Method
    ----------                              --------
    Customize features for source files     ALWAYS_AVOID variable
    Customize features for REPL             ALWAYS_AVOID_REPL variable
    Select features by just importing       AUTOLOAD_ALL variable
    Select features on per-file basis       inject_compat method

    AUTOLOAD_ALL versus calling inject_compat:
        If you want to choose different features for different source
        files, you NEED TO:
            Add following TWO lines to the top of each source file
                from py23compat import inject_compat
                inject_compat()
            You can avoid SOME features across ALL source files by
            adding those features to ALWAYS_AVOID, and add ADDITIONAL
            features to avoid on a per-file basis by adding avoid=[...]
            to the inject_compat() call in each source file

            In this model, you NEED to add TWO lines to each source file,
            though you may not need the avoid=[..] in the call to
            inject_compat in all source files.

        If all your source files are in the SAME STAGE of Python-2-3
        compatibility / migration, you can:
            Disable some features across ALL source files using
            ALWAYS_AVOID
            Set AUTOLOAD_ALL = True
            Load remaining features in each source file by JUST importing
            py23compat using a line like:
                import py23compat     # noqa: F401

            '# noqa: F401' asks PEP8 not to complain about an unused import

            In this model, you CANNOT customize features on a per-file
            basis.

    The features avoided in the REPL are separate (ALWAYS_AVOID_REPL).
    Features are ALWAYS injected into the REPL by JUST importing py23compat.

is_int and is_str methods:
-------------------------
    Some modules (such as simplejson, but I am sure there are more), can
    RETURN objects of type 'unicode' or 'long' in Python2. If you have
    disabled 'unicode' and 'long' in Python2 (e.g. by having an empty
    ALWAYS_AVOID list), then you have no way to check if the returned
    valus is an instance of 'unicode' or 'long' respectively. In addition,
    in such cases, the returned variable will NOT be an instance of
    str | int respectively (although it's BEHAVIOR will be similar to those
    respective types).

    In such cases, you can use the is_str and is_int methods by importing
    them from this module. On Python2 is_int will match int and long and
    is_str will match str and unicode, while on Python3 is_int will match
    only int and is_str will match only str.

    You only need these methods if you would have otherwise used isinstance
    for this purpose.

Python version variables:
------------------------
    PY2-->boolean: Whether running in python2 (any minor version)
    PY3-->boolean: Whether running in python3 (any minor version)
    PYPY-->boolean: Whether running in pypy (any minor version)
    PY_MINOR-->int: Python minor version

Variables, imports and what is injected:
---------------------------------------
    REPL    Variable            import          Effect
    -----------------------------------------------------------------------
    YES     ALWAYS_AVOID_REPL   plain import    Except ALWAYS_AVOID_REPL
                                                is_int and is_str automatic

    YES     ALWAYS_AVOID_REPL   import +        Except ALWAYS_AVOID_REPL
            avoid param         inject_compat   avoid param has no effect
                                                is_int, is_str need import

    Source  ALWAYS_AVOID        plain import    Except ALWAYS_AVOID
                                                is_int, is_str need import

    Source  ALWAYS_AVOID        import +        Except ALWAYS_AVOID AND
            avoid param         inject_compat   except avoid param
                                                is_int, is_str need import
    -----------------------------------------------------------------------

    inject_compat ONLY injects compatibility names into:
        Caller's stack frame
        Top-most stack frame ONLY if running in REPL

    Importing this module ONLY injects compatibility names into:
        Top-most stack frame ONLY if running in REPL
        Into caller's (importer's) stack frame if AUTOLOAD_ALL is set

    When using the REPL, if ANY module IMPORTS py23compat, the
    compatibility names will be injected into the TOP-MOST stack
    frame (repeat: ONLY when using the REPL)

    Note the difference between the CALLER (importer) stack frame and
    the TOP-MOST stack frame.

Usage for NEW python packages starting from scratch:
---------------------------------------------------
    A. Make a COPY of this module (file) for EACH package - it has
       variables at the top that you CAN (and SHOULD) change to reflect
       the stage the package is in  (in terms of Python2-3 compatibility)

    B. For a NEW package started from scratch, I recommend:
        1. Keep ALWAYS_AVOID and ALWAYS_AVOID_REPL EMPTY
        2. Set AUTOLOAD_ALL = True
            Allows injecting compatibility code by JUST importing py23compat
            Still obeys ALWAYS_AVOID and ALWAYS_AVOID_REPL
            if you need them
        3. Write your package using (only) Python3 idioms and constructs.
          It should run unchanged in Python2 (need to pip install future)

Usage for making existing Python2 packages compatible with Python2-3:
--------------------------------------------------------------------

    A. Make a COPY of this module (file) for EACH package - it has
       variables at the top that you CAN (and SHOULD) change to reflect
       the stage the package is in  (in terms of Python2-3 compatibility)

    B. Make a list of all the changed features and obsoleted features
       being used by your package.
            See future_imports for NEW behavior in Python3
            See builtins_imports for CHANGED behavior
            See obsolete_imports for OBSOLETED classes, methods

    E. Use one of the following strategies. Note there are MANY possible
       strategies, and the python-future website has a much more robust
       and complete discussion of migration strategies.

        Feature-by-feature
            1. Start by adding ALL the features your package is using that has
               been changed or obsoleted in Python3 to ALWAYS_AVOID.

            2. If you regularly explore your package interactively using a REPL,
                add JUST the changed features (not the obsoleted features) to
                ALWAYS_AVOID_REPL also.

            3. Go through your package feature-by-feature and once a
                feature has been upgraded across your package, remove it
                from ALWAYS_AVOID and ALWAYS_AVOID_REPL.

            4. During this time, you can keep AUTOLOAD_ALL = True and
               JUST import py23compat at the top of each source file

            5. Once you have emptied ALWAYS_AVOID and ALWAYS_AVOID_REPL,
               your package should run unchanged in Python 2 and 3

        File-by-file
            1. Add two lines at the top of each source file:
                   from py23compat import inject_compat
                   inject_compat(avoid=xyz)
               where xyz is a set of features to disable for that file

            2. In each file, upgrade each disabled feature and then
               remove it from the avoid list

            3. Once the avoid list is empty, you can change the two lines
               at the top to be just:
                   import py23compat

            4. Once all files have been upgraded and have an empty avoid list,
               your package should run unchanged in Python 2 and 3

Known bug: interaction with pydoc on Python2:
--------------------------------------------
If using AUTOLOAD_ALL=True, AND 'unicode' is not in ALWAYS_AVOID,
pydoc2 (pydoc on python2) will fail for this module.
To use pydoc on py23compat with AUTOLOAD_ALL=True, do one of the
following:

  - Use pydoc3 in a python3 virtualenv
  - In a REPL, import py23compat and then do help(py23compat)
  - Call pydoc setting (shell) environment var PY23COMPAT_NO_AUTOLOAD)
  - Alias pydoc to 'PY23COMPAT_NO_AUTOLOAD=yes pydoc'

Note that PY23COMPAT_NO_AUTOLOAD disables effect of AUTOLOAD_ALL in
ALL conditions- not just for pydoc - there is no way to effect special
behavior when being imported by pydoc


More information on python-future and Python2-3 compatibility:
-------------------------------------------------------------
    See the python-future site for more information on writing python
    programs that can run in Python 2.x or 3.x, Python 2-->3 migration
    and the python-future package.

    Python-future Quick Start Guide: http://python-future.org/quickstart.html

    Idioms for writing Python 2-3 compatible code:
        http://python-future.org/compatible_idioms.html

    Importing explicitly:
    http://python-future.org/imports.html#explicit-imports

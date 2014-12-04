.. _sec-usage:

Usage
=====

pyChapel provides interoperability with Chapel in three forms:

1. Chapel code inlined in Python
2. Chapel code from source-files
3. Compile Chapel modules into Python modules

The following sections describe the different interoperability modes with
examples such that you can get a feeling of which mode might best serve your needs.

.. _subsec-inlined:

Inlined
~~~~~~~

If you read :ref:`sec-getting-started` you might be disappointed since you were
promised something more exiting than ``Hello, world``. Don't worry we will get
there after we've gone through the basics.

When using `inlined` mode you get access to Chapel by decorating functions with
the ``pych.extern.Chapel`` decorator and by writing those functions using certain
conventions.

.. literalinclude:: /examples/chapel.inline.helloworld.py
   :language: python

The Chapel code goes into the ``docstring`` of the decorated function.

.. note:: About scope: The notion of ``inlining`` might give the expectancy that the scope of the decorated function will be inherited such that globals and locals within the Python module might be accessible. However, this is not the case, only the variables passed as arguments will be available.

Inlining is meant to be used as a convenience where small chunks of Python can easily be re-written to more efficient Chapel code or simply get started with small snippets of Chapel within the comforts of Python.

The downside is that you lose syntax highlighting of Chapel code, and when the small snippet grows it becomes unmanagable. In those and what your motivation might be using :ref:`subsec-source-files` or :ref:`subsec-compiling-modules` might be preferable.

.. _subsec-source-files:

Source-files
~~~~~~~~~~~~

Instead of using the ``docstring`` of the decorated function a Chapel source-file can be used to used instead. Using either the ``bfile`` or the ``sfile`` argument to the decorator. 

using bfile
-----------

Using the ``bfile`` argument behaves exactly as the ``inline`` except the
function-body is taken from the given file.

.. literalinclude:: /examples/chapel.bfile.hello.py
   :language: python

The ``bfile`` target could contain something like below.

.. literalinclude:: /examples/bfiles/chapel/bfile.hello.chpl
   :language: chapel

It is not of consequence, but it is still worth noting that the Chapel source targeted with ``bfile`` most commonly will not be a valid Chapel program.  The example above is insidently valid, so to illustrate an example is provided
below where the ``bfile`` target is not a valid Chapel program.

Is this example the ``fibonacci`` function is decorated:

.. literalinclude:: /examples/chapel.bfile.fib.py
   :language: python

and function-body is taken from the provided ``bfile``:

.. literalinclude:: /examples/bfiles/chapel/bfile.fib.chpl
   :language: chapel

It not not of consequence, since pyChapel does not expect a valid Chapel program from ``bfile``, only a function body.

The ``bfile`` argument is simply meant as a means to provide function-bodies in
a form which might be slightly more convenient in some cases. If you want to use existing wellformed Chapel modules and functions then ``sfile`` or :ref:`subsec-compiling-modules` is the mode of interoperability your are looking for.

using sfile
-----------

``sfile`` maps Python functions to procedures in existing Chapel modules. This
allows you to work conveniently within your Chapel environment, with all the
joys that brings in terms of syntax highlighting, debugging and testing the module code, and
then expose the procedures you want access to in Python. 

Consequently the ``sfile`` argument expects the target to be a well-formed
Chapel module with a well-defined and exported procedure declaration.
In contrast to the ``docstring`` and ``bfile`` which only expects to be provided with a function body.

.. literalinclude:: /examples/chapel.sfile.hw.py
   :language: python

Where ``sfile.hello.chpl`` contains:

.. literalinclude:: /examples/sfiles/chapel/sfile.hello.chpl
   :language: chapel

The decorated Python function will map to a procedure within the Chapel module
using the function-naming conventions described in subsection
:ref:`subsec-conventions`.

.. _subsec-conventions:

Conventions and Arguments
~~~~~~~~~~~~~~~~~~~~~~~~~

pyChapel relies on naming conventions, and an unconventional use of
default arguments of Python functions. The following Chapel module will serve as an
example throughout this section.

.. literalinclude:: /examples/sfiles/chapel/hellolib.exported.chpl
   :language: chapel

function names
--------------

An ``@Chapel`` decorated Python function will map to a Chapel procedure of the same name.

.. note: In the case of ``inlined`` function bodies and ``bfile`` then this Chapel procedure is dynamically generated and compiled behind the scenes and thus never exposed to the pyChapel user.

Mapping a Python function to the Chapel procedure ``hello_caller`` from the
Chapel module ``HelloLib`` is done by writing and decorating the following
function:

.. literalinclude:: /examples/chapel.sfile.hellolib.py
   :language: python

Lets say that you insist of calling your ``@Chapel`` decorated Python function ``hello_world`` and you
refuse to rename, nor export the Chapel procedure under a different name, then you can map the Python function
using ``ename``:

.. literalinclude:: /examples/chapel.sfile.hellolib.ename.py
   :language: python

Real-world cases usually involve motivation other than stubborn unwillingness to
refactor code. Regardless of the motivation the ``ename`` decorator argument
serves the purpose of mapping functions when the naming convention is not
applicable.

default arguments
-----------------

The ``@Chapel`` decorated Python functions aren't really used as functions, they
are instead used a means to declare a foreign function within Python. You do not like the look of them do you? I know... but you just need to seem them in the right light.

Which is: See them as as foreign function-declarations instead of Python function definitions.

That worked right? Yeah, I know it did.

It is a unconventional use of Python function but it serves as much less verbose way to declare foreign functions in comparison to other ffi-libraries in Python.

.. _subsec-compiling-modules:

Compiling Chapel modules into Python modules
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The command-line utility ``pych`` is what you need.

.. code-block:: bash

  usage: pych [-h] [-c source_file | -k | -z | -s | -b | -t | -a | -v]

  Tool aiding the pych module

  optional arguments:
    -h, --help            show this help message and exit
    -c source_file, --compile source_file
                          Compile the given Chapel module into a Python module.
    -k, --check           Check the 'pych' installation, configuration, and
                          environment.
    -z, --testing         Run 'pych' testing.
    -s, --sfiles          Show information about source-files (sfiles).
    -b, --bfiles          Show information about function-body files (bfiles).
    -t, --templates       Show information about templates.
    -a, --object-store    Show information about the object-store (.so files).
    -v, --version         Print version


Compiling the Chapel module ``hellolib.exported.chpl``:

.. literalinclude:: /examples/sfiles/chapel/hellolib.exported.chpl
   :language: chapel

Into a Python module, is done by invoking:

.. code-block:: bash

   pych --compile hellolib.exported.chpl

Resulting in a Python module named ``a_out.py`` in the current working
directory. Rename the ``.py`` file:

.. code-block:: bash

   mv a_out.py hellolib.py

Such that it can be imported and used as:

.. code-block:: python

  from hellolib import hello_caller, add_ints

  if __name__ == "__main__":
    hello_caller()
    print add_ints(2, 3)


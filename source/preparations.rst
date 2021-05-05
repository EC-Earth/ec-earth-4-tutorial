Preparations
============

This section of the tutorial will explain how to get the |ece4| source code
and how to install the tools needed to build and run |ece4|.

.. important::

    |ece4| is not free or open source software! Some of the software
    components that make up |ece4| require the user to aquire a license.
    Furthermore, the use of |ece4| is restricted to EC-Earth consortium
    member institutions.

An account at the `EC-Earth Development Portal <https://dev.ec-earth.org>`_
is needed to access the |ece4| source code.

Checking out the EC-Earth 4 source code
---------------------------------------

The |ece4| source code is provided as a Subversion repository at the EC-Earth
Development Portal. To get the current version of the code, use your login
credentials and check out::

    > svn checkout https://svn.ec-earth.org/ecearth4/trunk ece-4

.. note::

    The name of the directory (:file:`ece-4` above) is just an example. You
    can chose any other directory, but :file:`ece-4` will be used throughout
    this tutorial.

    It is assumed from now on, that you are located within the :file:`ece-4`
    directory (or a subdirectory thereunder), unless noted otherwise.
    The current directory will be indicated by the prompt used in the code
    snippets.


Creating a Python virtual environment
-------------------------------------

ScriptEngine and it's dependencies are written in Python3 and they are
provided as Python packages. It is usually a good idea to install packages in
user space and a `Python virtual environment
<https://docs.python.org/3/tutorial/venv.html>`_ will be used in this
tutorial to install all requirements. For a detailed description of
ScriptEngine installation options, have a look at the `ScriptEngine
documentation
<https://scriptengine.readthedocs.io/en/latest/installation.html>`_

As a first step, check the Python version available by default::

    ece-4 > python --version
    Python 3.8.3

Make sure that the version reported is between 3.6 and 3.8 (inclusive), where
the patch level (the last part of the dot-separated version number) does not
matter.  On some systems, the default ``python`` executable may still point to
some Python2 version.  In this case, try ``python3 --version`` and use
``python3`` also in the next step.

Now, create a new virtual environment in the |ece4| directory with::

    ece-4> python -m venv .ECE4

which will create a :file:`.ECE4` directory for the environment.

.. note::

    Again, the name of the directory for the virtual environment
    (:file:`.ECE4`) is an example and any other name may be chosen.
    In the example above, a hidded directory (the name starting with a dot)
    is used in order to keep the |ece4| source and runtime directory clean.

The virtual Python environment is only *created* once, but it has to be
*activated* each time it is to be used, and also for each shell it is to be
used in. Activating the virtual environment is done by sourcing the
:file:`activate` script in the environment directory::

    ece-4> source .ECE4/bin/activate

after which, the prompt should change to show the activation status::

    (.ECE4) ece-4>

This prompt will be used in the examples of this tutorial, to keep reminding
that the environment must be activated.

.. hint::

    It may be convenient to create a symbolic link with ``ln -s
    .ECE4/bin/activate``, after which the environment can be activated by
    typing ``. activate`` in the :file:`ece-4` directory.

It is always a good idea to upgrade the Python package manager, :file:`pip`, in
a new virtual environment::

    (.ECE4) ece-4> pip install -U pip

after which the environment is ready for the installation of ScriptEngine.


Installing ScriptEngine
-----------------------

Since ScriptEngine is provided as a package at `PyPi <https://pypi.org>`_, it
can easily be installed with :file:`pip`::

    (.ECE4) ece-4> pip install scriptengine

Some of the runtime scripts use a particular ScriptEngine task package, so it
is best installed right away::

    (.ECE4) ece-4> pip install scriptengine-tasks-hpc

This completes the ScriptEngine installation. It can be tested with::

    (.ECE4) ece-4> se --version
    0.8.5

(Note that the version can differ, but it should not be lower than 0.8.5)


Installing the OCP-Tool
-----------------------

Download and install the OCP-Tool in the EC-Earth 4 virtual environment::

    (.ECE4) ece-4> cd ..
    (.ECE4) > git clone https://github.com/uwefladrich/ocp-tool
    (.ECE4) > cd ocp-tool

The code that adapts the OCP-Tool to |ece4| is currently developed in a
branch, which is checked out with::

    (.ECE4) ocp-tool> git checkout --track origin/refactor-for-se

Thereafter, the OCP-Tool is installed into the :file:`.ECE4` virtual
environment::

    (.ECE4) ocp-tool> pip install -e .

.. note::

    The installation of the OCP-Tools is still a bit difficult at the moment,
    because it is still very much under development. The last three steps could
    be much simplified by providing an OCP-Tool package at Pypi and this will be
    considered in the future.

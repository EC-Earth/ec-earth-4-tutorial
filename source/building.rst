Building the EC-Earth 4 components
==================================

.. _ScriptEngine documentation:  https://scriptengine.readthedocs.io

Overview
--------

This section will cover how the |ece4| components are built (i.e. compiled). It
is assumed that the |ece4| source code has been checked out, ScriptEngine and
the OCP-Tools are installed and the directory structure is created according to
the previous section.

The source code of the |ece4| components resides in subdirectory of `sources`::

    (.ECE4) ece-4> ls -1 sources/
    amip-forcing/
    lpjg/
    nemo-4.0.1/
    oasis3-mct-4.0/
    oifs-43r3/
    runoff-mapper/
    se/
    tm5mp/
    util/
    xios-2.5/

Since this tutorial covers only the AMIP and GCM configurations, only OpenIFS
(``oifs-43r3``), NEMO (``nemo-4.0.1``), OASIS (``oasis3-mct-4.0``), XIOS
(``xios-2.5``), the Runoff-mapper (``runoff-mapper``) and the
AMIP-Forcing-reader (``amip-forcing``) are relevant. The ScriptEngine scripts
needed to built the components are located in the ``se`` subdirectory:

.. code-block:: bash

    (.ECE4) ece-4> cd sources/se
    (.ECE4) se> ls -1
    platforms/
    templates/
    compile-all.yml
    compile-amipfr.yml
    compile-nemo.yml
    compile-oasis.yml
    compile-oifs.yml
    compile-rnfm.yml
    compile-xios.yml
    user-settings.yml

The directory contains a number of YAML scripts, which are passed to
ScriptEngine and will control the user and platform configuration, the
preparation of makefiles and the like, and finally trigger the compilation of
the components. Detailed information about how ScriptEngine works with YAML
scripts can be found in the `ScriptEngine documentation`_. For this tutorial it suffices to
mention that multiple scripts can be passed to the ScriptEngine ``se``
executable and are processed in order:

.. code-block:: bash

    (.ECE4) se> se first.yml second.yml third.yml 
    2021-05-07 08:55:23 INFO [se.cli] Logging configured and started
    2021-05-07 08:55:23 INFO [se.task:echo <60da064106>] Hello from firs...
    Hello from first script!
    2021-05-07 08:55:23 INFO [se.task:echo <fc27e03e41>] Hello from seco...
    Hello from second script!
    2021-05-07 08:55:23 INFO [se.task:echo <2206bc645d>] Hello from thir...
    Hello from third script!

For a brief overview of ``se`` options and arguments, run ``se --help``.

.. note:: The actual names of all scripts are just convenient names. There is
          nothing special about them and they could be changed at will. Scripts
          could also be split or merged. As far as ScriptEngine is concerned, it
          will just process, in order, all scripts given as arguments.

The basic approach for building the |ece4| components, will be to call the
scripts for user settings, the platform settings and the actual compilation
commands in one go::

    (.ECE4) se> se user-settings.yml platforms/<MY_PLATFORM>.yml compile-<COMPONENT>.yml
    
.. note:: It is important to always include the user and platform settings
          scripts in every compilation! The settings are not magically stored
          between the ScriptEngine runs (other than in the scripts) and must
          always be included!

The order in which the |ece4| components must be compiled (due to
inter-component dependencies) is

- OASIS
- XIOS
- OpenIFS, NEMO, AMIP-Forcing-reader, Runoff-mapper (in any order)

.. hint:: There is also a ``compile-all.yml`` script, which compiles all
          components in the correct order. The user and platform settings still
          have to be provided.


User settings
-------------

The ``user-settings.yml`` file is presently very simple and basically provides
the path of the |ece4| source code::

    # User-dependent configuration for EC-Earth 4 build
    - base.getenv:
        home: HOME
    - base.context:
        main:
            base_dir: "{{home}}/Projects/ece-4"
            src_dir: !noparse "{{main.base_dir}}/sources"


.. warning:: The syntax of Python, YAML, Markdown and some other modern
             languages and formats is based on indentation. Tab characters can
             not provide consistent indentation, and it has therefore become a
             major offence to use tab characters in source code!
             Make sure that your editor replaces tab characters with spaces and
             that you follow the indentation convention (i.e. how many spaces
             per level) of the source code you are working with.

The ``user-settings.yml`` example takes help of the ``HOME`` environment
variable to define the |ece4| directories, but that is just for convenience.
Again, for details regarding the specific syntax used in the configuration
definition, refer to the `ScriptEngine documentation`_, particularly the
`Writing Scripts section
<https://scriptengine.readthedocs.io/en/latest/scripts.html>`_ therein.

For the course of the tutorial, it is important that the ``main.base_dir``
configuration parameter points to the correct directory.


Platform settings
-----------------

The settings for different platforms are organised in the ``platforms/``
subdirectory, similar to |ece3|. The platform configuration is naturally a bit
heavier than the user settings and comprises things like compiler and library
settings, preprocessor macros and other details.

Compared to |ece3|, the configuration is more structured, making use of
dictionary structure in YAML, For example, this is how the compilers could be
configured for a particular platform::

    - base.context:
        build:
            lang:
                fortran:
                    compiler: mpif90
                    flags: -g -O2 -fdefault-real-8 -fdefault-double-8 -ffree-line-length-none
                    preprocessor: !noparse "{{build.lang.c.preprocessor}}"
                c:
                    compiler: mpicc
                    flags: -g
                    preprocessor: cpp
                linker:
                    command: !noparse "{{build.lang.fortran.compiler}}"
                make:
                    command: gmake

Assuming the above configuration, the Fortran compiler could be referred to in
any makefile as ``{{build.lang.fortran.compiler}}`` and the make command as
``{{build.lang.make.command}}``.

As another example, the section to configure some needed libraries could look
like::

    - base.context:
        build:
            libs:
                mpi: null
                lapack:
                    libs: [openblas]
                grib:
                    base_dir: /software/sse/manual/eccodes/2.8.2/nsc1-gcc-2018a-eb
                    inc_dir: !noparse "{{build.libs.grib.base_dir}}/include"
                    lib_dir: !noparse "{{build.libs.grib.base_dir}}/lib"
                    libs: [eccodes_f90, eccodes, pthread]
                netcdf:
                    mod_dir: /software/sse/manual/netcdf/4.4.1.1/nsc1-gcc-2018a-eb/include
                    libs: [netcdff, netcdf]
                hdf5: null

Again, the configuration parameters are available for later use in a structured
way, such as ``{{build.libs.grib.lib_dir}}``.

Some details are worth further explanation in the above example: The syntax
``mpi: null`` and ``hdf5: null`` is used in YAML to denote an empty value. Thus,
there is no special MPI or HDF5 configuration on this platform (configuration
provided by modules).

Second, the ``*.libs`` parameters must be defined as YAML lists (with brackets
and commas), even if there is only one library.

.. hint:: If you want to test user or platform setting scripts, it is possible
          to run these with ScriptEngine without actual compilation:

          ``> se user-settings.yml platforms/<MY_PLATFORM>.yml``

          i.e. just omit the compile script(s). Thus, you can make sure the
          setting scripts are syntactically correct before you attempt the
          actual compilation.

Generally, it will be easiest to start from a known-to-work platform file, copy
and adapt to the platform.


Building
--------

It is advised to start by build the |ece4| components one at a time. For the
AMIP configuration, this could be done by:

.. code-block:: bash

    (.ECE4) se> se user-settings.yml platforms/<MY_PLATFORM>.yml compile-oasis.yml
    (.ECE4) se> se user-settings.yml platforms/<MY_PLATFORM>.yml compile-xios.yml
    (.ECE4) se> se user-settings.yml platforms/<MY_PLATFORM>.yml compile-oifs.yml
    (.ECE4) se> se user-settings.yml platforms/<MY_PLATFORM>.yml compile-amipfr.yml
    
and for the GCM configuration by:

.. code-block:: bash

    (.ECE4) se> se user-settings.yml platforms/<MY_PLATFORM>.yml compile-oasis.yml
    (.ECE4) se> se user-settings.yml platforms/<MY_PLATFORM>.yml compile-xios.yml
    (.ECE4) se> se user-settings.yml platforms/<MY_PLATFORM>.yml compile-oifs.yml
    (.ECE4) se> se user-settings.yml platforms/<MY_PLATFORM>.yml compile-nemo.yml
    (.ECE4) se> se user-settings.yml platforms/<MY_PLATFORM>.yml compile-rnfm.yml

When this approach works properly, you can compile the configuration in one go,
either by adding all compilation scripts at once:

.. code-block:: bash

    (.ECE4) se> se user-settings.yml \
                   platforms/<MY_PLATFORM>.yml \
                   compile-oasis.yml \
                   compile-xios.yml \
                   compile-oifs.yml \
                   compile-nemo.yml \
                   compile-rnfm.yml

or by using the convenience script ``compile-all.yml``:

.. code-block:: bash

    (.ECE4) se> se user-settings.yml platforms/<MY_PLATFORM>.yml compile-all.yml

The ``compile-all.yml`` script compiles *all* components (both for AMIP and GCM
configurations) and allows also to configure the build process by setting
``build.clean`` (clean all sources and compile from scratch) and ``build.jobs``
(the number of parallel processes to be used)::

    - base.context:
        build:
          clean: yes
          jobs: 10
        components:
          - oasis
          - xios
          - oifs
          - nemo
          - rnfm
          - amipfr
    - base.include:
        src: "compile-{{component}}.yml"
      loop:
        with: component
        in: "{{components}}"

Note how ``compile-all.yml`` is just a top-level script that is using the other
``compile-*.yml`` scripts under the hood.

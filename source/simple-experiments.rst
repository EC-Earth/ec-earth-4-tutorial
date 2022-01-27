Running simple tests
====================

.. warning::
    Running even simple experiments with |ece4| is a more diverse task than
    building the model. There are more dependencies to be fulfilled (w.r.t. user
    and computational environment) and more choices to be made (w.r.t.
    experiment configuration). Hence, the following part of the tutorial needs
    probably more adaptation to your needs, compared with the previous.

    Furthermore, a number of choices or features may be hard-coded in the
    scripts, or not yet supported at all. This will change as development of
    |ece4| and this tutorial progresses.

To prepare for a simple test experiment, we start from the ScriptEngine scripts
provided in ``runtime/se`` and subdirectories:

.. code-block:: bash

    (.ECE4) ece-4 > cd runtime/se
    (.ECE4) se > ls -1
    example.yml
    example-slurm.yml
    scriptlib/
    templates/

The ScriptEngine runtime environment (SE RTE) is split into separate YAML
scripts, partly with respect to the model component they are dealing with, and
partly with respect to the runtime stage they belong to. This is done in order
to provide a modular approach for different configurations and avoid overly
complex scripts and duplication. Most of the YAML scripts are provided in the
``scriptlib`` subdirectory.

However, this splitting is not "build into" ScriptEngine or the SE RTE, it is
entirely up to the user to adapt the scripts for her needs, possibly splitting
up the scripts in vastly different ways.


Main structure of the run scripts
---------------------------------

In order to use and control the modular YAML scripts provided under
``scriptlib``, the user has to provide a top level run script. To make this task
easier, an example script (``example.yml``) is provided. The ``example.yml``
script is rather simplistic. It allows for configuring of some basic experiment
settings, but it does in particular not provide control over the batch job settings
used for the experiment run (only the number of MPI processes per component).
Hence, a slightly more complicated example script, ``example-slurm.yml``, is provided,
which allows for the configuration of further batch job details. However, the two
example scripts share most of the configuration details explained below.

The top level run script defines a number of configuration parameters before it
calls ``scriptlib/main.yml`` (using the ScriptEngine ``base.include`` task),
which controls the actual model execution phases.

Exactly which configuration parameters are included in the top level script is
up to the user and should be adapted to the experiment needs. The example script
defines very basic settings, like the experiment ID, the experiment schedule,
the model components, and a few more.

As an example, this is how the experiment ID and a description are defined::

    base.context:
    main:
        experiment_id: TEST
        experiment_description: |
            This is an example run of the ECE4 GCM

There could be more configuration parameters put in this script, if they are
expected to change with the experiment set-up. This means that configuration
parameter definitions could be moved here from ``scriptlib/main.yml`` (or any
script called from there). For the purpose of this tutorial, this simple
approach will suffice.

The top level script defines a list of components (``main.components``)::

    - base.context:
        main:
            components:
                - oifs
                - nemo
                - oasis
                - xios
                - rnfm


This is for the GCM configuration (OpenIFS, NEMO, OASIS, XIOS, runoff-mapper).
The corresponding list for the AMIP configuration (OpenIFS, OASIS, XIOS,
AMIP-Forcing-reader) would be::

    - base.context:
        main:
            components:
                - oifs
                - oasis
                - xios
                - amipfr

The main structure of ``scriptlib/main.yml`` proceeds then as follows::

    # Configure 'main' and all components
    - base.include:
        src: "scriptlib/config-{{component}}.yml"
        ignore_not_found: yes
      loop:
        with: component
        in: "{{['main'] + main.components}}"

    # On first leg: setup 'main' and all components
    # ...

    # Pre step for 'main' and all components
    # ...

    # Start model run for leg
    # ...

    # Run post step for all components
    # ...

    # Monitoring
    # ...

    # Re-submit
    # ...

Basically, the run script defines the following stages:

#. ``config-*``, which sets configuration parameters for each component.
#. ``setup-*``, which runs, for each component, once at the beginning of the experiment.
#. ``pre-*``, which runs, for each component, at each leg before the executables.
#. ``run``, which starts the actual model run (i.e. the executables).
#. ``post-*``, which is run, for each component, at each leg after the model run has completed.
#. ``resubmit``, which submits the model for the following leg.
#. ``monitor``, which prepares data for on-line monitoring.

Not every stage has to be present in each model run, and not all stages have to
be present for all components. For all stages and components that are present,
there is a corresponding ``scriptlib/<stage>-<component>.yml`` script, which is
included (via the ``base.include`` ScriptEngine task). Hence, the main
implementation logic of ``main.yml`` is to go through all stages and execute all
component scripts for that stage, if they exist.

Note that there is an artificial model component, called ``main``, which is
executed first in all stages. The corresponding ``scriptlib/<stage>-main.yml``
files includes tasks that are general and not associated with a particular
component of the model.


Running batch jobs from ScriptEngine
------------------------------------

ScriptEngine can send jobs to the SLURM batch system when the
``scriptengine-tasks-hpc`` package is installed, as described in  the
:ref:`Preparations` section. Here is an example of the ``hpc.slurm.sbatch`` task
in ``example.yml``::

    # Submit batch job
    - hpc.slurm.sbatch:
        account: my_slurm_account
        nodes: 14
        time: !noparse 0:30:00
        job-name: "ece-4-{{main.experiment_id}}"
        output: job.out
        error: job.out

What this task does is to run the entire ``se`` command, including all scripts
given at the command line, as a batch job with the given arguments (e.g.
account, number of nodes, and so on).

As a simplified example, a ScriptEngine script such as::

    - hpc.slurm.sbatch:
        account: my_slurm_account
        nodes: 1
        time: !noparse 0:30:00
    - base.echo:
        msg: Hello from batch job!

would in the first place submit a batch job and then stop. When the batch job
executes, the first task (``hpc.slurm.sbatch``) would execute again, but do
nothing because it already runs in a batch job. Then, the next task
(``base.echo``) would be executed, writing the message to standard output in the
batch job.

.. important:: The ``scriptengine-tasks-hpc`` package provides only support for
            SLURM at the moment. Support for the PBS scheduler is envisaged, but
            hasn't been implemented due to some peculiarities of the ``qsub``
            command. Since SLURM is the prevalent job scheduler on most HPC
            systems (with the noteable exception of the ECMWF cca/b systems), it
            is at the moment unclear if PBS support can be prioritised any time
            soon. For any input on this issue, please check out related issues at
            the `scriptengine-tasks-hpc Github repository
            <https://github.com/uwefladrich/scriptengine-tasks-hpc>`_.


The experiment schedule
-----------------------

ScriptEngine supports recurrence rules (rrules, `RFC 5545
<https://datatracker.ietf.org/doc/html/rfc5545>`_) via the Python `rrule
module <https://dateutil.readthedocs.io/en/stable/rrule.html>`_ in order to
define schedules with recurring events.

This is used in the SE RTE to specify the experiment schedule, with start date,
leg restart dates, and end date. This allows a great deal of flexibility when
defining the experiment, allowing for irregular legs with restarts at almost any
combination of dates.

.. warning:: Event though rrules provide a lot of flexibility for the experiment
            schedule, it is not certain that all parts of the SE RTE and the
            model code can deal with arbitrary start/restart dates. This feature
            is provided in order to not limit the definition of a schedule at a
            technical level in the RTE.

A simple schedule with yearly restarts could look like::

    - base.context:
        schedule:
            all: !rrule >
                DTSTART:19900101
                RRULE:FREQ=YEARLY;UNTIL=20000101

which would define the start date of the experiment as 1990-01-01 00:00 and
yearly restart on the 1st of January until the end date 2000-01-01 00:00 is
reached, i.e. 10 legs.

As another example, two-year legs from 1850 until 1950 would be defined as::

    - base.context:
        schedule:
            all: !rrule >
                DTSTART:18500101
                RRULE:FREQ=YEARLY;INTERVAL=2;UNTIL=19500101


The ``run.sh`` template
-----------------------

The start of the model component executables in the appropriate MPI environment
is handled via a short shell script that is produced from a template. This
happens in the ``scriptlib/setup-main.yml`` script::

    - base.template:
        src: run-gcc+ompi.sh.j2
        dst: run.sh

which picks the given run script template (``run-gcc+openmpi.sh.j2`` in this
case) from the ``templates/`` directory, runs it through Jinja2, and places the
resulting script under the name ``run.sh`` in the run directory. From there, it
is later started in the "run" stage by ``scriptlib/run.yml``::

    - base.command:
        name: sh
        args: [run.sh]

There are, at the moment, a number of platform dependencies hidden in the run
script template and the whole process is still under development in order to
provide a robust and portable mechanism to start the MPI processes. One idea is
to support starting MPI processes directly from a ScriptEngine task in
``scriptengine-tasks-hpc``.


Initial data
------------

The directory with initial data for |ece4| is configured in
``example.yml``::

    - base.context:
          main:
              # ...
              inidir:  /path/to/your/initial-data

For now, the set if initial data can be downloaded from the SMHI Publisher at
NSC, the link is given on the |ece4| `Tutorial Wiki page`_ at the EC-Earth
Development Portal.

.. note:: An account is needed to access the EC-Earth Development Portal,
        because the information is restricted to EC-Earth consortium member
        institutes.

.. _Tutorial Wiki page: https://dev.ec-earth.org/projects/ec-earth-4/wiki/EC-Earth_4_Tutorial


Minimal set of changes
----------------------

In the simplest case, only few things have to be changed in ``example.yml`` (or
the corresponding top level script provided by the user) in order to run a
simple experiment:

- ``main.inidir``
- ``nemo.initial_state``
- ``main.schedule``
- possibly adaptations in the run script template
- batch job details in the batch submission task

Once all changes are made, a run can be started by:

.. code-block:: bash

    (.ECE4) se > se example.yml


Changing the OpenIFS grid
-------------------------

Using the extended version of the OCP-Tools (see :ref:`Installing the OCP-Tool`)
allows to select one of the predefined grids in ``example.yml``::

  - base.context:
        oifs:
            select_grid: !noparse_jinja "{{oifs.grids.TCO95L91}}"

The list of supported grids can be found in ``scriptlib/config-oifs.yml``,
together with the corresponding number of vertical levels and time steps.
Note that the time step settings for different grids have not yet been
thoroughly tested, and relies on `ECMWF recommendations`_.

The OpenIFS grid can be chosen for both AMIP and GCM configurations, the
OCP-Tool extensions will set up the correct combination of OpenIFS, NEMO,
runoff-mapper, or AMIP-forcing-reader grids, as appropriate.

However, note that the remapping weights for OASIS3-MCT are computed at the
start-up of the first leg! This will take some extra computing time,
particularly for higher resolution. Work is ongoing to configure the weight
computation in a way that allows parallelisation, thereby reducing the
substantial overhead.

.. warning:: When choosing the OpenIFS grid, the time step is solely based on
            the ECMWF recommendations. No automatic adaptation of the time step
            to the coupling time step is done for GCM configurations! This means
            that if OpenIFS, NEMO and coupling time step are not consistent, the
            model with most likely crash.

.. _ECMWF recommendations: https://confluence.ecmwf.int/display/OIFS/4.3+OpenIFS%3A+Horizontal+Resolution+and+Configurations

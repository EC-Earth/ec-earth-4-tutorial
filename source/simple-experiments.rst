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
    scripts/
    templates/
    TEST.yml
    ece-4.yml

The ScriptEngine runtime environment (SE RTE) is split into separate YAML
scripts, partly with respect to the model component they are dealing with, and
partly with respect to the runtime stage they belong to. This is done in order
to provide a modular approach for different configurations and avoid overly
complex scripts and duplication.

However, this splitting is not "build into" ScriptEngine or the SE RTE, it is
entirely up to the user to adapt the scripts for her needs, possibly splitting
up the scripts in vastly different ways.


Main structure of the run scripts
---------------------------------

Two top level scripts are present in the ``runtime/se`` directory: ``TEST.yml``
and ``ece-4.yml``. The logic of the SE RTE follows the same as for the building
process, namely that ScriptEngine can process multiple scripts, in order, given
at the command line. Thus, ``TEST.yml`` is an example script that defines a few
experiment specific configuration parameters, while ``ece-4.yml`` contains the
main logic of the run scripts.

Looking at ``TEST.yml`` first, we find just two configuration parameters::

    base.context:
    main:
        experiment_id: TEST
        experiment_description: |
            This is an example run of the ECE4 GCM

There could be more configuration parameters put in this script, if they are
expected to change with the experiment set-up. This means that configuration
parameter definitions could be moved here from ``ece-4.yml`` (or any script
called from there). For the purpose of this tutorial, this simple approach will
suffice.

The main top-level ``ece-4.yml`` script defines a list of components
(``main.components``)::

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

The main structure of ``ece-4.yml`` proceeds then as follows::

    # Configure 'main' and all components
    - base.include:
        src: "scripts/config-{{component}}.yml"
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
there is a corresponding ``scripts/<stage>-<component>.yml`` script, which is
included (via the ``base.include`` ScriptEngine task). Hence, the main
implementation logic of ``ece-4.yml`` is to go through all stages and execute
all component scripts for that stage, if they exist.

Note that there is an artificial model component, called ``main``, which is
executed first in all stages. The corresponding ``scripts/<stage>-main.yml``
files include tasks that are general and not associated with a particular
component of the model.

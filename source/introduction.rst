
Introduction
============

|ece4| is the next generation climate model developed by the `EC-Earth
consortium <http://www.ec-earth.org>`_. This tutorial explains how to get
and prepare the model source code, build the model, and how to run simple
technical tests.

It is important to realise that |ece4| is still at an experimental stage, and
so is this tutorial. The model code, as well as the build and run-time
environment will frequently change. Features will be missing and platforms
may not yet be supported. An attempt is made to keep this tutorial
up-to-date, which means that details will change often and without prior
notice. This includes in particular URLs to repositories and references to
branches, steps to build or run the model, and supported features. It is
therefore advised to double check the latest on-line version of the tutorial
in case of problems.

The experimental state of development also implies that some of the steps are
more complicated than they ought to be. Whenever this is the case, the tutorial
mentions this and improvements can be expected.

The sources for this tutorial (not the model code!) are `hosted at Github
<https://github.com/uwefladrich>`_ and automatically `published to Read the
Docs <https://ece-4-tutorial.readthedocs.io>`_ on every change. If there are
problems with the tutorial as such, please `open an issue
<https://github.com/uwefladrich/issues>`_ in the Github project or consider
contributing to the tutorial with `a pull request`_.

- corresponding ECE Dev Portal issue/wiki page
  (complementing information)

This tutorial is based on and covers:

- the trunk version of |ece4|, including OpenIFS 43r3v1 and NEMO 4.0.1
- the `ScriptEngine <https://scriptengine.readthedocs.io>`_ build and runtime environment
- the OCP-Tool for automatic creation of coupling information
- AMIP and GCM configurations


.. note::

    It is possible to prepare Makefiles, configurations and run scripts
    manually, thus building and running |ece4| without using ScriptEngine.
    However, this is not covered in this tutorial.


.. _a pull request: https://docs.github.com/en/github/collaborating-with-issues-and-pull-requests/proposing-changes-to-your-work-with-pull-requests

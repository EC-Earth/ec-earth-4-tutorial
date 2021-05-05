
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
therefore adviced to double check the latest online version of the tutorial
in case of problems.

- currently some steps are more complicated than they should be

The sources for this tutorial (not the model code!) are `hosted at Github
<https://github.com/uwefladrich>`_ and automatically `published to Read the
Docs <https://ece-4-tutorial.readthedocs.io>`_ on every change. If there are
problems with the tutorial as such, please `open an issue
<https://github.com/uwefladrich/issues>`_ in the Github project or consider
contributing to the tutorial with `a pull request`_.

- corresponding ECE Dev Portal issue/wiki page
  (complementing information)

This tutorial is based on `ScriptEngine
<https://scriptengine.readthedocs.io>`_ for building |ece4| and making simple
test runs, and instructions how to install ScriptEngine are therefore included.

.. note::

    It is possible to manually prepare Makefiles, configurations and run
    scripts, thus building and running |ece4| without using ScriptEngine.
    However, this is not the topic of this tutorial.


.. _a pull request: https://docs.github.com/en/github/collaborating-with-issues-and-pull-requests/proposing-changes-to-your-work-with-pull-requests

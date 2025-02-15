.. _parsing-methods:

Parsing Methods
===============

Cosmos offers several options to parse your dbt project:

- ``automatic``. Tries to find a user-supplied ``manifest.json`` file. If it can't find one, it will run ``dbt ls`` to generate one. If that fails, it will use Cosmos' dbt parser.
- ``dbt_manifest``. Parses a user-supplied ``manifest.json`` file. This can be generated manually with dbt commands or via a CI/CD process.
- ``dbt_ls``. Parses a dbt project directory using the ``dbt ls`` command.
- ``custom``. Uses Cosmos' custom dbt parser, which extracts dependencies from your dbt's model code.

There are benefits and drawbacks to each method:

- ``dbt_manifest``: You have to generate the manifest file on your own. When using the manifest, Cosmos gets a complete set of metadata about your models. However, Cosmos uses its own selecting & excluding logic to determine which models to run, which may not be as robust as dbt's.
- ``dbt_ls``: Cosmos will generate the manifest file for you. This method uses dbt's metadata AND dbt's selecting/excluding logic. This is the most robust method. However, this requires the dbt executable to be installed on your machine (either on the host directly or in a virtual environment).
- ``custom``: Cosmos will parse your project and model files for you. This means that Cosmos will not have access to dbt's metadata. However, this method does not require the dbt executable to be installed on your machine.

If you're using the ``local`` mode, you should use the ``dbt_ls`` method.

If you're using the ``docker`` or ``kubernetes`` modes, you should use either ``dbt_manifest`` or ``custom`` modes.


``automatic``
-------------

When you don't supply an argument to the ``load_mode`` parameter (or you supply the value ``"automatic"``), Cosmos will attempt the other methods in order:

1. Use a pre-existing ``manifest.json`` file (``dbt_manifest``)
2. Try to generate a ``manifest.json`` file from your dbt project (``dbt_ls``)
3. Use Cosmos' dbt parser (``custom``)

To use this method, you don't need to supply any additional config. This is the default.

``dbt_manifest``
----------------

If you already have a ``manifest.json`` file created by dbt, Cosmos will parse the manifest to generate your DAG.

You can supply a ``manifest_path`` parameter on the DbtDag / DbtTaskGroup with a path to a ``manifest.json`` file.

To use this:

.. code-block:: python

    DbtDag(
        project_config=ProjectConfig(
            manifest_path="/path/to/manifest.json",
        ),
        render_config=RenderConfig(
            load_mode=LoadMode.DBT_MANIFEST,
        )
        # ...,
    )

``dbt_ls``
----------

.. note::

    This only works for the ``local`` execution mode.

If you don't have a ``manifest.json`` file, Cosmos will attempt to generate one from your dbt project. It does this by running ``dbt ls`` and parsing the output.

When Cosmos runs ``dbt ls``, it also passes your ``select`` and ``exclude`` arguments to the command. This means that Cosmos will only generate a manifest for the models you want to run.

To use this:

.. code-block:: python

    DbtDag(
        render_config=RenderConfig(
            load_mode=LoadMode.DBT_LS,
        )
        # ...,
    )


``custom``
----------

If the above methods fail, Cosmos will default to using its own dbt parser. This parser is not as robust as dbt's, so it's recommended that you use one of the above methods if possible.

The following are known limitations of the custom parser:

- it does not read from the ``dbt_project.yml`` file
- it does not parse Python files or models

To use this:

.. code-block:: python

    DbtDag(
        render_config=RenderConfig(
            load_mode=LoadMode.CUSTOM,
        )
        # ...,
    )

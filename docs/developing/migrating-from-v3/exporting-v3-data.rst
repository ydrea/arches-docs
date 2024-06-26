#######################
Export v3 Business Data
#######################

In your v3 command line run::

    python manage.py v3v4 -o export-resources --format JSON

You will get a console update during the process, which could take a few minutes. The result will be one file:

+ ``v3resources-all-<date>.json``

Place the file(s) somewhere easy to access.

.. important::

    If you have a very large database (maybe 25k+ resources), we recommend using ``--format JSONL``. This will create a `JSON Lines <http://jsonlines.org/>`_ file, which requires minimal memory resources. Exporting the entire database to a single JSON file can crash servers without enough memory. For even more control over the export, add ``--split`` to the command above. One JSON/JSONL file will be created **per resource type**. This is extremely helpful for debugging migration issues.

Export v3 Resource Relations
----------------------------

In your v3 command line run::

    python manage.py v3v4 -o export-relations

You will get a console update during the process, which could take a few minutes. The result will be one file:

+ ``v3relations-all-<date>.csv``

Place the file somewhere easy to access.

Export v3 Reference Data
------------------------

In your v3 command line run::

    python manage.py v3v4 -o export-skos --name Arches

The result will be one file:

+ ``v3scheme-arches-<date>.xml``

Place the file somewhere easy to access. This is the "Arches" scheme from your RDM, which is, typically, where your entire concept set will exist. If you are using a different concept scheme, subsitute its name for "Arches" in the command above.

.. warning::

    You are only able to migrate one scheme. If your v3 dropdown lists are composed of concepts from two different schemes (i.e. you added another scheme alongside "Arches", added concepts to it, and then added those concepts to dropdown lists) you'll need to manually consolidate these schemes into one before exporting.

    **Dropdown Lists themselves are not migrated, they are recreated in v4 based on Top Concepts.**

Transfer all v3 uploaded media files
------------------------------------

You must move all of the media files that have been uploaded to your v3 deployment to your v4 project.

By default, the directory in your new v4 project should be called ``my_project/my_project/uploadedfiles``. If this directory doesn't exist, create it, and move all of the v3 media into it.

.. admonition:: AWS S3 and Azure Users

    You should be able to continue using the same storage bucket, and just point your v4 project at it. Just make sure your content is in a folder called ``uploadedfiles``. **In theory this should work, but we haven't tested it.**

----

Now that you have exported all of the data you need from your v3 deployment, head back to :ref:`Migrating Your Data`.
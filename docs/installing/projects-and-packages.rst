######################
Understanding Projects
######################

Arches Projects facilitate all of the customizations that you will need to make one installation of Arches different from the next. You can update HTML or CSS to modify web page branding, and add functions, datatypes, and widgets to introduce new functionality. A project sits outside of your virtual environment, and can thus be transferred to any other system where Arches is installed.

To create a project, see :ref:`Creating a new Arches Project` in the installation guide.

Project Structure
-----------------

The general structure of a new Arches project is:::

	my_project/
	└─ manage.py
	└─ my_project/
	      └─ settings.py
	      └─ datatypes/
	      └─ functions/
	      └─ media/
	      └─ templates/
	      └─ widgets/

*Not all files are shown*

.. important:: At this level, "projects" are completely different from the mobile data collection "projects" that are mentioned elsewhere in this documentation.

settings.py
^^^^^^^^^^^^

Many project-specific settings are defined here. You should use ``settings_local.py`` to store variables that you may want to keep out of the public eye (db passwords, API keys, etc.).

templates
^^^^^^^^^^^

This directory holds HTML templates that you can modify to customize the branding and general appearance of your project.

datatypes, functions, and widgets
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

These directories will store the custom extensions that you can create for the project. Developers interested in pursuing these customizations should start with :ref:`Creating Extensions`.

######################
Understanding Packages
######################

A package is an external collection of Arches data (resource models, business data, concepts, collections) and customization files (widgets, datatypes, functions, system settings) that you can load into an Arches project.

	No new user starts wit loading a package!

Loading a Package
-----------------

To load a package simply run the load_package command using your \*project's manage.py file:

.. code-block:: bash

    python manage.py packages -o load_package -s https://github.com/package/archive/branch.zip -db

	Are any of these destructive?
-db    `true` to run setup_db to rebuild your database. default = 'false'
-ow    `overwrite` to overwrite concepts and collections. default = 'ignore'

-st    `stage` to stage concepts and collections. default = 'stage'

	This requires at least a link to "load to github"!
-s     a path to a zipfile located on github or locally

	Operaations?
-o     operation name
-y     accept defaults (will overwrite existing branches and system settings with those in the package)

	What does this mean: "object's regular `save` method"?
-bulk  uses `bulk_save` methods which run faster but don't call an object's regular `save` method
-dev	 loads three test users

	How do I decide on this?
If you do not pass the `-db True` to the load_package command, your database will not be recreated. 

	Id?
If you already have resource models and branches with the same id as those you are importing, you will be prompted to confirm whether you would like to keep or overwrite each model or branch.

If you pass the `-bulk` argument, know that any resource instances that rely on functions to dynamically create/edit tiles will not be called during package load. Additionally, some logging statements may not print to console during import of reference data. Whereas the default `save` methods create an edit in the edit history for each individual tile created, `-bulk` will instead create a single edit for all tiles, of type: "bulk_create". Resource creation will still be individually saved to edit history.

.. note:: It is important to note that you cannot load a package directly into core Arches. Packages must be loaded into a project.

If you are a developer running the latest arches you probably want to create a project with a new Arches installation. This ensures that the `arches_project create` command uses the latest project templates.

    #. Uninstall arches from your virtualenv

        .. code-block:: bash

            pip uninstall arches

    #. Navigate into arches root folder delete the `build` directory

    #. Reinstall arches

        .. code-block:: bash

            python setup.py install
            python setup.py develop

    #. Navigate to where you want to create your new project and run:

        .. code-block:: bash

            arches-project create mynewproject

        .. note:: You can use the option ``[{-d|--directory} <directory_name>]`` to change the directory your new project will be created in.


    #. Finally run the `load_package` command using the project's manage.py file.

        .. code-block:: bash

            python manage.py packages -o load_package -s https://github.com/package/archive/branch.zip -db true


	This seems like a logical start...
Creating a New Package
----------------------

If you want to create additional projects with the same data or share your data with others that need to create similar projects, you probably want to create a package.

The `create_package` command will help you get started by generating the folder structure of a new package and loading the resource models of your current project into your new package.

#. To create new package simply run the create_package command. The following example would create a package called `mypackge`.

    .. code-block:: bash

        python manage.py packages -o create_package -d /Full/path/to/mypackage

    -d    full path to the package directory you would like to create
    -o    operation name

#. Below is a list of directories created by the `create_package` command and a brief description of what belongs in each. 

	Place? Like copy, or move?! 
Be sure not to place files that you do not want loaded into these directories. If, for example, you have draft business_data that is not ready for loading, just add a new directory and stage your files there. Directories other than what is listed below will be ignored by the loader.

    business_data
        Resource instance .csv and corresponding .mapping files, each sharing the same base name.
    business_data/files
        Files to be added to the uploaded files directory
    business_data/relations
        Resource relationship files (.relations)
    business_data/resource_views
        sql views of flattened resource models
    extensions/function
        Each function in this directory should have its own directory with a template (.htm), viewmodel (.js) and module (.py). Each file must share the same base name.
    extensions/datatypes
        Each datatype in this directory should have its own directory with a template (.htm), viewmodel (.js) and module (.py). Each file must share the same base name.
    extensions/widgets
        Each widget in this directory should have its own folder with a template (.htm), viewmodel (.js) and configuration file (.json). Each file must share the same base name.
    graphs/branches
        arches.json files representing branches
    graphs/resource_models
        arches.json files representing resource models
    map_layers/mapbox_styles/overlays*
        Each overlay should have a directory with a mapbox style as exported from mapbox including a `style.json` file, `license.txt` file and an `icons` directory
    map_layers/mapbox_styles/basemaps*
        Each basemap should have a directory with a mapbox style as exported from mapbox including a `style.json` file, `license.txt` file and an `icons` directory
    map_layers/tile_server/overlays*
        Each overlay should have a directory with a `.vrt` file and `.xml` to style and configure the layer. Each file must share the same base name.
    map_layers/tile_server/basemaps*
        Each overlay should have a directory with a `.vrt` file and `.xml` to style and configure the layer. Each file must share the same base name.
    preliminary_sql
        sql files containing database operations necessary for your project.
    reference_data/concepts
        SKOS concepts .xml files
    reference_data/collections
        SKOS collection .xml files
    system_settings
        The system settings file for your project

    \* map layer configuration
        By default mapbox-style layers will be loaded with the name property found in the layer's style.json file. The default name for tile server layers will be the basename of the layer's xml file. For both mapbox-style and tile server layers the default icon-class will be `fa fa-globe`. To customize the name and icon-class, simply add a meta.json file to the layer's directory with the following object:

        .. code-block:: javascript

            {
                "name": "example name",
                "icon": "fa example-class"
            }

#. It is not necessary to populate every directory with data. Only add those files that you would like to share.

    Once you've added the necessary files to your package, simply compress it as a zip file or push it to a github repository and it's ready to be loaded.

Configuring a Package
---------------------

Two different files are used to define custom settings for your package.

	Is there a model for this?
- ``package_settings.py``
    The django settings relevant to your project not managed in system settings. For example, you may want to include your time wheel configuration and your analysis SRID settings in this file so that users do not have add these settings manually to their own settings file after loading your package. **This file is copied into your project when the package is loaded.**

	Phrases like " may look like" shouldn't be in a thing called "Documentation"!
- ``package_config.json``
    This file allows you to configure other parts of the data loading process. For example, the order in which the business data files are loaded. Contents of this file may look like

	I don't know where to start with this :(
    .. code-block:: json

          {
              "permitted_resource_relationships": [],
              "business_data_load_order": [
                  "a_LHD_Investigative_Activities_HM.csv",
                  "LHD_Actors.csv",
                  "LHD_Archive_Sources.csv",
                  "LHD_Bibliographic_Sources.csv",
                  "LHD_Heritage_Asset_Areas_PC.csv",
                  "LHD_Heritage_Asset_Artefacts_HM.csv",
                  "LHD_Organizations.csv",
                  "Lincoln_Heritage_Asset_Monument.csv"
              ]
          }


Updating an Existing Package
----------------------------

If you make changes to the resource models in your project you may want to update your package with those changes. You can do that with the `update_package` command:

    .. code-block:: bash

        python manage.py packages -o update_package -d /Full/path/to/mypackage

    -d    full path to the package directory you would like to update
    -o    operation name
    -y    accept defaults (will overwrite existing resource models with those from your project)

Bear in mind that this command will not update a package directly on Github. It will however update a package in a local directory that you have cloned from an existing package on Github or created yourself with the `create_package` command.

.. _dojox/app:

=========
dojox/app
=========

:since: V1.7

.. contents ::
   :depth: 2

Introduction
============

``dojox/app`` is an application framework designed to allow simple configuration of potentially nested views and to
facilitate the transitioning between these views based on a configuration file. Its main current targets are mobile
(phone & tablet) devices but it is not restricted to this and can be used for desktop applications as well. Thanks
to ``dojox/app`` the applications are easily configurable and buildable for easy and fast deployment. There are two
core modules that will need to work together to accomplish these goals:

* ``dojox/app`` - A library that provides high-level application controllers, defined by metadata which describes the overall structure and navigation of the application and it's views.
* ``dojox/mvc`` – An optional library that provides the ability to have view concerns separated from model or data concerns by using simple binding between them so that they can be kept in sync.

If ``dojox/app`` is mandatory, ``dojox/mvc`` can be used or not based on your data binding requirements.

Creating an Application
=======================

As with any Dojo-based web application, it's important to create your HTML page with a script tag referencing the Dojo
loader dojo. Once done you require the ``dojox/app`` main module and provide it with the application JSON configuration file as follows:

.. js ::

  require(["dojox/app/main", "dojox/json/ref", "dojo/text!./config.json"],
      function(Application, json, config){
    // startup the application
    Application(json.fromJson(config));
  });

Upon loading the configuration, the views, stores and models described in the configuration are created and the application
lifecycle is started.

Here is an excerpt of a typical configuration file:

.. js ::

	{
		"id": "todoApp",
		"name": "ToDo App",
		"description": "These is a ToDo sample application based on the Dojo Toolkit and provided under Dojo license.",
		"loaderConfig": {
			"paths": {
				"todoApp": "../demos/todoApp"
			}
		},
		"dependencies": [
			"dojox/mobile/TabBar",
			// ...
			"dojox/mvc/at"
		],
		"modules": [
			"todoApp/todoApp"
		],
		"controllers": [
			"dojox/app/controllers/History"
		],
		"stores": {
			"listsDataStore":{
			   "type": "dojo/data/ItemFileWriteStore",
			   "params": {
					"url": "./resources/data/lists.json"
			   }
			},
			// ...
		},
		"models": {
			"listsmodel": {
				"modelLoader": "dojox/app/utils/mvcModel",
				"type": "dojox/mvc/EditStoreRefListController",
				"params":{
					"datastore": {"$ref":"#stores.listsDataStore"}
				}
			},

			"allitemlistmodel": {
				"modelLoader": "dojox/app/utils/mvcModel",
				"type": "dojox/mvc/EditStoreRefListController",
				"params":{
					"datastore": {"$ref":"#stores.allitemlistStore"}
				}
			}
		},
		"defaultView": "items,ViewListTodoItemsByPriority",
		"defaultTransition": "fade",
		"views": {
			"configuration": {
				"defaultView": "SelectTodoList",
				"defaultTransition": "fade",
				"definition": "none",

				"views": {
					"SelectTodoList": {
						"template": "todoApp/templates/configuration/SelectTodoList.html"
					},

					"ModifyTodoLists": {
						"template": "todoApp/templates/configuration/ModifyTodoLists.html"
					},

					"EditTodoList": {
						"template": "todoApp/templates/configuration/EditTodoList.html"
					}
				}
			},
			// ...
		}
	}

You can find the entire configuration file for this typical application `here <https://github.com/cjolif/dojo-todo-app/blob/master/config-phone.json>`_

Once started the corresponding application looks like the following:

.. image :: ./app/AppExample.png

See the todoApp example in Dojo demos installation directory for the full application.

Building an Application
=======================

Once you have created your configuration file and the application you might want to build the application for production.
For that ``dojox/app`` comes with extensions to the Dojo build system in order to help you build your application from the
configuration file.

In order to achieve that you will need to create a simple Dojo build system profile that will contain the key
information of your build and import the ``dojox/app`` extensions into the build process.

.. js::

	// import the dojox/app extension to the build system
	require(["dojox/app/build/buildControlApp"], function(bc){
	});

	var profile = {
		basePath: "..",
		releaseDir: "./layoutApp/release",
		action: "release",
		cssOptimize: "comments",
	/*	multipleAppConfigLayers: true,*/
		packages:[{
			name: "dojo",
			location: "../../../dojo"
		},{
			name: "dijit",
			location: "../../../dijit"
		},{
			name: "dojox",
			location: "../../../dojox"
		},{
			name: "myApp",
			location: "../../../myApp",
		}],
		layers: {
			"myApp/myApp": {
				include: [ "myApp/index.html" ]
			}
		}
	};


You will then need to reference that profile as well as your configuration file when running the Dojo build tool. For example:

.. ::

    node ../../dojo/dojo.js load=build --profile ../../dojox/app/tests/layoutApp/build.profile.js --appConfigFile layoutApp/config.json


By default the extension uses the first layer in the profile (here "myApp/myApp") to bundle all the modules for the
application. You can specify an alternate layer you want to target by passing -appConfigLayer "layer/name" on the command line.

Alternatively, you can make sure a layer per-view is built instead of a single layer for all the app by having the
multipleAppConfigLayers: true set to true in your profile. This is useful if you have a lot of views that won't get
navigated to in a typical usage of your application. In that case you might not want to load everything upfront. In this
case the definition file of each view will be used as the layer for the view.

**Limitation**

This extension does not support the "./" shortcut notation to reference the modules in the config file and default
definition file. You have to explicitly list your definition file and use absolute module paths. You can very easily
do that by creating an "myApp" module that you should instead of "." to reference your modules.


The Configuration File
======================

The configuration comes in the form of a JSON-like object of the following keys and property values:

id
--

String. The ``dojox/app`` application's id. A global variable with id’s name is created by ``dojox/app/main``.

.. js ::

  id: "sampleApp"


name
----

String. The ``dojox/app`` application's name.

.. js ::

  name: "Sample App"

description
-----------

String. The description of the ``dojox/app`` application

.. js ::

  description: "Sample application that does what is needed"

loaderConfig
------------

Object. Dependencies, modules and controllers are loaded using the Dojo AMD loader. This parameter allows to configure the loader itself and specify for example where custom modules can be found. See http://livedocs.dojotoolkit.org/loader/amd#module-identifiers

.. js ::

  "loaderConfig" : {
    "paths": {
      "mypackage" : "can/be/found/here"
     }
  }

dependencies
------------

Array of AMD modules identifiers. When defined at the top level dependencies of the ``dojox/app`` application. When defined at view level, dependencies for the view.

.. js ::

  "dependencies": [
    "dojox/mobile/TabBar",
    "dojox/mobile/RoundRect",
    "dojox/mobile/TabBarButton",
    "dojox/mobile/Button",
    "dojox/mobile/RoundRect",
    "dojox/mobile/Heading"
  ]

These are typically modules that are not required by the JavaScript module of the application or the view but that are still
needed by the application or the view. Examples are the store or MVC modules or the modules needed by the markup of the view.
If the application is using the full Dojo parser it does not necessarily need to list the markup dependencies as the parser
will auto-require them.

modules
-------

Array of AMD modules identifiers. Modules for the application. Used as the mixins when declaring the Application class in ``dojox/app/main``. They modify the top level behavior of the application, how it processes the config, or any other life cycle. The “``dojox/app/module/lifecycle``” is automatically mixed into the application you don’t have the explicitly include it.

.. js ::

  "modules": [
    "mypackage/custom/module"
  ]

They are automatically added to the list of dependencies and as such do not need to be listed in the dependencies property.

controllers
-----------

Array of AMD modules identifiers. Controllers for the application. All the controllers defined here will be loaded during application startup to respond to application events and controller the application logic. The “``dojox/app/controllers/Load``” and “``dojox/app/controllers/Transition``” are automatically mixed into the application you don’t have the explicitly include them.

.. js ::

  "controllers": [
    "dojox/app/controllers/History",
    "mypackage/custom/appController"
  ]

They are automatically added to the list of dependencies and as such do not need to be listed in the dependencies property.

stores
------

Object. Dojo stores which are used by ``dojox/app`` to setup data models. A store item is an object with a a type and a params property. The type property is the AMD module identifier for the store class to be instantiated. The content of the params property is passed to the store class constructor to build an instance.

.. js ::

  "stores": {
    "store1":{
      "type": "dojo/store/Memory",
      "params": { // parameters used to initialize the data store
        "data": "modelApp.names"
      }
    },
    "store1":{
      "type": "dojo/store/JsonRest",
      "params": {
        "data": "modelApp.repeatData"
      }
    }
  }

All stores modules that are used must also be explicitly listed in the dependencies property.

template
--------

String. HTML file. When defined at application level defines the application root template. When defined at view level defined the view template.

.. js ::

  "template": "application.html"

models
------

Object. Models and their instantiation parameters. A model item is an object with three properties: the model type, the modelLoader and the params. The modelLoader property defines whether an MVC or a simple model must be loaded. The type property defines which class must be used for that model using an AMD module identifier and finally the params property content is passed to the model class constructor to build an instance.

.. js ::

  "models": {
    "model1": {
        "modelLoader": "dojox/app/utils/mvcModel",
        "type": "dojox/mvc/EditStoreRefListController",
        "params":{
          "store": {"$ref":"#stores.namesStore"}
        }
    },
    "model2": {
      "modelLoader": "dojox/app/utils/simpleModel",
      "params":{
        "store": {"$ref":"#stores.namesStore"}
      }
    }
  }

All model modules that are used must also be explicitly listed in the dependencies property.

defaultView
-----------

String. The name of the view to load when the application is initialized.

.. js ::

  "defaultView": "home"


defaultTransition
-----------------

String. The default animation type for the view transition.

.. js ::

  "defaultTransition": "slide"


views
-----
Object. The children views of an application or of a view. Dependencies may be defined on views for optimization and organization purposes. Models might also be defined on views if they are view-specific. Finally a view item as three additional properties: transition for specific view transitions, template for defining the view rendering and finally definition to provide an AMD module to be mixed into the view to control it.
AMD modules identifiers starting with “.” will be resolved relative to the application root. All other modules identifiers will be resolved according to the Dojo AMD loader rules and in particular with respect to its configuration provided as part of the loaderConfig attribute.
By default if no definition module is specified for a view it is looked up automatically in "./views/<viewId>.js". If you don’t want a definition module at all you should specify the "none" value.

.. js ::

  "views": {
    // simple view without any children views
    // views can has its own dependencies which will be loaded
    // before the view is first initialized.
    "home": {
      "dependencies":[
        "dojox/mobile/RoundRectList",
        "dojox/mobile/ListItem",
        "dojox/mobile/EdgeToEdgeCategory"
      ],
      "template": "./views/simple/home.html"
      "definition": "./views/simple/home.js "
    },

    // simple composite view which loads all views and shows the default
    "main":{
      // all views in the main view will be bound to the user model
      "models": [],
      "template": "simple.html",
      "defaultView": "main",
      "defaultTransition": "slide",
      // the child views available to this view
      "views": {
        "main":{
          "template": "./views/simple/main.html"
          "definition": "none"
        },
        "second":{
          "template": "./views/simple/second.html"
        },
        "third":{
          "template": "./views/simple/third.html"
        }
      },
      "dependencies":[
        "dojox/mobile/RoundRectList",
        "dojox/mobile/ListItem",
        "dojox/mobile/EdgeToEdgeCategory",
        "dojox/mobile/EdgeToEdgeList"
      ]
    },
    "repeat": {
      // model declared at view level will be accessible to this view
      // or its children.
      "models": {
        "repeatmodels": {
          "params":{
            "store": {"$ref":"#stores.repeatStore"}
          }
        }
      },
       "template": "./views/repeat.html",
      "dependencies":["dojox/mobile/TextBox"]
    }
  }

This configuration serves two purposes configuring the application within the client without having to do it by code
and help building the application for production.

The main dojox/app modules
==========================

``dojox/app`` is built around the following focused core modules that can be used in the configuration file:

:ref:`dojox/app/main <dojox/app/main>` is used to create a ``dojox/app`` Application object from the JSON configuration. The main responsibilities of ``dojox/app/main`` include loading the various controllers & data stores as well as managing the application lifecycle.

:ref:`dojox/app/View <dojox/app/view>` provides a view object in charge of the view rendering and lifecycle. It contains a template string which will be rendered.  A view can itself have nested View objects.

:ref:`dojox/app/Controller <dojox/app/Controller>` a base class for the various application controllers:

* ``dojox/app/controllers/Layout`` a controller that performs nested view layout

* ``dojox/app/controllers/Load`` a controller that loads the view templates and view definition modules

* ``dojox/app/controllers/History`` a controller that maintains application history using HTM5 history API. This will not work on platforms that don’t support it like IE, Android 3 & 4, iOS 4.

* ``dojox/app/controllers/HistoryHash`` an alternate  controller that maintains application history using URL hash. It works on all browsers but has limitations with regard to browser refresh and going back to an URL out of application’s history stack.

:ref:`dojox/app/model <dojox/app/model>`, depending on the application models configuration, is creating either simple or MVC models for the views to bind their widgets to.

:ref:`dojox/app/module <dojox/app/module>` a package containing various modules than can be used in the configuration file to be mixed into the Application object.


The following diagram represents the high level architecture of ``dojox/app`` and in particular how the modules listed
above interacts each others:

.. image :: ./app/AppDiagram.png


Comparison with ``dojox/mobile/app``
====================================

First please note that going forward ``dojox/mobile/app`` will be deprecated in favor of ``dojox/app``.

The main differences between ``dojox/app`` and ``dojox/mobile/app`` are the following:

* ``dojox/app`` enables the model binding
* ``dojox/app`` uses view structure to enable the nested views using a composite design pattern. ``dojox/mobile/app`` will require additional code for that.
* ``dojox/app`` contains the layout mechanism to ensure the content at different application/view level work well together
* ``dojox/mobile/app`` manages the navigation history in StageController by using a history stack. Dojox/app provides both HTML5 pushState & history stack for  managing the navigation history.

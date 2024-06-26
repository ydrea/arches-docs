###################
Managing Map Layers
###################

Different Types of Layers
-------------------------

Arches allows a great deal of customization for the layers on the search map. The contents of the following section will be useful when using the **Map Layer Manager** to customize your layers.

Resource Layers
```````````````

Resource Layers display the resource layers in your database. One Resource Layer is created for each node with a geospatial datatype (for example, ``geojson-feature-collection``). You are able to customize the appearance and visibility of each Resource Layer in the following ways.

**Styling**

Define the way features will look on the map. The example map has demonstration features that give you a preview of the changes you make. You can choose to use Advanced Editing to create a more nuanced style. Note that changes made in **Advanced Editing** will not be reflected if you switch back to basic editing. For styling reference, checkout the MapBox Style Specification.

**Clustering**

Arches uses "clustering" to better display resources at low zoom levels (zoomed out). You are able to control the clustering settings for each resource layer individually.

* Cluster Distance - distance (in pixels) within which resources will be clustered
* Cluster Max Zoom - zoom level after which clustering will stop being used
* Cluster Min Points - minimum number of points needed to create a cluster

**Caching**

Caching tiles will improve the speed of map rendering by storing tiles locally as they are creating. This eliminates the need for new tile generation when viewing a portion of the map that has already been viewed. However, caching is not a simple matter, and it is disabled by default. Caching is only advisable if you know what you are doing.

Basemaps and Overlays
`````````````````````

A Basemap will always be present in your map. Arches comes with a few default basemaps, but advanced users can configure and add more.

Overlays are the best way to incorporate map layers from external sources. On the search map, a user is able to activate as many overlays as desired simultaneously. Users can also change the transparency of overlays. New overlays can be added in the same manner as new basemaps.

.. admonition:: Adding New Basemaps or Overlays

    If you are a developer interested in creating new map layers (which could be new visualizations of resources or new basemaps and overlays), please see :ref:`creating_new_map_layers_reference`.

Styling
```````

Note that depending on the type of layer, there are different styling options. For styling reference, checkout the `MapBox Style Specification <https://www.mapbox.com/mapbox-gl-js/style-spec/#layers>`_.

Settings
````````

* Layer name - Enter a name to identify this basemap.
* Default search map - For basemaps, you can designate one to be the default. For overlays, you can choose whether a layer appears on the in the search map by default. Note that in the search map itself you can change the order of overlays.
* Layer icon - Associate an icon with this layer


Permissions
```````````
As of Arches version 7.4.0, you can assign different permissions to specific Arches *users* and *groups*. To manage such permissions, please review :ref:`Map Layer Permissions`.
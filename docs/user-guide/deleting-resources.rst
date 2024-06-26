##################
Deleting Resources
##################

Arches provides user interface features to delete node instances used to describe resource instances. Arches also provides user interface features to delete any single given resource instance. Finally, Arches provides a feature to delete all resource instances for a given resource model.


Select a Resource Instance to Edit or Delete
--------------------------------------------
To find a resource instance that you want to delete, you can start by using the search interface (learn more here: :ref:`Searching`). Assuming you are logged in as a user with permissions to edit the resource instance of interest, you should find a link to **Edit** the resource instance in the search results. The animation below illustrates the use of search and the **Edit** link:

.. image:: ../images/select-resource-instance-for-edit.gif


Delete a Node Instance for a Resource
-------------------------------------
One you have opened a resource instance to edit, you have the option to delete (and then update if you choose) node instances associated with the resource instance. Node instance data are data that describe a given resource instances according to the structures defined by "branches". The animation below illustrates different options one can use to delete an example annotation node instance:

.. image:: ../images/delete-node-instance-options.gif


Delete a Resource Instance Entirely
-----------------------------------
Sometimes you may wish to entirely delete a given resource instance. To do so, follow the directions above to find the resource instance you wish to delete and follow the **Edit** link. If you have edit permissions, on the resource instance edit page, you should see a **Manage** button toward the upper left corner of the page. Click on this, and select the **Delete Resource** option and then confirm your choice if you are sure you want to permanently delete the resource instance. See the animation below for an example resource instance deletion:

.. image:: ../images/delete-resource-instance.gif


Delete ALL Resource Instances for a Resource Model
--------------------------------------------------
Arches also provides a user interface feature for bulk deletion of all resource instances created for a given resource model. To do so, navigate to the **Arches Designer** and select the **Resource Models** option. Then hover over the resource model of interest, and click on the **Manage** options on the right. You can then select the **Delete Instances** option, and after confirming your choice, you will delete *all* resource instances for that resource model. The animation below illustrates deletion of all resource instances for a resource model:

.. image:: ../images/delete-all-resource-instances-for-model.gif

.. warning::
    Obviously, deleting *all* resource instances for a given resource model can be a drastic measure. It may be a good idea to export and backup your data prior to such major changes (see :ref:`Export Commands`).
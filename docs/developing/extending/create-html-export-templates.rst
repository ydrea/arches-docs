##############################
Creating HTML Export Templates
##############################

The HTML export templates are used to allow search results in Arches to be exported as a set of html files containing formatted report styled documents.

They are designed to be be read offline by embedding a small CSS framework called Milligram, along with some additional elements.

The templates must be written by a developer and added to the Arches project template folder.

There can only be one HTML export template for each model in the application. 

Template Location
=================

The templates should be saved to a folder in ``/path/to/project_workspace/project_name/project_name/templates/html_export``.

The templates must have the name format of ``<graph id>.htm``, with the <graph id> value being the model's UUID.

**example**: ``076f9381-7b00-11e9-8d6b-80000b44d1d9.htm``

Templating Language
===================

The templates are using the native Django templating engine, information of which can be found here: https://docs.djangoproject.com/en/stable/ref/templates/language/


Resources Context Data
======================

When the export process loads the template ready for it to be rendered, it will be passed a ``resources`` context object that contains the data for the resources to be written. This list is then iterated to build each record.

The structure of each resources object is as below:

.. code-block:: JSON

   {
      "displaydescription": " Excavation by Department Of Greater London Archaeology, April to May 1990, found a large 'soft spot' which was either a quarry ditch or was dug for dumping waste. Also uncovered features of 20th century date relating to a building called Green Acres.",
      "displayname": "Open Area Excavation at Lichfield Gardens",
      "graph_id": "b9e0701e-5463-11e9-b5f5-000d3ab1e588",
      "legacyid": "06eb7a47-baf7-4c79-aeab-2ffabe2502ea",
      "map_popup": "<Activity Descriptions>",
      "resource": {
         "...": "..."
      },
      "resourceinstanceid": "06eb7a47-baf7-4c79-aeab-2ffabe2502ea"
   }


Some id and display information is directly accessible from the object, such as the ``displayname`` and ``displaydescription``. The rest of the resource data is contained in the ``resource`` dictionary as a "disambiguated" version of the resource. 

The dict uses the branch/card/node names as the keys, with the ``@display_value`` key containing the presentation value for that node. There are other values included in the dicts that can be used to add richer functionality if needed.

.. note::

   This structure closely matches the JSON produced when looking at a resource when specifying the ``format=json&v=beta``

   .. code-block:: bash

      curl http://localhost:8000/resources/<resourceinstanceid>?format=json&v=beta

   Ensure that you use the ``v=beta`` parameter as the functionality is using this version of the data formatting.
   

.. code-block:: javascript

   "resource": {
       "Descriptions": [{
               "Description": {
                   "@display_value": "Amendment date:none"
               },
               "Description Language": {
                   "Description Language Metatype": null
               },
               "Description Type": {
                   "@display_value": "Notes",
                   "Description Metatype": null,
                   "concept_id": "f1cbae8f-0090-47dc-8252-ee533a2deb29",
                   "language_id": "en",
                   "value": "Notes",
                   "valueid": "daa4cddc-8636-4842-b836-eb2e10aabe18",
                   "valuetype_id": "prefLabel"
               }
           }
       ],
       "Designation and Protection Assignment": [],
       "Heritage Area Names": [],
       "Location Data": {},
       "System Reference Numbers": {}
   }

Custom Template Filters
=======================

The resources context data sent to the templates has an incompatibility with the standard Django dot notation for accessing values. Usually you would access a dictionary value using ``dict.key``, but the resource dictionary uses keys with spaces that can't be parsed ``resource_data.Activity Names``.

The other difficulty is that the resource dictionary may not contain the key you are looking for if no tiles for that data exists. Therefore, you need to check for the existence of the key before you access it.

To solve this, two new template filters were added to ``arches/arches/templatetags/template_tags.py``:

**has_key**

You can use `has_key` as part of an `if` tag to check if there is a key in the object. If you try to access the object without checking then it may error should the key not be present.

.. code-block:: HTML

   {% if asset_names|has_key:"Asset Name Use Type" %}
      {# you can access without error asset_names["Asset Name Use Type"] #}
   {% endif %}

**val_from_key**

This function allows you to retrieve a value from a key that is not Django templating compliant. These can be chained to access nested dictionaries (careful that you a sure the nested dictionary exists).

.. code-block:: HTML

   <h2>{{ asset_names|val_from_key:"Asset Name"|val_from_key:"@display_value" }}</h2>

**json_to_obj**

This function can be used in the rare instance where the value is JSON and you need to convert that to a dict or list in order to access the values.

.. code-block:: HTML

   <a href="{{ external_source|val_from_key:'URL'|val_from_key:'@display_value'|json_to_obj|val_from_key:'url' }}" target="_blank">Document Link</a>

**example** combining ``has_key`` and ``val_from_key``.

.. code-block:: HTML

   {% if asset_names|has_key:"Asset Name Use Type" %}
      <h2>{{ asset_names|val_from_key:"Asset Name"|val_from_key:"@display_value" }}</h2>
   {% endif %}

**example** of using chained filters to access nested values

.. code-block:: HTML

   <strong>Primary Reference Number: </strong>{{ resource_data|val_from_key:"System Reference Numbers"|val_from_key:"PrimaryReferenceNumber"|val_from_key:"Primary Reference Number"|val_from_key:"@display_value" }}


**example** transforming a url dataype value into a dict for use within a data table call

.. code-block:: HTML

   <div class="rcell" data-title="URL">
      {% with exref|val_from_key:"URL"|val_from_key:"@display_value"|json_to_obj as URL_Dict %}
         {% if URL_Dict|has_key:"url" and URL_Dict|has_key:"url_label" %}
            <a href="{{ URL_Dict|val_from_key:'url' }}">{{ URL_Dict|val_from_key:'url_label' }}</a>
         {% else %}
            <br />
         {% endif %}
      {% endwith %}
   </div>

Basic Template
==============

The basic template below will provide the CSS framework, add the custom template tags (used to add custom functions to the template engine), and the initial resource loop within which to start your document.

.. code-block:: HTML

   {% load template_tags %}
   <!DOCTYPE html>
   <html lang="en">
      <head>
         <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
         <meta charset="UTF-8">
         <meta name="viewport" content="width=device-width, initial-scale=1.0">
         <title>--ADD DOCUMENT TITLE--</title>
         <style>
         /* Milligram css */
         *,*:after,*:before{box-sizing:inherit}html{box-sizing:border-box;font-size:62.5%}body{color:#606c76;font-family:'Roboto', 'Helvetica Neue', 'Helvetica', 'Arial', sans-serif;font-size:1.6em;font-weight:300;letter-spacing:.01em;line-height:1.6}blockquote{border-left:0.3rem solid #d1d1d1;margin-left:0;margin-right:0;padding:1rem 1.5rem}blockquote *:last-child{margin-bottom:0}.button,button,input[type='button'],input[type='reset'],input[type='submit']{background-color:#9b4dca;border:0.1rem solid #9b4dca;border-radius:.4rem;color:#fff;cursor:pointer;display:inline-block;font-size:1.1rem;font-weight:700;height:3.8rem;letter-spacing:.1rem;line-height:3.8rem;padding:0 3.0rem;text-align:center;text-decoration:none;text-transform:uppercase;white-space:nowrap}.button:focus,.button:hover,button:focus,button:hover,input[type='button']:focus,input[type='button']:hover,input[type='reset']:focus,input[type='reset']:hover,input[type='submit']:focus,input[type='submit']:hover{background-color:#606c76;border-color:#606c76;color:#fff;outline:0}.button[disabled],button[disabled],input[type='button'][disabled],input[type='reset'][disabled],input[type='submit'][disabled]{cursor:default;opacity:.5}.button[disabled]:focus,.button[disabled]:hover,button[disabled]:focus,button[disabled]:hover,input[type='button'][disabled]:focus,input[type='button'][disabled]:hover,input[type='reset'][disabled]:focus,input[type='reset'][disabled]:hover,input[type='submit'][disabled]:focus,input[type='submit'][disabled]:hover{background-color:#9b4dca;border-color:#9b4dca}.button.button-outline,button.button-outline,input[type='button'].button-outline,input[type='reset'].button-outline,input[type='submit'].button-outline{background-color:transparent;color:#9b4dca}.button.button-outline:focus,.button.button-outline:hover,button.button-outline:focus,button.button-outline:hover,input[type='button'].button-outline:focus,input[type='button'].button-outline:hover,input[type='reset'].button-outline:focus,input[type='reset'].button-outline:hover,input[type='submit'].button-outline:focus,input[type='submit'].button-outline:hover{background-color:transparent;border-color:#606c76;color:#606c76}.button.button-outline[disabled]:focus,.button.button-outline[disabled]:hover,button.button-outline[disabled]:focus,button.button-outline[disabled]:hover,input[type='button'].button-outline[disabled]:focus,input[type='button'].button-outline[disabled]:hover,input[type='reset'].button-outline[disabled]:focus,input[type='reset'].button-outline[disabled]:hover,input[type='submit'].button-outline[disabled]:focus,input[type='submit'].button-outline[disabled]:hover{border-color:inherit;color:#9b4dca}.button.button-clear,button.button-clear,input[type='button'].button-clear,input[type='reset'].button-clear,input[type='submit'].button-clear{background-color:transparent;border-color:transparent;color:#9b4dca}.button.button-clear:focus,.button.button-clear:hover,button.button-clear:focus,button.button-clear:hover,input[type='button'].button-clear:focus,input[type='button'].button-clear:hover,input[type='reset'].button-clear:focus,input[type='reset'].button-clear:hover,input[type='submit'].button-clear:focus,input[type='submit'].button-clear:hover{background-color:transparent;border-color:transparent;color:#606c76}.button.button-clear[disabled]:focus,.button.button-clear[disabled]:hover,button.button-clear[disabled]:focus,button.button-clear[disabled]:hover,input[type='button'].button-clear[disabled]:focus,input[type='button'].button-clear[disabled]:hover,input[type='reset'].button-clear[disabled]:focus,input[type='reset'].button-clear[disabled]:hover,input[type='submit'].button-clear[disabled]:focus,input[type='submit'].button-clear[disabled]:hover{color:#9b4dca}code{background:#f4f5f6;border-radius:.4rem;font-size:86%;margin:0 .2rem;padding:.2rem .5rem;white-space:nowrap}pre{background:#f4f5f6;border-left:0.3rem solid #9b4dca;overflow-y:hidden}pre>code{border-radius:0;display:block;padding:1rem 1.5rem;white-space:pre}hr{border:0;border-top:0.1rem solid #f4f5f6;margin:3.0rem 0}input[type='color'],input[type='date'],input[type='datetime'],input[type='datetime-local'],input[type='email'],input[type='month'],input[type='number'],input[type='password'],input[type='search'],input[type='tel'],input[type='text'],input[type='url'],input[type='week'],input:not([type]),textarea,select{-webkit-appearance:none;background-color:transparent;border:0.1rem solid #d1d1d1;border-radius:.4rem;box-shadow:none;box-sizing:inherit;height:3.8rem;padding:.6rem 1.0rem .7rem;width:100%}input[type='color']:focus,input[type='date']:focus,input[type='datetime']:focus,input[type='datetime-local']:focus,input[type='email']:focus,input[type='month']:focus,input[type='number']:focus,input[type='password']:focus,input[type='search']:focus,input[type='tel']:focus,input[type='text']:focus,input[type='url']:focus,input[type='week']:focus,input:not([type]):focus,textarea:focus,select:focus{border-color:#9b4dca;outline:0}select{background:url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 30 8" width="30"><path fill="%23d1d1d1" d="M0,0l6,8l6-8"></path></svg>') center right no-repeat;padding-right:3.0rem}select:focus{background-image:url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 30 8" width="30"><path fill="%239b4dca" d="M0,0l6,8l6-8"></path></svg>')}select[multiple]{background:none;height:auto}textarea{min-height:6.5rem}label,legend{display:block;font-size:1.6rem;font-weight:700;margin-bottom:.5rem}fieldset{border-width:0;padding:0}input[type='checkbox'],input[type='radio']{display:inline}.label-inline{display:inline-block;font-weight:normal;margin-left:.5rem}.container{margin:0 auto;max-width:112.0rem;padding:0 2.0rem;position:relative;width:100%}.row{display:flex;flex-direction:column;padding:0;width:100%}.row.row-no-padding{padding:0}.row.row-no-padding>.column{padding:0}.row.row-wrap{flex-wrap:wrap}.row.row-top{align-items:flex-start}.row.row-bottom{align-items:flex-end}.row.row-center{align-items:center}.row.row-stretch{align-items:stretch}.row.row-baseline{align-items:baseline}.row .column{display:block;flex:1 1 auto;margin-left:0;max-width:100%;width:100%}.row .column.column-offset-10{margin-left:10%}.row .column.column-offset-20{margin-left:20%}.row .column.column-offset-25{margin-left:25%}.row .column.column-offset-33,.row .column.column-offset-34{margin-left:33.3333%}.row .column.column-offset-40{margin-left:40%}.row .column.column-offset-50{margin-left:50%}.row .column.column-offset-60{margin-left:60%}.row .column.column-offset-66,.row .column.column-offset-67{margin-left:66.6666%}.row .column.column-offset-75{margin-left:75%}.row .column.column-offset-80{margin-left:80%}.row .column.column-offset-90{margin-left:90%}.row .column.column-10{flex:0 0 10%;max-width:10%}.row .column.column-20{flex:0 0 20%;max-width:20%}.row .column.column-25{flex:0 0 25%;max-width:25%}.row .column.column-33,.row .column.column-34{flex:0 0 33.3333%;max-width:33.3333%}.row .column.column-40{flex:0 0 40%;max-width:40%}.row .column.column-50{flex:0 0 50%;max-width:50%}.row .column.column-60{flex:0 0 60%;max-width:60%}.row .column.column-66,.row .column.column-67{flex:0 0 66.6666%;max-width:66.6666%}.row .column.column-75{flex:0 0 75%;max-width:75%}.row .column.column-80{flex:0 0 80%;max-width:80%}.row .column.column-90{flex:0 0 90%;max-width:90%}.row .column .column-top{align-self:flex-start}.row .column .column-bottom{align-self:flex-end}.row .column .column-center{align-self:center}@media (min-width: 40rem){.row{flex-direction:row;margin-left:-1.0rem;width:calc(100% + 2.0rem)}.row .column{margin-bottom:inherit;padding:0 1.0rem}}a{color:#9b4dca;text-decoration:none}a:focus,a:hover{color:#606c76}dl,ol,ul{list-style:none;margin-top:0;padding-left:0}dl dl,dl ol,dl ul,ol dl,ol ol,ol ul,ul dl,ul ol,ul ul{font-size:90%;margin:1.5rem 0 1.5rem 3.0rem}ol{list-style:decimal inside}ul{list-style:circle inside}.button,button,dd,dt,li{margin-bottom:1.0rem}fieldset,input,select,textarea{margin-bottom:1.5rem}blockquote,dl,figure,form,ol,p,pre,table,ul{margin-bottom:2.5rem}table{border-spacing:0;display:block;overflow-x:auto;text-align:left;width:100%}td,th{border-bottom:0.1rem solid #e1e1e1;padding:1.2rem 1.5rem}td:first-child,th:first-child{padding-left:0}td:last-child,th:last-child{padding-right:0}@media (min-width: 40rem){table{display:table;overflow-x:initial}}b,strong{font-weight:bold}p{margin-top:0}h1,h2,h3,h4,h5,h6{font-weight:300;letter-spacing:-.1rem;margin-bottom:2.0rem;margin-top:0}h1{font-size:4.6rem;line-height:1.2}h2{font-size:3.6rem;line-height:1.25}h3{font-size:2.8rem;line-height:1.3}h4{font-size:2.2rem;letter-spacing:-.08rem;line-height:1.35}h5{font-size:1.8rem;letter-spacing:-.05rem;line-height:1.5}h6{font-size:1.6rem;letter-spacing:0;line-height:1.4}img{max-width:100%}.clearfix:after{clear:both;content:' ';display:table}.float-left{float:left}.float-right{float:right}
         /* General */
         html{font-size:55%}body{color:#000}.section-title blockquote{border:.3rem solid #d1d1d1;background-color:#90C0D8}h3{margin-bottom:.2rem}.container{margin:0;max-width:100%}hr{border-top:.3rem solid #d1d1d1;margin:2rem 0}ul{list-style:none}.location-details li{margin-bottom:0}@media print{.section-title:not(:first-child){page-break-before:always}.keeptogether{break-inside:avoid}}
         /* Responsive tables */
         .rtable{margin:0 0 40px 0;width:100%;box-shadow:0 1px 3px rgba(0,0,0,.2);display:table}@media screen and (max-width:580px){.rtable{display:block}}.rrow{display:table-row}.rrow:nth-of-type(odd){background-color:#fff}.rrow.rheader{font-weight:600;background:#d1d1d1}@media screen and (max-width:1024px){.rrow{padding:14px 0 7px;display:block;border-bottom:1px solid #d1d1d1}.rrow.rheader{padding:0;height:6px}.rrow.rheader .rcell{display:none}.rrow .rcell{margin-bottom:10px;border:none}.rrow .rcell:before{margin-bottom:3px;content:attr(data-title);min-width:98px;font-size:.85em;line-height:10px;font-weight:700;text-transform:uppercase;display:block}}.rcell{padding:6px 12px;display:table-cell;border-bottom:1px solid #d1d1d1}.rcell ul{list-style:none}.rcell li{margin-bottom:0}@media screen and (max-width:1024px){.rcell{padding:2px 16px;display:block}}@media print{html{font-size:40%}.rtable{box-shadow:none;border:1px solid #d1d1d1}.rcell{border:1px solid #d1d1d1}.location-details .row{flex-direction:row}}
         </style>
      </head>
      <body>
         <header>
            <h1>--ADD MAIN HEADER TITLE--</h1>
         </header>
         <main>
            {% for resource in resources %}
            <h2>{{ resource.displayname }}</h2>
            <p>{{ resource.displaydesicription }}</p>
               {% with resource_data=resource.resource %}
               ... build template
               {% endwith %}
            {% endfor %}
         </main>
      </body>
   </html>


Below show examples of how you can fetch specific data out of the resource object to build a section of the document.

.. code-block:: HTML

   {% with resource_data=resource.resource %}
   <section class="section-title">
      <blockquote>
         {% if resource_data|has_key:"Heritage Asset Names" %}
            {% for n in resource_data|val_from_key:"Heritage Asset Names" %}
               {% if n|has_key:"Asset Name Use Type" %}
                  {% if n|val_from_key:"Asset Name Use Type"|val_from_key:"@display_value" == "Primary" %}
                     <h2>{{ n|val_from_key:"Asset Name"|val_from_key:"@display_value" }}</h2>
                  {% endif %}
               {% endif %}
            {% endfor %}
         {% endif%}
         <p>
            <strong>Primary Reference Number: </strong>{{ resource_data|val_from_key:"System Reference Numbers"|val_from_key:"PrimaryReferenceNumber"|val_from_key:"Primary Reference Number"|val_from_key:"@display_value" }}<br>
         </p>
      </blockquote>
   </section>
   {% endwith %}

Advanced Template Example
=========================

Below is an example that includes sections that build tables and group elements together.

.. note::

   Use ``<div class="keeptogether"></div>`` blocks around tables and other iterated sections to force the styling to keep things on the same page where possible when printing.

.. code-block:: HTML

   {% load template_tags %}
   <!DOCTYPE html>
   <html lang="en">
      <head>
         <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
         <meta charset="UTF-8">
         <meta name="viewport" content="width=device-width, initial-scale=1.0">
         <title>Report</title><style>
            /* Milligram css */
            *,*:after,*:before{box-sizing:inherit}html{box-sizing:border-box;font-size:62.5%}body{color:#606c76;font-family:'Roboto', 'Helvetica Neue', 'Helvetica', 'Arial', sans-serif;font-size:1.6em;font-weight:300;letter-spacing:.01em;line-height:1.6}blockquote{border-left:0.3rem solid #d1d1d1;margin-left:0;margin-right:0;padding:1rem 1.5rem}blockquote *:last-child{margin-bottom:0}.button,button,input[type='button'],input[type='reset'],input[type='submit']{background-color:#9b4dca;border:0.1rem solid #9b4dca;border-radius:.4rem;color:#fff;cursor:pointer;display:inline-block;font-size:1.1rem;font-weight:700;height:3.8rem;letter-spacing:.1rem;line-height:3.8rem;padding:0 3.0rem;text-align:center;text-decoration:none;text-transform:uppercase;white-space:nowrap}.button:focus,.button:hover,button:focus,button:hover,input[type='button']:focus,input[type='button']:hover,input[type='reset']:focus,input[type='reset']:hover,input[type='submit']:focus,input[type='submit']:hover{background-color:#606c76;border-color:#606c76;color:#fff;outline:0}.button[disabled],button[disabled],input[type='button'][disabled],input[type='reset'][disabled],input[type='submit'][disabled]{cursor:default;opacity:.5}.button[disabled]:focus,.button[disabled]:hover,button[disabled]:focus,button[disabled]:hover,input[type='button'][disabled]:focus,input[type='button'][disabled]:hover,input[type='reset'][disabled]:focus,input[type='reset'][disabled]:hover,input[type='submit'][disabled]:focus,input[type='submit'][disabled]:hover{background-color:#9b4dca;border-color:#9b4dca}.button.button-outline,button.button-outline,input[type='button'].button-outline,input[type='reset'].button-outline,input[type='submit'].button-outline{background-color:transparent;color:#9b4dca}.button.button-outline:focus,.button.button-outline:hover,button.button-outline:focus,button.button-outline:hover,input[type='button'].button-outline:focus,input[type='button'].button-outline:hover,input[type='reset'].button-outline:focus,input[type='reset'].button-outline:hover,input[type='submit'].button-outline:focus,input[type='submit'].button-outline:hover{background-color:transparent;border-color:#606c76;color:#606c76}.button.button-outline[disabled]:focus,.button.button-outline[disabled]:hover,button.button-outline[disabled]:focus,button.button-outline[disabled]:hover,input[type='button'].button-outline[disabled]:focus,input[type='button'].button-outline[disabled]:hover,input[type='reset'].button-outline[disabled]:focus,input[type='reset'].button-outline[disabled]:hover,input[type='submit'].button-outline[disabled]:focus,input[type='submit'].button-outline[disabled]:hover{border-color:inherit;color:#9b4dca}.button.button-clear,button.button-clear,input[type='button'].button-clear,input[type='reset'].button-clear,input[type='submit'].button-clear{background-color:transparent;border-color:transparent;color:#9b4dca}.button.button-clear:focus,.button.button-clear:hover,button.button-clear:focus,button.button-clear:hover,input[type='button'].button-clear:focus,input[type='button'].button-clear:hover,input[type='reset'].button-clear:focus,input[type='reset'].button-clear:hover,input[type='submit'].button-clear:focus,input[type='submit'].button-clear:hover{background-color:transparent;border-color:transparent;color:#606c76}.button.button-clear[disabled]:focus,.button.button-clear[disabled]:hover,button.button-clear[disabled]:focus,button.button-clear[disabled]:hover,input[type='button'].button-clear[disabled]:focus,input[type='button'].button-clear[disabled]:hover,input[type='reset'].button-clear[disabled]:focus,input[type='reset'].button-clear[disabled]:hover,input[type='submit'].button-clear[disabled]:focus,input[type='submit'].button-clear[disabled]:hover{color:#9b4dca}code{background:#f4f5f6;border-radius:.4rem;font-size:86%;margin:0 .2rem;padding:.2rem .5rem;white-space:nowrap}pre{background:#f4f5f6;border-left:0.3rem solid #9b4dca;overflow-y:hidden}pre>code{border-radius:0;display:block;padding:1rem 1.5rem;white-space:pre}hr{border:0;border-top:0.1rem solid #f4f5f6;margin:3.0rem 0}input[type='color'],input[type='date'],input[type='datetime'],input[type='datetime-local'],input[type='email'],input[type='month'],input[type='number'],input[type='password'],input[type='search'],input[type='tel'],input[type='text'],input[type='url'],input[type='week'],input:not([type]),textarea,select{-webkit-appearance:none;background-color:transparent;border:0.1rem solid #d1d1d1;border-radius:.4rem;box-shadow:none;box-sizing:inherit;height:3.8rem;padding:.6rem 1.0rem .7rem;width:100%}input[type='color']:focus,input[type='date']:focus,input[type='datetime']:focus,input[type='datetime-local']:focus,input[type='email']:focus,input[type='month']:focus,input[type='number']:focus,input[type='password']:focus,input[type='search']:focus,input[type='tel']:focus,input[type='text']:focus,input[type='url']:focus,input[type='week']:focus,input:not([type]):focus,textarea:focus,select:focus{border-color:#9b4dca;outline:0}select{background:url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 30 8" width="30"><path fill="%23d1d1d1" d="M0,0l6,8l6-8"></path></svg>') center right no-repeat;padding-right:3.0rem}select:focus{background-image:url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 30 8" width="30"><path fill="%239b4dca" d="M0,0l6,8l6-8"></path></svg>')}select[multiple]{background:none;height:auto}textarea{min-height:6.5rem}label,legend{display:block;font-size:1.6rem;font-weight:700;margin-bottom:.5rem}fieldset{border-width:0;padding:0}input[type='checkbox'],input[type='radio']{display:inline}.label-inline{display:inline-block;font-weight:normal;margin-left:.5rem}.container{margin:0 auto;max-width:112.0rem;padding:0 2.0rem;position:relative;width:100%}.row{display:flex;flex-direction:column;padding:0;width:100%}.row.row-no-padding{padding:0}.row.row-no-padding>.column{padding:0}.row.row-wrap{flex-wrap:wrap}.row.row-top{align-items:flex-start}.row.row-bottom{align-items:flex-end}.row.row-center{align-items:center}.row.row-stretch{align-items:stretch}.row.row-baseline{align-items:baseline}.row .column{display:block;flex:1 1 auto;margin-left:0;max-width:100%;width:100%}.row .column.column-offset-10{margin-left:10%}.row .column.column-offset-20{margin-left:20%}.row .column.column-offset-25{margin-left:25%}.row .column.column-offset-33,.row .column.column-offset-34{margin-left:33.3333%}.row .column.column-offset-40{margin-left:40%}.row .column.column-offset-50{margin-left:50%}.row .column.column-offset-60{margin-left:60%}.row .column.column-offset-66,.row .column.column-offset-67{margin-left:66.6666%}.row .column.column-offset-75{margin-left:75%}.row .column.column-offset-80{margin-left:80%}.row .column.column-offset-90{margin-left:90%}.row .column.column-10{flex:0 0 10%;max-width:10%}.row .column.column-20{flex:0 0 20%;max-width:20%}.row .column.column-25{flex:0 0 25%;max-width:25%}.row .column.column-33,.row .column.column-34{flex:0 0 33.3333%;max-width:33.3333%}.row .column.column-40{flex:0 0 40%;max-width:40%}.row .column.column-50{flex:0 0 50%;max-width:50%}.row .column.column-60{flex:0 0 60%;max-width:60%}.row .column.column-66,.row .column.column-67{flex:0 0 66.6666%;max-width:66.6666%}.row .column.column-75{flex:0 0 75%;max-width:75%}.row .column.column-80{flex:0 0 80%;max-width:80%}.row .column.column-90{flex:0 0 90%;max-width:90%}.row .column .column-top{align-self:flex-start}.row .column .column-bottom{align-self:flex-end}.row .column .column-center{align-self:center}@media (min-width: 40rem){.row{flex-direction:row;margin-left:-1.0rem;width:calc(100% + 2.0rem)}.row .column{margin-bottom:inherit;padding:0 1.0rem}}a{color:#9b4dca;text-decoration:none}a:focus,a:hover{color:#606c76}dl,ol,ul{list-style:none;margin-top:0;padding-left:0}dl dl,dl ol,dl ul,ol dl,ol ol,ol ul,ul dl,ul ol,ul ul{font-size:90%;margin:1.5rem 0 1.5rem 3.0rem}ol{list-style:decimal inside}ul{list-style:circle inside}.button,button,dd,dt,li{margin-bottom:1.0rem}fieldset,input,select,textarea{margin-bottom:1.5rem}blockquote,dl,figure,form,ol,p,pre,table,ul{margin-bottom:2.5rem}table{border-spacing:0;display:block;overflow-x:auto;text-align:left;width:100%}td,th{border-bottom:0.1rem solid #e1e1e1;padding:1.2rem 1.5rem}td:first-child,th:first-child{padding-left:0}td:last-child,th:last-child{padding-right:0}@media (min-width: 40rem){table{display:table;overflow-x:initial}}b,strong{font-weight:bold}p{margin-top:0}h1,h2,h3,h4,h5,h6{font-weight:300;letter-spacing:-.1rem;margin-bottom:2.0rem;margin-top:0}h1{font-size:4.6rem;line-height:1.2}h2{font-size:3.6rem;line-height:1.25}h3{font-size:2.8rem;line-height:1.3}h4{font-size:2.2rem;letter-spacing:-.08rem;line-height:1.35}h5{font-size:1.8rem;letter-spacing:-.05rem;line-height:1.5}h6{font-size:1.6rem;letter-spacing:0;line-height:1.4}img{max-width:100%}.clearfix:after{clear:both;content:' ';display:table}.float-left{float:left}.float-right{float:right}
            /* General */
            html{font-size:55%}body{color:#000}.section-title blockquote{border:.3rem solid #d1d1d1;background-color:#90C0D8}h3{margin-bottom:.2rem}.container{margin:0;max-width:100%}hr{border-top:.3rem solid #d1d1d1;margin:2rem 0}ul{list-style:none}.location-details li{margin-bottom:0}@media print{.section-title:not(:first-child){page-break-before:always}.keeptogether{break-inside:avoid}}
            /* Responsive tables */
            .rtable{margin:0 0 40px 0;width:100%;box-shadow:0 1px 3px rgba(0,0,0,.2);display:table}@media screen and (max-width:580px){.rtable{display:block}}.rrow{display:table-row}.rrow:nth-of-type(odd){background-color:#fff}.rrow.rheader{font-weight:600;background:#d1d1d1}@media screen and (max-width:1024px){.rrow{padding:14px 0 7px;display:block;border-bottom:1px solid #d1d1d1}.rrow.rheader{padding:0;height:6px}.rrow.rheader .rcell{display:none}.rrow .rcell{margin-bottom:10px;border:none}.rrow .rcell:before{margin-bottom:3px;content:attr(data-title);min-width:98px;font-size:.85em;line-height:10px;font-weight:700;text-transform:uppercase;display:block}}.rcell{padding:6px 12px;display:table-cell;border-bottom:1px solid #d1d1d1}.rcell ul{list-style:none}.rcell li{margin-bottom:0}@media screen and (max-width:1024px){.rcell{padding:2px 16px;display:block}}@media print{html{font-size:40%}.rtable{box-shadow:none;border:1px solid #d1d1d1}.rcell{border:1px solid #d1d1d1}.location-details .row{flex-direction:row}}
         </style></head>
      <body>
         <header>
            <h1>Heritage Assets</h1>
         </header>
         <main>
            {% for resource in resources %}
            {% with resource_data=resource.resource %}
            <section class="section-title">
               <blockquote>
                  {% if resource_data|has_key:"Heritage Asset Names" %}
                  {% for n in resource_data|val_from_key:"Heritage Asset Names" %}
                     {% if n|has_key:"Asset Name Use Type" %}
                     {% if n|val_from_key:"Asset Name Use Type"|val_from_key:"@display_value" == "Primary" %}
                        <h2>{{ n|val_from_key:"Asset Name"|val_from_key:"@display_value" }}</h2>
                     {% endif %}
                     {% endif %}
                  {% endfor %}
                  {% endif%}
                  <p><strong>Primary Reference Number: </strong>{{ resource_data|val_from_key:"System Reference Numbers"|val_from_key:"PrimaryReferenceNumber"|val_from_key:"Primary Reference Number"|val_from_key:"@display_value" }}<br>
                     <strong>ResourceID: </strong>{{ resource_data.resourceinstanceid }}
                  </p>
               </blockquote>
            </section>
            <section>
               <div class="container location-details">
                  {% if resource_data|has_key:"Location Data" %}
                  <div class="row">
                     <div class="column">
                        <div>
                           <h3>OSGB Reference</h3>
                           <p>{% if resource_data|val_from_key:"Location Data"|has_key:"National Grid References" %}
                              {{ resource_data|val_from_key:"Location Data"|val_from_key:"National Grid References"|val_from_key:"National Grid Reference"|val_from_key:"@display_value" }}
                              {% endif %}
                           </p>
                        </div>
                     </div>
                     <div class="column">
                        <div class="keeptogether"></div>
                        <div class="keeptogether">
                           <div>
                              <h3>Named Location</h3>
                              <p>{% if resource_data|val_from_key:"Location Data"|has_key:"Addresses" %}
                           {% for address in resource_data|val_from_key:"Location Data"|val_from_key:"Addresses"|val_from_key:"@display_value" %}
                              {% if address|has_key:"Address Status" %}
                              {% if address|val_from_key:"Address Status"|val_from_key:"@display_value" == "Primary" %}
                                 {{ address|val_from_key:"Full Address"|val_from_key:"@display_value" }}
                              {% endif %}
                              {% endif %}
                           {% endfor %}
                           {% endif %}
                           </p>
                           </div>
                        </div>
                     </div>
                     <div class="column">
                        <div class="keeptogether"></div>
                        <div class="keeptogether">
                           <div>
                              <h3>Localities/Administrative Areas</h3>
                           <p>{% if resource_data|val_from_key:"Location Data"|has_key:"Localities/Administrative Areas" %}
                           {% for area in resource_data|val_from_key:"Location Data"|val_from_key:"Localities/Administrative Areas" %}
                              <b>{{ area|val_from_key:"Area Type"|val_from_key:"@value" }}:</b> {{ area|val_from_key:"Area Names"|val_from_key:"Area Name"|val_from_key:"@display_value" }}
                           {% endfor %}
                           {% endif %}
                           </p>
                           </div>
                        </div>
                     </div>
                  </div>
                  {% endif %}
               </div>
            </section>
            <hr>
            <section>
               <div class="container">
                  {% if resource_data|has_key:"Descriptions" %}
                  <div class="keeptogether">
                     {% for desc in resource_data|val_from_key:"Descriptions" %}
                     <h3>{{ desc|val_from_key:"Description Type"|val_from_key:"@display_value" }}</h3>
                     <p>{{ desc|val_from_key:"Description"|val_from_key:"@display_value" }}</p>
                     {% endfor %}
                  </div>
                  {% endif %}
                  {% if resource_data|has_key:"External Cross References" %}
                  <div class="keeptogether">
                     <h3>External Cross References</h3>
                     <div class="rtable">
                        <div class="rrow rheader">
                           <div class="rcell">Number</div>
                           <div class="rcell">Description</div>
                           <div class="rcell">Source</div>
                        </div>
                        {% for src in resource_data|val_from_key:"External Cross References" %}
                        <div class="rrow">
                           <div class="rcell" data-title="Number">{{ src|val_from_key:"External Cross Reference Number"|val_from_key:"@display_value" }}</div>
                           <div class="rcell" data-title="Description">
                              {% if src|has_key:"External Cross Reference Notes" %}
                              {{ src|val_from_key:"External Cross Reference Notes"|val_from_key:"External Cross Reference Description"|val_from_key:"@display_value" }}
                              {% endif %}
                              </div>
                           <div class="rcell" data-title="Source">{{ src|val_from_key:"External Cross Reference Source"|val_from_key:"@display_value" }}</div>
                        </div>
                        {% endfor %}
                     </div>
                  </div>
                  {% endif %}
               </div>
            </section>
            {% endwith%}
            {% endfor %}
         </main>
      </body>
   </html>


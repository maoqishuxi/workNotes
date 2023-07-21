Odoo docs

Getting started

### Environment configuration

1. *Modules* may also be referred to as *addons* and the directories where the Odoo server finds them form the `addons_path`.

2. Each module is a directory within a *module directory*. Module directories are specified by using the [`--addons-path`](https://www.odoo.com/documentation/16.0/developer/reference/cli.html#cmdoption-odoo-bin-addons-path) option.

3. An Odoo module is declared by its [manifest](https://www.odoo.com/documentation/16.0/developer/reference/backend/module.html#reference-module-manifest).

4. When an Odoo module includes business objects (i.e. Python files), they are organized as a [Python package](https://docs.python.org/3/tutorial/modules.html#packages) with a `__init__.py` file. This file contains import instructions for various Python files in the module.

   ```
   module
   ├── models
   │   ├── *.py
   │   └── __init__.py
   ├── data
   │   └── *.xml
   ├── __init__.py
   └── __manifest__.py
   ```

### The Real Estate Advertisement module

1. **Reference**: the documentation related to this topic can be found in [manifest](https://www.odoo.com/documentation/16.0/developer/reference/backend/module.html#reference-module-manifest).

   1. A module must contain at least 2 files: the `__manifest__.py` file and a `__init__.py` file.
   2.  The `__init__.py` file can remain empty for now and we’ll come back to it in the next chapter.
   3. Its only required field is the `name`, but it usually contains much more information.
   4. A dependency means that the Odoo framework will ensure that these modules are installed before our module is installed.
   5. Restart the Odoo server and go to Apps. Click on Update Apps List, search for `estate` and… tadaaa, your module appears! Did it not appear? Maybe try removing the default ‘Apps’ filter 
   6. Remember to enable the [developer mode](https://www.odoo.com/documentation/16.0/applications/general/developer_mode.html#developer-mode) as explained in the previous chapter. You won’t see the Update Apps List button otherwise.
   7. Add the appropriate key to your `__manifest__.py` so that the module appears when the ‘Apps’ filter is on.
      `Application: True`

2. ### Object-Relational Mapping

   1. Models can be configured by setting attributes in their definition. The most important attribute is `_name`, which is required and defines the name for the model in the Odoo system. Here is a minimum definition of a model:

      ```
      from odoo import models
      
      class TestModel(models.Model):
          _name = "test_model"
      ```

      This definition is enough for the ORM to generate a database table named `test_model`

   2. <span style="color: orange">Any modification of the Python files requires a restart of the Odoo server.</span>

   3. Add a `_description` to your model to get rid of one of the warnings.
      `_description`: the model’s informal name

   4. ```
      from odoo import fields, models
      
      class TestModel(models.Model):
          _name = "test_model"
          _description = "Test Model"
      
          name = fields.Char()
      ```

      The `name` field is a [`Char`](https://www.odoo.com/documentation/16.0/developer/reference/backend/orm.html#odoo.fields.Char) which will be represented as a Python unicode `str` and a SQL `VARCHAR`.

   5. There are two broad categories of fields: ‘simple’ fields, which are atomic values stored directly in the model’s table, and ‘relational’ fields, which link records (of the same or different models).
      Simple field examples are [`Boolean`](https://www.odoo.com/documentation/16.0/developer/reference/backend/orm.html#odoo.fields.Boolean), [`Float`](https://www.odoo.com/documentation/16.0/developer/reference/backend/orm.html#odoo.fields.Float), [`Char`](https://www.odoo.com/documentation/16.0/developer/reference/backend/orm.html#odoo.fields.Char), [`Text`](https://www.odoo.com/documentation/16.0/developer/reference/backend/orm.html#odoo.fields.Text), [`Date`](https://www.odoo.com/documentation/16.0/developer/reference/backend/orm.html#odoo.fields.Date) and [`Selection`](https://www.odoo.com/documentation/16.0/developer/reference/backend/orm.html#odoo.fields.Selection).

   6. Much like the model itself, fields can be configured by passing configuration attributes as parameters:
      `name = fields.Char(required=True)`

   7. Some attributes are available on all fields, here are the most common ones:

      `string` (`str`, default: field’s name)

      ​	The label of the field in UI (visible by users).

      `required` (`bool`, default: `False`)

      ​	If `True`, the field can not be empty. It must either have a default 	value or always be given a value when creating a record.

      `help` (`str`, default: `''`)

      ​	Provides long-form help tooltip for users in the UI.

      `index` (`bool`, default: `False`)

      ​	Requests that Odoo create a [database index](https://use-the-index-luke.com/sql/preface) on the column.

3. ### Automatic Fields

   1. **Reference**: the documentation related to this topic can be found in [Automatic fields](https://www.odoo.com/documentation/16.0/developer/reference/backend/orm.html#reference-fields-automatic).

   2. You may have noticed your model has a few fields you never defined. Odoo creates a few fields in all models[1](https://www.odoo.com/documentation/16.0/developer/tutorials/getting_started/04_basicmodel.html#autofields). These fields are managed by the system and can’t be written to, but they can be read if useful or necessary:

      `id` (`Id`)

      The unique identifier for a record of the model.

      `create_date` ([`Datetime`](https://www.odoo.com/documentation/16.0/developer/reference/backend/orm.html#odoo.fields.Datetime))

      Creation date of the record.

      `create_uid` ([`Many2one`](https://www.odoo.com/documentation/16.0/developer/reference/backend/orm.html#odoo.fields.Many2one))

      User who created the record.

      `write_date` ([`Datetime`](https://www.odoo.com/documentation/16.0/developer/reference/backend/orm.html#odoo.fields.Datetime))

      Last modification date of the record.

      `write_uid` ([`Many2one`](https://www.odoo.com/documentation/16.0/developer/reference/backend/orm.html#odoo.fields.Many2one))

      User who last modified the record.

4. ### Security - A Brief Introduction

   1. The topic of security is covered in more detail in [Restrict access to data](https://www.odoo.com/documentation/16.0/developer/tutorials/restrict_data_access.html). This chapter aims to cover the minimum required for our new module.

      ```
      "id","country_id:id","name","code"
      state_au_1,au,"Australian Capital Territory","ACT"
      state_au_2,au,"New South Wales","NSW"
      state_au_3,au,"Northern Territory","NT"
      state_au_4,au,"Queensland","QLD"
      ...
      ```

      - `id` is an [external identifier](https://www.odoo.com/documentation/16.0/developer/glossary.html#term-external-identifier). It can be used to refer to the record (without knowing its in-database identifier).
      - `country_id:id` refers to the country by using its [external identifier](https://www.odoo.com/documentation/16.0/developer/glossary.html#term-external-identifier).
      - `name` is the name of the state.
      - `code` is the code of the state.

   2. all of these files must be declared in the `data` list within the `__manifest__.py` file.

   3. <span style="color: orange">The data files are sequentially loaded following their order in the `__manifest__.py` file. This means that if data `A` refers to data `B`, you must make sure that `B` is loaded before `A`.</span>

   4. Access rights are defined as records of the model `ir.model.access`. Each access right is associated with a model, a group (or no group for global access) and a set of permissions: create, read, write and unlink[2](https://www.odoo.com/documentation/16.0/developer/tutorials/getting_started/05_securityintro.html#unlink). Such access rights are usually defined in a CSV file named `ir.model.access.csv`.

   5. ```
      id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
      access_test_model,access_test_model,model_test_model,base.group_user,1,0,0,0
      ```

      - `id` is an [external identifier](https://www.odoo.com/documentation/16.0/developer/glossary.html#term-external-identifier).
      - `name` is the name of the `ir.model.access`.
      - `model_id/id` refers to the model which the access right applies to. The standard way to refer to the model is `model_<model_name>`, where `<model_name>` is the `_name` of the model with the `.` replaced by `_`. Seems cumbersome? Indeed it is…
      - `group_id/id` refers to the group which the access right applies to.
      - `perm_read,perm_write,perm_create,perm_unlink`: read, write, create and unlink permissions

   6. The XML files must be added to the same folders as the CSV files and defined similarly in the `__manifest__.py`. The content of the data files is also sequentially loaded when a module is installed or updated, therefore all remarks made for CSV files hold true for XML files. When the data is linked to views, we add them to the `views` folder.

5. A common pattern is Menu > Action > View. To access records the user navigates through several menu levels; the deepest level is an action which triggers the opening of a list of the records.

   1. Actions can be triggered in three ways:

      by clicking on menu items (linked to specific actions)

      by clicking on buttons in views (if these are connected to actions)

      as contextual actions on object

   2. A basic action for our `test.model` is:

      ```
      <record id="test_model_action" model="ir.actions.act_window">
          <field name="name">Test action</field>
          <field name="res_model">test.model</field>
          <field name="view_mode">tree,form</field>
      </record>
      ```

      id is an external identifier. It can be used to refer to the record (without knowing its in-database identifier).

      model has a fixed value of ir.actions.act_window (Window Actions (ir.actions.act_window)).

      name is the name of the action.

      res_model is the model which the action applies to.

      view_mode are the views that will be available; in this case they are the list (tree) and form views. We’ll see later that there can be other view modes.

   3. To reduce the complexity in declaring a menu (`ir.ui.menu`) and connecting it to the corresponding action, we can use the `<menuitem>` shortcut .

   4. A basic menu for our `test_model_action` is:

      ```
      <menuitem id="test_model_menu_action" action="test_model_action"/>
      ```

      The menu `test_model_menu_action` is linked to the action `test_model_action`, and the action is linked to the model `test.model`.

   5. The easiest way to define the structure is to create it in the XML file. A basic structure for our `test_model_action` is:

      ```
      <menuitem id="test_menu_root" name="Test">
          <menuitem id="test_first_level_menu" name="First Level">
              <menuitem id="test_model_menu_action" action="test_model_action"/>
          </menuitem>
      </menuitem>
      ```

   6. Default Values
      Any field can be given a default value. In the field definition, add the option `default=X` where `X` is either a Python literal value (boolean, integer, float, string) or a function taking a model and returning a value:

      ```
      name = fields.Char(default="Unknown")
      last_seen = fields.Datetime("Last Seen", default=lambda self: fields.Datetime.now())
      ```

   7. Reserved Fields
      Restart the server, create a new property, then come back to the list view… The property will not be listed! `active` is an example of a reserved field with a specific behavior: when a record has `active=False`, it is automatically removed from any search. To display the created property, you will need to specifically search for inactive records.

6. Basic Views

   1. Views are defined in XML files with actions and menus. They are instances of the `ir.ui.view` model.

   2. List
      The root element of list views is `<tree>`
      List views, also called tree views, display records in a tabular form.
      Their root element is `<tree>`. The most basic version of this view simply lists all the fields to display in the table (where each field is a column):

      ```
      <tree string="Tests">
          <field name="name"/>
          <field name="last_seen"/>
      </tree>
      ```

   3. <span style="color: orange">You will probably use some copy-paste in this chapter, therefore always make sure that the `id` remains unique for each view!</span>

   4. Form
      Form views are used to display the data from a single record. Their root element is `<form>`. They are composed of regular [HTML](https://en.wikipedia.org/wiki/HTML) with additional structural and semantic components.

   5. ```
      notebook
      ```

      defines a tabbed section. Each tab is defined through a `page` child element. Pages can have the following attributes:

      - `string` (required)

        the title of the tab

      - 

      - ```
        group
        ```

        used to define column layouts in forms. By default, groups define 2 columns and most direct children of groups take a single column. `field` direct children of groups display a label by default, and the label and the field itself have a colspan of 1 each.

        The number of columns in a `group` can be customized using the `col` attribute, the number of columns taken by an element can be customized using `colspan`.

        Children are laid out horizontally (tries to fill the next column before changing row).

        Groups can have a `string` attribute, which is displayed as the group’s title

      - ```
        sheet
        ```

        can be used as a direct child to `form` for a narrower and more responsive form layout

      - ```
        header
        ```

        combined with `sheet`, provides a full-width location above the sheet itself, generally used to display workflow buttons and status widgets

   6. Their root element is `<form>`. They are composed of high-level structure elements (groups and notebooks) and interactive elements (buttons and fields):

      ```
      <form string="Test">
          <sheet>
              <group>
                  <group>
                      <field name="name"/>
                  </group>
                  <group>
                      <field name="last_seen"/>
                  </group>
                  <notebook>
                      <page string="Description">
                          <field name="description"/>
                      </page>
                  </notebook>
              </group>
          </sheet>
      </form>
      ```

   7. In order to avoid relaunching the server every time you do a modification to the view, it can be convenient to use the `--dev xml` parameter when launching the server:

7. Relations Between Models

   1. Many2one
      A property can have **one** type, but the same type can be assigned to **many** properties. This is supported by the **many2one** concept.
   
   2. A many2one is a simple link to another object. For example, in order to define a link to the `res.partner` in our test model, we can write:
   
      ```
      partner_id = fields.Many2one("res.partner", string="Partner")
      ```
   
   3. By convention, many2one fields have the `_id` suffix. Accessing the data in the partner can then be easily done with:
   
      ```
      print(my_test_object.partner_id.name)
      ```
   
   4. In practice a many2one can be seen as a dropdown list in a form view.
   
   5. <span style="color: orange">Tip: do not forget to import any new Python files in `__init__.py`, add new data files in `__manifest.py__` or add the access rights ;-)</span>
   
   6. In Odoo, there are two models which we commonly refer to:
   
      - `res.partner`: a partner is a physical or legal entity. It can be a company, an individual or even a contact address.
      - `res.users`: the users of the system. Users can be ‘internal’, i.e. they have access to the Odoo backend. Or they can be ‘portal’, i.e. they cannot access the backend, only the frontend (e.g. to access their previous orders in eCommerce).
   
   7. The object `self.env` gives access to request parameters and other useful things:
   
      - `self.env.cr` or `self._cr` is the database *cursor* object; it is used for querying the database
      - `self.env.uid` or `self._uid` is the current user’s database id
      - `self.env.user` is the current user’s record
      - `self.env.context` or `self._context` is the context dictionary
      - `self.env.ref(xml_id)` returns the record corresponding to an XML id
      - `self.env[model_name]` returns an instance of the given model
   
   8. Many2many
   
      1. A property can have **many** tags and a tag can be assigned to **many** properties. This is supported by the **many2many** concept.
   
      2. A many2many is a bidirectional multiple relationship: any record on one side can be related to any number of records on the other side. For example, in order to define a link to the `account.tax` model on our test model, we can write:
   
         ```
         tax_ids = fields.Many2many("account.tax", string="Taxes")
         ```
   
      3. By convention, many2many fields have the `_ids` suffix. This means that several taxes can be added to our test model. It behaves as a list of records, meaning that accessing the data must be done in a loop:
   
         ```
         for tax in my_test_object.tax_ids:
             print(tax.name)
         ```
   
   9. One2many
   
      1. A one2many is the inverse of a many2one. For example, we defined on our test model a link to the `res.partner` model thanks to the field `partner_id`. We can define the inverse relation, i.e. the list of test models linked to our partner:
   
         ```
         test_ids = fields.One2many("test.model", "partner_id", string="Tests")
         ```
   
         The first parameter is called the `comodel` and the second parameter is the field we want to inverse.
   
         By convention, one2many fields have the `_ids` suffix. They behave as a list of records, meaning that accessing the data must be done in a loop:
   
         ```
         for test in partner.test_ids:
             print(test.name)
         ```
   
      2. <span style="color: orange">Because a [`One2many`](https://www.odoo.com/documentation/16.0/developer/reference/backend/orm.html#odoo.fields.One2many) is a virtual relationship, there *must* be a [`Many2one`](https://www.odoo.com/documentation/16.0/developer/reference/backend/orm.html#odoo.fields.Many2one) field defined in the comodel.</span>
   
      3. There are several important things to notice here. First, we don’t need an action or a menu for all models. Some models are intended to be accessed only through another model. This is the case in our exercise: an offer is always accessed through a property.
   
         Second, despite the fact that the `property_id` field is required, we did not include it in the views. How does Odoo know which property our offer is linked to? Well that’s part of the magic of using the Odoo framework: sometimes things are defined implicitly. When we create a record through a one2many field, the corresponding many2one is populated automatically for convenience.
   
   10. Computed Fields
   
       1. To create a computed field, create a field and set its attribute `compute` to the name of a method. The computation method should set the value of the computed field for every record in `self`.
   
       2. By convention, `compute` methods are private, meaning that they cannot be called from the presentation tier, only from the business tier (see [Chapter 1: Architecture Overview](https://www.odoo.com/documentation/16.0/developer/tutorials/getting_started/01_architecture.html#tutorials-getting-started-01-architecture)). Private methods have a name starting with an underscore `_`.
   
       3. Dependencies
   
       4. The value of a computed field usually depends on the values of other fields in the computed record. The ORM expects the developer to specify those dependencies on the compute method with the decorator [`depends()`](https://www.odoo.com/documentation/16.0/developer/reference/backend/orm.html#odoo.api.depends). The given dependencies are used by the ORM to trigger the recomputation of the field whenever some of its dependencies have been modified:
   
          ```
          from odoo import api, fields, models
          
          class TestComputed(models.Model):
              _name = "test.computed"
          
              total = fields.Float(compute="_compute_total")
              amount = fields.Float()
          
              @api.depends("amount")
              def _compute_total(self):
                  for record in self:
                      record.total = 2.0 * record.amount
          ```
   
       5. `self` is a collection.
   
          The object `self` is a *recordset*, i.e. an ordered collection of records. It supports the standard Python operations on collections, e.g. `len(self)` and `iter(self)`, plus extra set operations such as `recs1 | recs2`.
   
          Iterating over `self` gives the records one by one, where each record is itself a collection of size 1. You can access/assign fields on single records by using the dot notation, e.g. `record.name`.
   
       6. For relational fields it’s possible to use paths through a field as a dependency:
   
          ```
          description = fields.Char(compute="_compute_description")
          partner_id = fields.Many2one("res.partner")
          
          @api.depends("partner_id.name")
          def _compute_description(self):
              for record in self:
                  record.description = "Test for partner %s" % record.partner_id.name
          ```
   
       7. Inverse Function
          You might have noticed that computed fields are read-only by default. This is expected since the user is not supposed to set a value.
   
       8. In some cases, it might be useful to still be able to set a value directly. In our real estate example, we can define a validity duration for an offer and set a validity date. We would like to be able to set either the duration or the date with one impacting the other.
   
          ```
          from odoo import api, fields, models
          
          class TestComputed(models.Model):
              _name = "test.computed"
          
              total = fields.Float(compute="_compute_total", inverse="_inverse_total")
              amount = fields.Float()
          
              @api.depends("amount")
              def _compute_total(self):
                  for record in self:
                      record.total = 2.0 * record.amount
          
              def _inverse_total(self):
                  for record in self:
                      record.amount = record.total / 2.0
          ```
   
       9. Note that the `inverse` method is called when saving the record, while the `compute` method is called at each change of its dependencies.
   
       10. Additional Information
           Computed fields are **not stored** in the database by default. Therefore it is **not possible** to search on a computed field unless a `search` method is defined. 
   
       11. Another solution is to store the field with the `store=True` attribute. While this is usually convenient, pay attention to the potential computation load added to your model. Lets re-use our example:
   
           ```
           description = fields.Char(compute="_compute_description", store=True)
           partner_id = fields.Many2one("res.partner")
           
           @api.depends("partner_id.name")
           def _compute_description(self):
               for record in self:
                   record.description = "Test for partner %s" % record.partner_id.name
           ```
       
       12. It is also worth noting that a computed field can depend on another computed field. The ORM is smart enough to correctly recompute all the dependencies in the right order… but sometimes at the cost of degraded performance.
       
       13. Onchanges
           The ‘onchange’ mechanism provides a way for the client interface to update a form without saving anything to the database whenever the user has filled in a field value. To achieve this, we define a method where `self` represents the record in the form view and decorate it with [`onchange()`](https://www.odoo.com/documentation/16.0/developer/reference/backend/orm.html#odoo.api.onchange) to specify which field it is triggered by. Any change you make on `self` will be reflected on the form:
       
           ```
           from odoo import api, fields, models
           
           class TestOnchange(models.Model):
               _name = "test.onchange"
           
               name = fields.Char(string="Name")
               description = fields.Char(string="Description")
               partner_id = fields.Many2one("res.partner", string="Partner")
           
               @api.onchange("partner_id")
               def _onchange_partner_id(self):
                   self.name = "Document for %s" % (self.partner_id.name)
                   self.description = "Default description for %s" % (self.partner_id.name)
           ```
       
       14. Action Type
       
           - Add a button in the view, for example in the `header` of the view:
       
             ```
             <form>
                 <header>
                     <button name="action_do_something" type="object" string="Do Something"/>
                 </header>
                 <sheet>
                     <field name="name"/>
                 </sheet>
             </form>
             ```
       
           - and link this button to business logic:
       
             ```
             from odoo import fields, models
             
             class TestAction(models.Model):
                 _name = "test.action"
             
                 name = fields.Char()
             
                 def action_do_something(self):
                     for record in self:
                         record.name = "Something"
                     return True
             ```
       
             By assigning `type="object"` to our button, the Odoo framework will execute a Python method with `name="action_do_something"` on the given model.
       
           - The first important detail to note is that our method name isn’t prefixed with an underscore (`_`). This makes our method a **public** method, which can be called directly from the Odoo interface (through an RPC call). Until now, all methods we created (compute, onchange) were called internally, so we used **private** methods prefixed by an underscore. You should always define your methods as private unless they need to be called from the user interface.
       
           - 


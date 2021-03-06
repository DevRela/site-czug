===========
分组表单
===========
.. Contents::
.. sectnum::

Group forms allow you to split up a form into several logical units without
much overhead. To the parent form, groups should be only dealt with during
coding and be transparent on the data extraction level.

准备
--------

For the examples to work, we have to bring up most of the form framework:

  >>> from z3c.form import testing
  >>> testing.setupFormDefaults()

So let's first define a complex content component that warrants setting up
multiple groups:

  >>> import zope.interface
  >>> import zope.schema

  >>> class IVehicleRegistration(zope.interface.Interface):
  ...     firstName = zope.schema.TextLine(title=u'First Name')
  ...     lastName = zope.schema.TextLine(title=u'Last Name')
  ...
  ...     license = zope.schema.TextLine(title=u'License')
  ...     address = zope.schema.TextLine(title=u'Address')
  ...
  ...     model = zope.schema.TextLine(title=u'Model')
  ...     make = zope.schema.TextLine(title=u'Make')
  ...     year = zope.schema.Int(title=u'Year')

  >>> class VehicleRegistration(object):
  ...     zope.interface.implements(IVehicleRegistration)
  ...
  ...     def __init__(self, **kw):
  ...         for name, value in kw.items():
  ...             setattr(self, name, value)

定义分组
----------------
The schema above can be separated into basic, license, and car information,
where the latter two will be placed into groups. First we create the two
groups:

  >>> from z3c.form import field, group

  >>> class LicenseGroup(group.Group):
  ...     label = u'License'
  ...     fields = field.Fields(IVehicleRegistration).select(
  ...         'license', 'address')

  >>> class CarGroup(group.Group):
  ...     label = u'Car'
  ...     fields = field.Fields(IVehicleRegistration).select(
  ...         'model', 'make', 'year')

Most of the group is setup like any other (sub)form. Additionally, you can
specify a label, which is a human-readable string that can be used for layout
purposes.

使用分组
--------------
Let's now create an add form for the entire vehicle registration. In
comparison to a regular add form, you only need to add the ``GroupForm`` as
one of the base classes. The groups are specified in a simple tuple:

  >>> import os
  >>> from zope.app.pagetemplate import viewpagetemplatefile
  >>> from z3c.form import form, tests

  >>> class RegistrationAddForm(group.GroupForm, form.AddForm):
  ...     fields = field.Fields(IVehicleRegistration).select(
  ...         'firstName', 'lastName')
  ...     groups = (LicenseGroup, CarGroup)
  ...
  ...     template = viewpagetemplatefile.ViewPageTemplateFile(
  ...         'simple_groupedit.pt', os.path.dirname(tests.__file__))
  ...
  ...     def create(self, data):
  ...         return VehicleRegistration(**data)
  ...
  ...     def add(self, object):
  ...         self.getContent()['obj1'] = object
  ...         return object


Note: The order of the base classes is very important here. The ``GroupForm``
class must be left of the ``AddForm`` class, because the ``GroupForm`` class
overrides some methods of the ``AddForm`` class.

分组的html输出结果
----------------------------
Now we can instantiate the form:

  >>> request = testing.TestRequest()

  >>> add = RegistrationAddForm(None, request)
  >>> add.update()

After the form is updated the tuple of group classes is converted to group
instances:

  >>> add.groups
  (<LicenseGroup object at ...>, <CarGroup object at ...>)

We can now render the form:

  >>> print add.render()
  <html>
    <body>
      <form action=".">
        <div class="row">
          <label for="form-widgets-firstName">First Name</label>
          <input type="text" id="form-widgets-firstName"
                 name="form.widgets.firstName"
                 class="text-widget required textline-field"
                 value="" />
        </div>
        <div class="row">
          <label for="form-widgets-lastName">Last Name</label>
          <input type="text" id="form-widgets-lastName"
                 name="form.widgets.lastName"
                 class="text-widget required textline-field"
                 value="" />
        </div>
        <fieldgroup>
          <legend>License</legend>
          <div class="row">
            <label for="form-widgets-license">License</label>
            <input type="text" id="form-widgets-license"
                   name="form.widgets.license"
                   class="text-widget required textline-field"
                   value="" />
          </div>
          <div class="row">
            <label for="form-widgets-address">Address</label>
            <input type="text" id="form-widgets-address"
                   name="form.widgets.address"
                   class="text-widget required textline-field"
                   value="" />
          </div>
        </fieldgroup>
        <fieldgroup>
          <legend>Car</legend>
          <div class="row">
            <label for="form-widgets-model">Model</label>
            <input type="text" id="form-widgets-model"
                   name="form.widgets.model"
                   class="text-widget required textline-field"
                   value="" />
          </div>
          <div class="row">
            <label for="form-widgets-make">Make</label>
            <input type="text" id="form-widgets-make"
                   name="form.widgets.make"
                   class="text-widget required textline-field"
                   value="" />
          </div>
          <div class="row">
            <label for="form-widgets-year">Year</label>
            <input type="text" id="form-widgets-year"
                   name="form.widgets.year"
                   class="text-widget required int-field"
                   value="" />
          </div>
        </fieldgroup>
        <div class="action">
          <input type="submit" id="form-buttons-add"
                 name="form.buttons.add" class="submit-widget button-field"
                 value="Add" />
        </div>
      </form>
    </body>
  </html>

提交除错处理
-------------------
Let's now submit the form, but forgetting to enter the address:

  >>> request = testing.TestRequest(form={
  ...     'form.widgets.firstName': u'Stephan',
  ...     'form.widgets.lastName': u'Richter',
  ...     'form.widgets.license': u'MA 40387',
  ...     'form.widgets.model': u'BMW',
  ...     'form.widgets.make': u'325',
  ...     'form.widgets.year': u'2005',
  ...     'form.buttons.add': u'Add'
  ...     })

  >>> add = RegistrationAddForm(None, request)
  >>> add.update()
  >>> print add.render()
  <html>
    <body>
      <i>There were some errors.</i>
      <form action=".">
        ...
        <fieldgroup>
          <legend>License</legend>
          <ul>
            <li>
              Address: <div class="error">Required input is missing.</div>
            </li>
          </ul>
          ...
        </fieldgroup>
        ...
      </form>
    </body>
  </html>

As you can see, the template is clever enough to just report the errors at the
top of the form, but still report the actual problem within the group. So what
happens, if errors happen inside and outside a group?

  >>> request = testing.TestRequest(form={
  ...     'form.widgets.firstName': u'Stephan',
  ...     'form.widgets.license': u'MA 40387',
  ...     'form.widgets.model': u'BMW',
  ...     'form.widgets.make': u'325',
  ...     'form.widgets.year': u'2005',
  ...     'form.buttons.add': u'Add'
  ...     })

  >>> add = RegistrationAddForm(None, request)
  >>> add.update()
  >>> print add.render()
  <html>
    <body>
      <i>There were some errors.</i>
      <ul>
        <li>
          Last Name: <div class="error">Required input is missing.</div>
        </li>
      </ul>
      <form action=".">
        ...
        <fieldgroup>
          <legend>License</legend>
          <ul>
            <li>
              Address: <div class="error">Required input is missing.</div>
            </li>
          </ul>
          ...
        </fieldgroup>
        ...
      </form>
    </body>
  </html>

Let's now successfully complete the add form.

  >>> from zope.app.container import btree
  >>> context = btree.BTreeContainer()

  >>> request = testing.TestRequest(form={
  ...     'form.widgets.firstName': u'Stephan',
  ...     'form.widgets.lastName': u'Richter',
  ...     'form.widgets.license': u'MA 40387',
  ...     'form.widgets.address': u'10 Main St, Maynard, MA',
  ...     'form.widgets.model': u'BMW',
  ...     'form.widgets.make': u'325',
  ...     'form.widgets.year': u'2005',
  ...     'form.buttons.add': u'Add'
  ...     })

  >>> add = RegistrationAddForm(context, request)
  >>> add.update()

The object is now added to the container and all attributes should be set:

  >>> reg = context['obj1']
  >>> reg.firstName
  u'Stephan'
  >>> reg.lastName
  u'Richter'
  >>> reg.license
  u'MA 40387'
  >>> reg.address
  u'10 Main St, Maynard, MA'
  >>> reg.model
  u'BMW'
  >>> reg.make
  u'325'
  >>> reg.year
  2005

Let's now have a look at an edit form for the vehicle registration:

  >>> class RegistrationEditForm(group.GroupForm, form.EditForm):
  ...     fields = field.Fields(IVehicleRegistration).select(
  ...         'firstName', 'lastName')
  ...     groups = (LicenseGroup, CarGroup)
  ...
  ...     template = viewpagetemplatefile.ViewPageTemplateFile(
  ...         'simple_groupedit.pt', os.path.dirname(tests.__file__))

  >>> request = testing.TestRequest()

  >>> edit = RegistrationEditForm(reg, request)
  >>> edit.update()

After updating the form, we can render the HTML:

  >>> print edit.render()
  <html>
    <body>
      <form action=".">
        <div class="row">
          <label for="form-widgets-firstName">First Name</label>
          <input type="text" id="form-widgets-firstName"
                 name="form.widgets.firstName"
                 class="text-widget required textline-field"
                 value="Stephan" />
        </div>
        <div class="row">
          <label for="form-widgets-lastName">Last Name</label>
          <input type="text" id="form-widgets-lastName"
                 name="form.widgets.lastName"
                 class="text-widget required textline-field"
                 value="Richter" />
         </div>
        <fieldgroup>
          <legend>License</legend>
          <div class="row">
            <label for="form-widgets-license">License</label>
            <input type="text" id="form-widgets-license"
                   name="form.widgets.license"
                   class="text-widget required textline-field"
                   value="MA 40387" />
          </div>
          <div class="row">
            <label for="form-widgets-address">Address</label>
            <input type="text" id="form-widgets-address"
                   name="form.widgets.address"
                   class="text-widget required textline-field"
                   value="10 Main St, Maynard, MA" />
          </div>
        </fieldgroup>
        <fieldgroup>
          <legend>Car</legend>
          <div class="row">
            <label for="form-widgets-model">Model</label>
            <input type="text" id="form-widgets-model"
                   name="form.widgets.model"
                   class="text-widget required textline-field"
                   value="BMW" />
          </div>
          <div class="row">
            <label for="form-widgets-make">Make</label>
            <input type="text" id="form-widgets-make"
                   name="form.widgets.make"
                   class="text-widget required textline-field"
                   value="325" />
          </div>
          <div class="row">
            <label for="form-widgets-year">Year</label>
            <input type="text" id="form-widgets-year"
                   name="form.widgets.year"
                   class="text-widget required int-field"
                   value="2005" />
          </div>
        </fieldgroup>
        <div class="action">
          <input type="submit" id="form-buttons-apply"
                 name="form.buttons.apply" class="submit-widget button-field"
                 value="Apply" />
        </div>
      </form>
    </body>
  </html>

The behavior when an error occurs is identical to that of the add form:

  >>> request = testing.TestRequest(form={
  ...     'form.widgets.firstName': u'Stephan',
  ...     'form.widgets.lastName': u'Richter',
  ...     'form.widgets.license': u'MA 40387',
  ...     'form.widgets.model': u'BMW',
  ...     'form.widgets.make': u'325',
  ...     'form.widgets.year': u'2005',
  ...     'form.buttons.apply': u'Apply'
  ...     })

  >>> edit = RegistrationEditForm(reg, request)
  >>> edit.update()
  >>> print edit.render()
  <html>
    <body>
      <i>There were some errors.</i>
      <form action=".">
        ...
        <fieldgroup>
          <legend>License</legend>
          <ul>
            <li>
              Address: <div class="error">Required input is missing.</div>
            </li>
          </ul>
          ...
        </fieldgroup>
        ...
      </form>
    </body>
  </html>

Let's now complete the form successfully:

  >>> request = testing.TestRequest(form={
  ...     'form.widgets.firstName': u'Stephan',
  ...     'form.widgets.lastName': u'Richter',
  ...     'form.widgets.license': u'MA 4038765',
  ...     'form.widgets.address': u'11 Main St, Maynard, MA',
  ...     'form.widgets.model': u'Ford',
  ...     'form.widgets.make': u'F150',
  ...     'form.widgets.year': u'2006',
  ...     'form.buttons.apply': u'Apply'
  ...     })

  >>> edit = RegistrationEditForm(reg, request)
  >>> edit.update()

The success message will be shown on the form, ...

  >>> print edit.render()
  <html>
    <body>
      <i>Data successfully updated.</i>
      ...
    </body>
  </html>

and the data is correctly updated:

  >>> reg.firstName
  u'Stephan'
  >>> reg.lastName
  u'Richter'
  >>> reg.license
  u'MA 4038765'
  >>> reg.address
  u'11 Main St, Maynard, MA'
  >>> reg.model
  u'Ford'
  >>> reg.make
  u'F150'
  >>> reg.year
  2006

And that's it!


混合不同对象的分组
-----------------------------

You can customize the content for a group by overriding a group's
``getContent`` method.  This is a very easy way to get around not
having object widgets.  

准备
~~~~~~~~~~
For example, suppose we want to maintain the
vehicle owner's information in a separate class than the vehicle.  We
might have an ``IVehicleOwner`` interface like so.

  >>> class IVehicleOwner(zope.interface.Interface):
  ...     firstName = zope.schema.TextLine(title=u'First Name')
  ...     lastName = zope.schema.TextLine(title=u'Last Name')

Then out ``IVehicleRegistration`` interface would include an object
field for the owner instead of the ``firstName`` and ``lastName``
fields.

  >>> class IVehicleRegistration(zope.interface.Interface):
  ...     owner = zope.schema.Object(title=u'Owner', schema=IVehicleOwner)
  ...
  ...     license = zope.schema.TextLine(title=u'License')
  ...     address = zope.schema.TextLine(title=u'Address')
  ...
  ...     model = zope.schema.TextLine(title=u'Model')
  ...     make = zope.schema.TextLine(title=u'Make')
  ...     year = zope.schema.Int(title=u'Year')

Now let's create simple implementations of these two interfaces.

  >>> class VehicleOwner(object):
  ...     zope.interface.implements(IVehicleOwner)
  ...
  ...     def __init__(self, **kw):
  ...         for name, value in kw.items():
  ...             setattr(self, name, value)

  >>> class VehicleRegistration(object):
  ...     zope.interface.implements(IVehicleRegistration)
  ...
  ...     def __init__(self, **kw):
  ...         for name, value in kw.items():
  ...             setattr(self, name, value)

创建/使用组
~~~~~~~~~~~~~`
Now we can create a group just for the owner with its own
``getContent`` method that simply returns the ``owner`` object field
of the ``VehicleRegistration`` instance.

  >>> class OwnerGroup(group.Group):
  ...     label = u'Owner'
  ...     fields = field.Fields(IVehicleOwner, prefix='owner')
  ...
  ...     def getContent(self):
  ...         return self.context.owner

输出
~~~~~~~~~~

When we create an Edit form for example, we should omit the ``owner``
field which is taken care of with the group.

  >>> class RegistrationEditForm(group.GroupForm, form.EditForm):
  ...     fields = field.Fields(IVehicleRegistration).omit(
  ...         'owner')
  ...     groups = (OwnerGroup,)
  ...
  ...     template = viewpagetemplatefile.ViewPageTemplateFile(
  ...         'simple_groupedit.pt', os.path.dirname(tests.__file__))


  >>> reg = VehicleRegistration(
  ...               license=u'MA 40387',
  ...               address=u'10 Main St, Maynard, MA',
  ...               model=u'BMW',
  ...               make=u'325',
  ...               year=u'2005',
  ...               owner=VehicleOwner(firstName=u'Stephan',
  ...                                  lastName=u'Richter'))
  >>> request = testing.TestRequest()

  >>> edit = RegistrationEditForm(reg, request)
  >>> edit.update()

When we render the form, the group appears as we would expect but with
the ``owner`` prefix for the fields.

  >>> print edit.render()
  <html>
    <body>
      <form action=".">
        <div class="row">
          <label for="form-widgets-license">License</label>
          <input type="text" id="form-widgets-license"
                 name="form.widgets.license"
                 class="text-widget required textline-field"
                 value="MA 40387" />
        </div>
        <div class="row">
          <label for="form-widgets-address">Address</label>
          <input type="text" id="form-widgets-address"
                 name="form.widgets.address"
                 class="text-widget required textline-field"
                 value="10 Main St, Maynard, MA" />
        </div>
        <div class="row">
          <label for="form-widgets-model">Model</label>
          <input type="text" id="form-widgets-model"
                 name="form.widgets.model"
                 class="text-widget required textline-field"
                 value="BMW" />
        </div>
        <div class="row">
          <label for="form-widgets-make">Make</label>
          <input type="text" id="form-widgets-make"
                 name="form.widgets.make"
                 class="text-widget required textline-field"
                 value="325" />
        </div>
        <div class="row">
          <label for="form-widgets-year">Year</label>
          <input type="text" id="form-widgets-year"
                 name="form.widgets.year"
                 class="text-widget required int-field" value="2005" />
        </div>
        <fieldgroup>
          <legend>Owner</legend>
          <div class="row">
            <label for="form-widgets-owner-firstName">First Name</label>
            <input type="text" id="form-widgets-owner-firstName"
                   name="form.widgets.owner.firstName"
                   class="text-widget required textline-field"
                   value="Stephan" />
          </div>
          <div class="row">
            <label for="form-widgets-owner-lastName">Last Name</label>
            <input type="text" id="form-widgets-owner-lastName"
                   name="form.widgets.owner.lastName"
                   class="text-widget required textline-field"
                   value="Richter" />
          </div>
        </fieldgroup>
        <div class="action">
          <input type="submit" id="form-buttons-apply"
                 name="form.buttons.apply"
                 class="submit-widget button-field" value="Apply" />
        </div>
      </form>
    </body>
  </html>

Now let's try and edit the owner.  For example, suppose that Stephan
Richter gave his BMW to Paul Carduner because he is such a nice guy.

  >>> request = testing.TestRequest(form={
  ...     'form.widgets.owner.firstName': u'Paul',
  ...     'form.widgets.owner.lastName': u'Carduner',
  ...     'form.widgets.license': u'MA 4038765',
  ...     'form.widgets.address': u'Berkeley',
  ...     'form.widgets.model': u'BMW',
  ...     'form.widgets.make': u'325',
  ...     'form.widgets.year': u'2005',
  ...     'form.buttons.apply': u'Apply'
  ...     })
  >>> edit = RegistrationEditForm(reg, request)
  >>> edit.update()

We'll see if everything worked on the form side.

  >>> print edit.render()
  <html>
    <body>
      <i>Data successfully updated.</i>
      ...
    </body>
  </html>

Now the owner object should have updated fields.

  >>> reg.owner.firstName
  u'Paul'
  >>> reg.owner.lastName
  u'Carduner'
  >>> reg.license
  u'MA 4038765'
  >>> reg.address
  u'Berkeley'
  >>> reg.model
  u'BMW'
  >>> reg.make
  u'325'
  >>> reg.year
  2005

And that's it!

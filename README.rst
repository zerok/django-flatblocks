django-flatblocks
=================

.. attention::

   This project is no longer maintained. Please take a look at
   `jazzband/django-flatblocks
   <https://github.com/jazzband/django-flatblocks/>`_ for a more
   up-to-date version!

django-flatblocks is a simple application for handling small text-blocks on
websites. Think about it like ``django.contrib.flatpages`` just not for a 
whole page but for only parts of it, like an information text describing what
you can do on a site.

Installation
------------

Probably the easiest way to install this application is to first run `pip
install django-flatblocks`.  Once this step is complete add "flatblocks" to
your INSTALLED_APPS setting in your settings.py file and run `python manage.py
syncdb` to update your database.


Upgrading
---------

django-flatblocks uses `South`_ for handling data and schema migrations
starting with version 0.6.0, so the South-typical update path applies here.

If you're upgrading from a 0.5.x version or earlier you will have to migrate
in 3 steps:

1. Install south.

2. Migrate your database to the first version of flatblocks using South::

   ./manage.py migrate flatblocks 0001 --fake

3. Then migrate your dataabase to the latest version of flatblocks' database
   and data structure::

   ./manage.py migrate flatblocks

Usage
------------

Once you've created some instances of the ``flatblocks.models.FlatBlock``
model, you can load it it using the ``flatblock_tags`` templatetag-library::
    
    {% load flatblock_tags %}
    
    <html>
        <head>
            <!-- ... -->
        </head>
        <body>
            <div id="page">
                <div id="main">
                    <!-- ... -->
                </div>
                <div id="sidebar">
                    {% flatblock "page.info" %}
                </div>
            </div>
        </body>
    </html>

This way you can display a text block with the name 'page.info'. If you 
have the name of a block in a template variable, leave out the quotes.

This tag also accepts an optional argument where you can specify the number
of seconds, the that block should be cached::
    
    {% flatblock "page.info" 3600 %}

Additionally you can also specify which template should be used to render the
flatblock::
    
    {% flatblock "page.info" using "my_template.html" %}
    # ...
    {% flatblock "page.about" 3600 using "my_template.html" %}

As with the slug of the flatblock also with the template name you have the
choice of using the literal name of the template or pass it to the templatetag
as a variable.

The content of a flatblock (as well as its header) can also be evaluated as a
full-fledged Django template::
    
    {% flatblock "page.info" evaluated %}

This also works with the other parameters like the cache timeout and the
custom template::
    

    {% flatblock "page.info" evaluated using "my_template.html" %}
    {% flatblock "page.about" 3600 evaluated using "my_template.html" %}


edit-view
---------

With ``flatblocks.views.edit`` django-flatblocks offers a simple view to edit
your flatblocks from your frontend. To use it simply include it in your
URLconf and create a ``flatblocks/edit.html`` template.

By default the view doesn't do any permission checking, so you should decorate
it accordingly in your URLconf::
    
    from flatblocks.views import edit
    from django.contrib.auth.decorators import login_required

    # ...

    urlpatterns = pattern('',
        url(r'^flatblocks/(?P<pk>\d+)/edit/$', login_required(edit),
            name='flatblocks-edit'),
        # ...
        )

The template can operate on following variables:

* ``form``
* ``flatblock``
* ``origin`` (the URL of the previous page)

Additionally the view offers some basic customization hooks via these keyword
arguments:

``template_name``
    Name of the template to be used for rendering this view. By default
    ``flatblocks/edit.html`` is used.

``success_url``
    After successfully editing a flatblock the view will redirect the user to
    the URL specified here. By default the view will try to determine the last
    visited page before entering the edit-view (which is normally a page where
    the flatblock is used) and redirect the user back there.

``modelform_class``
    If you want to use a customized ModelForm class for flatblocks you can
    specify it here.

``permission_check``
    This argument lets you specify a callback function to do some
    flatblock-specific permission checking. Such a function could look like
    this::
        
        def my_permcheck(request, flatblock):
            if request.user.is_staff or flatblock.slug == 'free_for_all':
                return True
            return HttpResponseRedirect('/')
    
    With this permission callback set, a user that is not a staff-user is not
    allowed to edit this view unless it's the "free_for_all" block. If these
    criteria are not met, the user is redirected to the root URL of the page. 

    The contract here is pretty simple. The permission callback should return
    ``False``, if the user should receive a 403 message when trying to edit
    this link. If the function returns an instance of ``HttpResponse`` the
    view will proceed from the assumption that your view already did
    everything there is to do and return that response-object. Any other
    return value tells the view that the permissions are OK for the current
    user and that it should proceed.


History
------------

Since this application targets use-cases that are basically applicable to 
most web-projects out there, there are tons of solutions similar to this one.
In fact, this app is a fork originally from `django-chunks`_ developed by 
Clint Ecker.

In November 2008 Kevin Fricovsky created the `original fork`_ in order to add
an additional "active"-flag to each chunk. That project was later on `forked 
by Peter Baumgardner`_ who removed that flag again and added a "header"-field 
in order to directly associate and optional title with each text block.

This fork aims now to add more features like variable chunks and also
integrate some of the features developed by H. Waara and S. Cranford in
the `django-better-chunks`_ fork (``django.contrib.site``- and i18n-support).

Releases
--------

0.8:
    * Python 3 & Django 1.6 support

0.7:
    * Support for evaluated blocks offering access to context variables

0.6:
    * South support
    * Installation and upgrade instructions

    Note: This is primarily a transitional release to get South in here and
    open this project up for some database changes in the future.

0.5.1
    * Removed rendering of the content attribute from the admin list by Michael Fladischer
    * PyBabel compatibility by Michael Fladischer
    * Fixed caching issue with memcache backend

0.5
    * Hungarian translation by Török Gábor
    * Method added to demo edit form (#5) by Bill Evans

0.4
    * FlatBlock autocreation by Mikhail Korobov (can be enabled/disabled
      with FLATBLOCKS\_AUTOCREATE\_STATIC\_BLOCKS setting)
    * Various fixes by Mikhail Korobov
    * Fix by Henrik Heimbuerger for the manifest

0.3.5
    * Russian translation by Mikhail Korobov

0.3.4
    * Norwegian translation by Eivind Uggedal

0.3.3
    * FlatBlock.save should also accept optional kwargs.

0.3.2
    * All settings are now in the flatblocks.settings module

0.3.1
    * Fixes a bug with FlatBlock.save() failing to reset the cache
    * Buildout integration for easier testing
    * Example urls.py and flatblocks/edit.html-template

0.3
    * createflatblock and deleteflatblock commands
    * On saving a flatblock its cache will be cleared
    * As last argument of the template tag you can now also specify a template
      name.
0.2
    * Translatable
    * ``flatblocks.views.edit`` view for editing flatblocks
0.1
    Initial release

.. _`original fork`: http://github.com/howiworkdaily/django-flatblock/
.. _`django-chunks`: http://code.google.com/p/django-chunks/
.. _`django-better-chunks`: http://bitbucket.org/hakanw/django-better-chunks/
.. _`forked by Peter Baumgardner`: http://github.com/lincolnloop/django-flatblock/
.. _`south`: http://south.aeracode.org/

===============
Wagtail TinyMCE
===============

Wagtail TinyMCE offers integration of the
`TinyMCE rich text editor <http://www.tinymce.com>`_ into the
`Wagtail CMS <http://wagtail.io>`_.

As of Wagtail 1.5 this integrates using Wagtail's alternative rich text editor feature and requires no extra customisation or patching.

Caltech Fork
============

In August 2022, we wanted a WYSIWYG editor for tables in the Caltech catalog. Specifically, we
needed something that would let us do row-spans and column-spans to make the table headers that were
in use in the catalog when it was built in InDesign. This `issue on the original repo
<https://github.com/isotoma/wagtailtinymce/issues/28>`__ pointed us to a `fork that was working as of Wagtail 2
<https://github.com/Junatum/wagtailtinymce/tree/custom-1.2>`__.
Our fork updates this project to work with Wagtail 4.x to Wagtail 6.x.

This package is using TinyMCE 4.7.13 and documents can be found at their
`repository <https://github.com/tinymce/tinymce-docs-4x>`__.

.. important::

    To support Wagtail 7 in the future, we will need to remove the usage
    of ``WidgetWithScript`` for ``wagtailtinymce.rich_text.TinyMCERichTextArea``
    class and add a custom Stimulus controller.

Installation
============

Add ``wagtailtinymce`` to your |INSTALLED_APPS Django setting|_.

.. |INSTALLED_APPS Django setting| replace:: ``INSTALLED_APPS`` Django setting
.. _`INSTALLED_APPS Django setting`: https://docs.djangoproject.com/en/1.9/ref/settings/#installed-apps

Add or change ``WAGTAILADMIN_RICH_TEXT_EDITORS`` in your settings, to use the widget ``wagtailtinymce.rich_text.TinyMCERichTextArea``.

For example, to use TinyMCE for all ``RichTextField`` and ``RichTextBlock`` instances:

.. code-block:: python

    WAGTAILADMIN_RICH_TEXT_EDITORS = {
        'default': {
            'WIDGET': 'wagtailtinymce.rich_text.TinyMCERichTextArea'
        },
    }

Or, to use TinyMCE for certain instances...

.. code-block:: python

    WAGTAILADMIN_RICH_TEXT_EDITORS = {
        'default': {
            'WIDGET': 'wagtail.wagtailadmin.rich_text.HalloRichTextArea'
        },
        'tinymce': {
            'WIDGET': 'wagtailtinymce.rich_text.TinyMCERichTextArea'
        },
    }

...and declare fields with the corresponding key in the ``editor`` parameter:

.. code-block:: python

    html_field = RichTextField(editor='tinymce', ...)
    stream_field = StreamField([('html', RichTextBlock(editor='tinymce', ...)), ...])

TinyMCE configuration
=====================

The ``TinyMCERichTextArea`` constructor accepts keyword arguments for buttons, menus and options which are merged with defaults and passed to TinyMCE.

However, Wagtail does not currently allow for passing parameters to this constructor. To change the configuration you must create and register a subclass of ``TinyMCERichTextArea`` and pass these parameters or amend defaults in the subclass constructor.

Buttons
-------

These are configured as a list of menu bars, each containing a list of groups, each containing a list of button names.

By default, TinyMCE is loaded with buttons for undo/redo, a styles dropdown, bold/italic, lists and tables, link/unlink, Wagtail documents/images/embeds, paste filter toggle and edit fullscreen.

Menu
----

These are configured as a list of menu names.

By default, TinyMCE is loaded with no menubar.

Options
-------

This is a dict. By default, TinyMCE is loaded with the following options set:

- ``browser_spellcheck``
- ``noneditable_leave_contenteditable`` (required for Wagtail image/embed handling)
- ``language`` (taken from Django settings)
- ``language_load``

TinyMCE Init Passthru
---------------------

To pass any additional keys to the ``init()`` function of TinyMCE (`see the TinyMCE docs <https://www.tinymce.com/docs/configure/>`_),
set the ``passthru_init_keys`` keyword arg to a dictionary of options.

TinyMCE plugins and tools
=========================

TinyMCE is loaded with the following plugins:

- ``hr``
- ``code``
- ``fullscreen``
- ``noneditable`` (required for Wagtail image/embed handling)
- ``paste``
- ``table`` (and ``inserttable`` tool)

To add further plugins and tools to TinyMCE, use the
``insert_tinymce_js`` and ``insert_tinymce_css`` hooks. Once you have the hook in place use the
following JavaScript to register the plugin with TinyMCE:

.. code-block:: javascript

    registerMCEPlugin(name, path, language);

For example:

.. code-block:: javascript

    registerMCEPlugin('myplugin', '/static/js/my-tinymce-plugin.js', 'en_GB');

The ``language`` parameter is optional and can be omitted.

A complete ``wagtail_hooks.py`` file example:

.. code-block:: python

    import json

    from django.templatetags.static import static
    from django.utils import translation
    from django.utils.html import format_html
    from django.utils.safestring import mark_safe
    from wagtail.core import hooks

    @hooks.register('insert_tinymce_js')
    def my_plugin_js():
        return format_html(
            """
            <script>
                registerMCEPlugin("myplugin", {});
            </script>
            """,
            mark_safe(json.dumps(static('js/my-tinymce-plugin.js'))),
            to_js_primitive(translation.to_locale(translation.get_language())),
        )

How to upgrade TinyMCE
======================

#. Clone [TinyMCE](https://github.com/tinymce/tinymce) repo in a different folder.
#. Follow the instructions on this repo to build it using Grunt.
#. As of version 4.6.4, a `/js` folder will be generated in the root of the TinyMCE repo.
#. Copy its contents to `wagtailtinymce/wagtailtinymce/static/wagtailtinymce/js/vendor`, replacing the contents of this folder.
#. Create a new branch with the versioning instructions below so that this new version is accessible with `pip`. (The branch for the version `4.6.4` is `TinyMCE4.6.4`.)

Versioning
==========
The version number of this package is the TinyMCE version, followed by
the release number of this package for that TinyMCE version.

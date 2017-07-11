---
title: Plugin Systems
tags:
  - code
  - python
  - plugins
---

*"What are plugins?" and other proceedings of the inaugural PyCon 2017
 Comparative Plugin Systems [BoF][bof]*

Within the programming world, and the Python ecosystem in particular,
there are a lot of presumptions around plugins. Specifically, we take
them for granted. "It's *just* a plugin." "Oh, *another* plugin library?"

This past PyCon, I resolved to dismiss this dismissals by revisiting
plugins, and it may have been the best programming decision I've made
all year.

<img align=right width="40%" src="/uploads/illo/snake_plugin_sm.png">

[TOC]

# Why plugins?

For all types of software, open-source or otherwise, the scalability
of development poses a problem long before scalability of performance
and other technical challenges. Engaging more developers creates code
contention and bugs. Too many cooks is all it takes to spoil the
broth.

> **All growing projects need an API for code integration.**

Call them plugins, modules, or extensions, from your browser to your
kernel, they are *the* widely successful solution. Tellingly, the only
thing wider than the success of plugin-based architecture is the
variety of implementations.

Python's dynamic nature in particular seems to encourage
inventiveness. The more the merrier, usually, but at some point we
cloud a tricky space. How different could these plugin systems be? How
wide is the range of functionalities, really? How does a developer
choose the right plugin system for a given project? For that matter,
what is a plugin system anyway? No one I talked to had clear answers.

So when [PyCon 2017][pycon_2017] rolled around, I knew exactly what I
wanted to do: call together a team of developers to get to the bottom
of the above, or at the very least, answer the question,

> *"What happens when you ask a dozen veteran Python programmers
> to spill their guts about plugins?"*

<img width="100%" src="/uploads/pycon_2017_plugin_bof_crop.jpg">

[bof]: https://en.wikipedia.org/wiki/Birds_of_a_feather_(computing)
[pycon_2017]: https://us.pycon.org/2017/about/

# Setting examples

Our group leapt into action by listing off plugin systems as fast as we could:

* [stevedore](https://docs.openstack.org/stevedore/latest/)
* [twisted.plugin](http://twistedmatrix.com/documents/current/core/howto/plugin.html)
* [Mercurial extensions](https://www.mercurial-scm.org/wiki/WritingExtensions)
* [pytest plugins](https://docs.pytest.org/en/latest/plugins.html)
* [gather](http://gather.readthedocs.io/en/latest/)
* [venusian](https://docs.pylonsproject.org/projects/venusian/en/latest/)
* [pluginbase](http://pluginbase.pocoo.org/)
* [straight.plugin](https://straightplugin.readthedocs.io/en/latest/)
* [pylint plugins](https://docs.pylint.org/en/1.6.0/plugins.html)
* [flake8 plugins](http://flake8.pycqa.org/en/latest/plugin-development/)
* [raw setuptools entrypoints](http://setuptools.readthedocs.io/en/latest/setuptools.html#dynamic-discovery-of-services-and-plugins)
* [zope.component](https://docs.zope.org/zope.component/)
* [Django command extensions](https://django-extensions.readthedocs.io/en/latest/)
* [SQLAlchemy dialects/DBAPIs](http://docs.sqlalchemy.org/en/latest/dialects/index.html)
* [Sphinx extensions](http://www.sphinx-doc.org/en/stable/extdev/index.html#dev-extensions)
* [Buildout extensions](http://docs.buildout.org/en/latest/topics/extensions.html)
* [Pike](http://pyarmory-pike.readthedocs.io/en/latest/)
* Others that came and went a little too fast to jot down

With our plate heaping with examples like these, we all felt ready to
dig into our big questions.

# Taxonomizing

For our first bit of analysis, we asked: What practical and
fundamental attributes differentiate these approaches? If we had to
create a taxonomy, what characteristics would we look for?

## Discovery

A plugin system's first job is locating plugins to load. The split here
is whether plugins are individually specified, or automatically
discovered based on paths and patterns.

In either case, we need paths. Some systems provide search
functionality, exchanging explicitness for convenience. This can be a
good trade, especially when plugins number in the double digits, or
whenever less technical users are concerned.

## Generalizability

You'll notice our list of example plugin systems included several very
specialized examples, from pylint to SQLAlchemy. Many projects even
use totally internal plugin systems to achieve better
factoring. Bespoke plugin systems like pylint's are a valuable
reference for anyone looking to account for patterns in their own system,
especially generic systems like pike and gather.

## Install location

Our next key differentiator was the degree to which the plugin system
leveraged Python's own package management facilities. Some systems,
like venussian, were designed to encourage `pip install`-ing plugins,
searching for them in `site-packages`, alongside the application
itself.

Other systems have their own search paths, locating plugins in the
user directory and elsewhere on the filesystem. Still other systems
are designed for plugins inside the application tree, as is the case
with [Django apps][django_apps].

[django_apps]: https://docs.djangoproject.com/en/1.11/ref/applications/

## Plugin independence

One of the most challenging parts of plugin development is finding
ways of independently reusing and testing code, while keeping in mind
its role as an optional component of another application.

In some systems, like Django's, the tailoring is so tightly coupled
that reusability doesn't make sense. But other approaches, like
[gather][gather]'s, keeps plugin code independently reusable.

[gather]: http://gather.readthedocs.io/en/latest/
[conda]: https://github.com/conda/conda
[conda_build]: https://github.com/conda/conda-build

## Dependency registration

Almost all plugins work by providing some set of *hooks* which are
findable and callable by the core. We found another differentiator in
whether and how plugins could gain access to resources from the core,
and even other plugins.

Not all systems support this, preferring to keep plugins as leaf
participants in the application. Those simplistic setups hit limits
fast. The next best, and most common, solution is to simply pass the
whole core state at the time of hook invocation, providing plugins
with the same access as the core. It works, but the API becomes the
whole system state.

More advanced systems allow plugins to publish an inventory of
dependencies, which the core then injects. Higher granularity enables
lazier evaluation for a performance boost, and more explicit structure
helps create a more maintainable application overall.

# Drawing a line

Feeling like we were getting closer to the nature of things, we
reversed direction, asking instead: What *isn't* a plugin system?

Establishing explicit boundaries and specific counterexamples proved
instrumental to producing a final definition.

Is [`eval()`][eval] a plugin system? We thought maybe, at first. But
the more we thought about it, no, because the code itself was not
sufficiently abstracted through a loading or namespacing system.

[eval]: https://docs.python.org/2/library/functions.html#eval

Is [DNS][dns] a plugin system? It has names and namespaces galore. But
no, because code is not being loaded *in*. Remote services in general
are beyond the boundary of what a plugin can be. They exist out there,
and we call out to them. They're plugins, not callouts.

[dns]: https://en.wikipedia.org/wiki/Domain_Name_System

# A definition

So with our boundaries established, we were ready to offer a
definition:

> *A plugin system is a software facility used by a running program to
> discover and load code, often containing hooks called by the host
> application*

But, by this definition, isn't Python's built-in `import`
functionality a plugin system? Mostly, yes! Python's import system is
a plugin system.

* For discovery it uses [`sys.path`][sys_path], various "site"
  directories and files, and [much more][sys_path_hooks].
* For installation, it uses `site-packages`,
  [user `.local` directories][user_installs], and more.
* As far as independent reusability, virtually every module
  [can be made its own entrypoint][dash_m].
* As for dependency registration, every module is tossed into
  [`sys.modules`][sys_modules] with the others, but also has access to
  `import` and `sys`, making roughly every module an equal partner in
  application state.

[dash_m]: https://docs.python.org/2/using/cmdline.html#cmdoption-m
[sys_path]: https://docs.python.org/2/library/sys.html#sys.path
[sys_modules]: https://docs.python.org/2/library/sys.html#sys.modules
[sys_path_hooks]: https://docs.python.org/2/library/sys.html#sys.path_hooks
[user_installs]: https://pip.readthedocs.io/en/latest/user_guide/#user-installs

Python's import system is a powerful one, with a
[plugin system][import_hooks] of its own. But finders, loaders, and
import hooks aren't *Python's* plugin system. For that, you need to
look to [the `site` module][site_mod].

[import_hooks]: https://docs.python.org/3/reference/import.html#finders-and-loaders
[site_mod]: https://docs.python.org/2/library/site.html

# Motivation

And yet, with our hour nearly up, all these proximate details still
hadn't distilled into an ultimate motivation behind plugins. To
satisfy our inquiry, we return to one of software engineering's
fundamental principles: [Separation of concerns][soc].

[soc]: https://en.wikipedia.org/wiki/Separation_of_concerns

We want to reason about our software. We want to know what state it is
in. What we all want is the ability to say, "the core is ready,
proceeding to load modules/extensions/plugins." We want to defer
loading _some_ code so that we can add extra instrumentation, checks,
resiliency, and error messages to that loading process. If something
misbehaves, we can do better than a stack trace and an `ImportError`.

And while Python's import system is a plugin system of sorts, because
we use it all the time, we've already used up most of the concern
separation potential of `import`. Hence, all the creativity around
plugin systems, seeking a balance between feeling native to Python,
while not still successfully separating concerns.

# In conclusion

So now we have achieved a complete view of the Python plugin system
ecosystem, from motivation to manifestation.

By numbers alone, it may seem on the face like there are more than
enough Python plugin solutions. But looking at the basic taxonomy
above, it's clear that there are several gaps still waiting to be
filled.

By taking a holistic look at the implementations and motivations, the
PyCon 2017 Plugins Open Session ended with the conclusion that even
this wide selection could use expansion.

So go forth and build, and continue to build! The future of
well-factored code depends on it.[^further]

<img width="50%" src="/uploads/illo/snake_puzzle_sm.png">

[^further]: For additional reading, I recommend doing what we did
            after our discussion, finding and reading
            [this post from Eli Bendersky][bender_post]. While it
            focuses more on specific implementations and less about
            generalized systems, Eli's post overlaps in many very
            reaffirming ways, much to our relief and
            gratifications. The worked example of building
            ReStructured Text plugins is a perfect complement to the
            post above.

[bender_post]: http://eli.thegreenplace.net/2012/08/07/fundamental-concepts-of-plugin-infrastructures


<!-- Resiliency? Whether or not loading a failed plugin load was fatal
to the rest of the application -->
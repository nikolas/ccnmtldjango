ccnmtldjango is a Paste Template that we use to replace
the standard `django-admin.py startproject` command.

It does the same basic task of setting up a directory
structure for a django app, but it has been extended to
implement a lot of CCNMTL specific functionality
and configuration.

What it provides for us that startproject doesn't:

* Django Wind (a Django bridge to Columbia's central auth service -
  http://www.jasig.org/cas/deployments/columbia-university)
  is included and configured by default so our apps can use WIND
  for auth automatically. Ie, anyone with a Columbia UNI by
  default will have an account. The CCNMTL developer UNIs are
  automatically set up as superusers, and the group affil that
  CCNMTL staff all have gets automatically mapped to staff. These
  are generally useful defaults for us.
* virtualenv and pip setup with source tarballs bundled and
  bootstrappable, `manage.py`'s shebang set to use it. This
  basically fits it into our one-step automated deployment and
  containment approach.
* use wheel packages wherever possible
* sorl.thumbnail (a handy dandy image thumbnailing library) is included by default
* flatpages enabled
* settings split for dev/prod/staging
* apache/django.wsgi configured
* sample apache config for mod_wsgi setup using Virtual
  Environments (everything is streamlined so that we can just
  symlink the generated apache config file into our production
  server's `/etc/apache/sites-enabled/` directory and it's good to go)
* media dirs for dev/prod/staging configured
* `django-typogrify` included (http://code.google.com/p/typogrify/)
  along with `smartypants.py` (which it uses)
* `raven` included
  (https://github.com/dcramer/raven/) and configured for our sentry
  setup
* raven configured to not run on south migrations
* `django-munin` included (https://github.com/ccnmtl/django-munin)
* `South` is included for database migrations
* `django-annoying` is included (I like `@render_to`)
* `django-nose` installed and set up as test runner (much nicer!)
* coverage reports configured for `django-nose`
* sqlite in-memory database used for unit tests
* south tests automatically skipped on `./manage.py test` (they break)
* `django_compressor` added and set up to compress css on production
* `django_statsd` for graphite integration
* `/stats/` page wired up to display basic traffic stats for the app
* 'main' app with templated index view wired up and ready to go
* `uuid.py`
* jquery (1.7.2, minified) included
* base templates included
* django admin enabled (and authenticated with WIND for tlc)
* `restclient`
* `httplib2`
* `requests`
* `imageuploader`
* markdown is included and enabled
* database defaulted to postgresql
* transaction middleware enabled by default (cause data corruption is teh suck)
* timezone set
* I18n turned off (we are unfortunately monolingual. no sense in denying it)
* PIL
* `psycopg2` (stripped of its `mx.DateTime` dependency)
* a nice default template design with alternate base templates for multi-column layout.
* `flake8` (http://pypi.python.org/pypi/flake8) is installed by default
  for code linting
* lettuce/selenium included for nice browser tests, including sample
  index feature and helpful terrain.py functions
* backbone.js
* underscore.js
* layout based on twitter bootstrap
* `django-waffle` included for feature flipping (https://github.com/jsocol/django-waffle)
* `django-jenkins` included and set up for our Jenkins instance (https://github.com/kmmbvnr/django-jenkins)
* `django-smoketest` included, wired up, and a sample `smoke.py`
  included (https://github.com/ccnmtl/django-smoketest)
* `django-extensions` included to do a variety of things like use
  IPython, Werkzeug debugger, kcachegrind profiling, etc. (https://github.com/django-extensions/django-extensions)
* `django-impersonate` included and configured for easier debugging

To use ccnmtldjango, you need python 2.7+, virtualenv, pip, and a recent
setuptools installed on your machine.

First, if you don't already have ccnmtldjango installed, do

    $ pip install ccnmtldjango

It should automatically pull in the needed dependencies (just
PasteScript, actually). If that gives you problems, the most likely
cause is that one or more of your setuptools, pip, or virtualenv
libraries is old. So upgrade those first and try again.

Running

    $ paster create --list-templates

should include ccnmtldjango

Now, to quickstart a django project, do

    $ paster create --template=ccnmtldjango myprojectname

`myprojectname` should be a python module name (ie, lowercase,
no punctuation, etc). It will create a directory called
`myprojectname` that has a django project in it.

paster still doesn't do anything with file permissions, so we still
need to manually set a couple:

    $ cd myprojectname
    $ chmod 755 manage.py bootstrap.py build_lettuce_db.sh

I couldn't figure out a way to insert random strings into the code via
Paste Template, so one thing that ccnmtldjango is missing compared to
a regular django startproject is that the `SECRET_KEY` variable in
settings_shared.py is always the same default. That's clearly not a
good idea, so make sure you change that to some other random string
that will be unique to your project. (Ideally, put that in a
`local_settings.py` that doesn't get checked into version control).

This is probably a good point to check the project into version control.

We use containment for django too, with virtualenv:

    $ ./bootstrap.py

That will create a `ve` directory which contains a virtualenv and has
had all the libraries specified in the `requirements.txt` file
installed into it (this includes django itself). The `ve` directory
should never be checked into svn since it's generated. If you need
other libraries for your application, `requirements.txt` then re-run
`./bootstrap.py`.

Keep in mind that with virtualenv, there's no need to `activate` an
environment. Instead, a ve has a `bin` directory which contains a
python executable. If you use that instead of the system python
executable, it uses the libraries in that virtualenv.

ccnmtldjango assumes that your project will use a postgresql database
with the same name as your project. So, for our example, you would
then do:

    $ createdb myprojectname

and it is all set to use it:

    $ ./manage.py syncdb
    $ ./manage.py migrate

will install the tables that django needs for it's common apps (sites,
sessions, admin, flatpages, etc) and have you create an admin user.

The `./manage.py syncdb` automagically sets up an "example.com"
site. This should be changed to your site domain (e.g. `localhost:8000`)
via the admin console. `http://localhost:8000/admin/sites/site/`. (if it
matters for your application)

Tests should pass out of the box:

    $ ./manage.py test

They can be run via Jenkins as well and generate the right reports in
a `reports` directory (which you will want to gitignore).

    $ ./manage.py jenkins

Your application is ready to run now:

    $ ./manage.py runserver

will start a server on `http://localhost:8000/`. The admin
app should be accessible (via the user account you created during
syncdb, or via WIND to tlc users (or ones specified in the
`WIND_SUPERUSER_MAPPER_GROUPS` list in `settings_shared.py`). So go ahead
and login to `http://localhost:8000/admin/`

Even without any application specific code, flatpages is included so
you can put content on the web right away.

Lettuce and Selenium have also been set up, so (as long as you have
firefox installed in a regular way), you should be able to do:

    $ LETTUCE_SKIP_SELENIUM=1 ./manage.py harvest --settings=yourapp.settings_lettuce

And see a couple basic lettuce tests (defined in
`main/features/index.feature`) run by and pass.

If you have your browser stuff setup properly (running on your local
machine, etc), you can run that without `LETTUCE_SKIP_SELENIUM` and
run full browser tests.

From this point out, it's basic django development. You'll probably
want to do a `./manage.py startapp` to create your own application
within the project and so on.

--------------------------
Setting up a fresh checkout

The first time you check out an existing ccnmtl-template project from
svn/git:

     $ ./bootstrap.py
     $ ./manage.py runserver <IP Address>:<PORT>


------------------------------------------
Differences from a standard Django install

Obviously, a bunch of libraries and such have been added and there's
the whole virtualenv thing. There are also some differences from a
standard django project (ie, the result of `django-admin.py startproject`) that you should be awayre of.

First, the settings have been split up to make dev/staging/prod
deployments easier to configure. A regular django install will have
one `settings.py` file that contains all the settings. Django
developers will usually copy that settings file and make changes when
deploying to production. ccnmtldjango takes advantage of the fact that
settings are just python code and can be imported and overridden. So
we have a `settings_shared.py` which contains most of the
settings. `settings.py` (which should be used for development) and
`settings_production.py` then import everything from from
`settings_shared.py`. `settings_production.py` then also overrides any
settings that should be different in the production deployment
(usually paths to templates and media files). Same deal for `settings_staging.py`

`TransactionMiddleware` is enabled by default. This means that each HTTP
request gets a transaction that commits or rolls back at the end of
the request. The default django setup for some reason does things
"autocommit" style where each database operation runs in its own
transaction, independent of the HTTP request.

The other big difference to be aware of is the top-level `templates`
directory. Standard django procedure is to have a templates directory
in each application in your project that contains the templates for
that application. ccnmtldjango has the top-level templates directory
for a couple reasons. First, since paster only creates the project
level directory and not the application directories, it was the only
way to have it include a default `base.html`, `admin/login.html`,
`registration/login.html` and so on. I also just like the approach of
having a project-level templates directory, especially for the
`base.html` template. Django allows multiple template directories and
searches through them in a predictable order, so you can (and probably
should) still create application level template directories, list them
in `TEMPLATE_DIRS` ahead of the project level one, and override whatever
templates you want in those.

I18N is turned off since it's fairly rare that we do multi-lingual
stuff and it's a performance hit to have it enabled if it's not being
used. If you need to do a multi-lingual django site, just re-enable it
and get to work.

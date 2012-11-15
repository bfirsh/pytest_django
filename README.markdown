pytest_django
=============

**Note:** This version is unmaintained. You are probably looking for [Andreas Pelme's fork](https://github.com/pelme/pytest_django).

pytest_django is a plugin for [py.test](http://pytest.org/) that provides a set of useful tools for testing [Django](http://www.djangoproject.com/) applications.

Requires:

  * Django 1.1
  * py.test 1.0.0

Installation
------------

    $ python setup.py install

Then simply create a `conftest.py` file in the root of your Django project 
containing:

    pytest_plugins = ['django']

Usage
-----

Run py.test in the root directory of your Django project:

    $ py.test

This will attempt to import the Django settings and run any tests.

Note that the default py.test collector is used, as well as any file within a 
tests directory. As such, so it will not honour `INSTALLED_APPS`. You must use 
`collect_ignore` in a `conftest.py` file to exclude any tests you don't want 
to be run.

See [py.test's documentation](http://pytest.org/) for more information, 
including usage of the `-k` option for selecting specific tests.

A `--settings` option is provided for explicitly setting a settings module, 
similar to `manage.py`.

The `--copy_live_db` switch will copy your live database to run the tests 
against, rather than creating a new one from scratch each time. This is useful 
if you're using a database migration system such as South, and have a large 
number of migrations defined, which can be very slow to run.

There is also a `--database` option which works in conjunction with 
`--copy_live_db` to specify a particular database to copy, to avoid problems 
caused by tests interacting with data that's in your database already but which
isn't created via migrations or test setup code. With this option, you can 
have a ready-migrated database which is used just for testing, while you 
continue to develop on your main database.

pytest_django makes py.test's built in unittest support fully backwards 
compatible with Django's unittest test cases. If they are failing, this is a 
bug.

Hooks
-----

The session start/finish and setup/teardown hooks act like Django's `test` 
management command and unittest test cases. This includes creating the test 
database and maintaining a constant test environment, amongst other things.

Funcargs
--------

### `client`

A Django test client instance.

Example:

    def test_something(client):
        assert 'Success!' in client.get('/path/')
        

### `rf`

An instance of Simon Willison's excellent 
[RequestFactory](http://www.djangosnippets.org/snippets/963/).

### `settings`

A Django settings object that restores itself after the tests have run, making
it safe to modify for testing purposes.

Example:
    
    def test_middleware(settings, client):
        settings.MIDDLEWARE_CLASSES = ('app.middleware.SomeMiddleware',)
        assert 'Middleware works!' in client.get('/')

Decorators
----------

### `@py.test.params`

A decorator to make parametrised tests easy. Takes a list of dictionaries of 
keyword arguments for the function. A test is created for each dictionary.

Example:

    @py.test.params([dict(a=1, b=2), dict(a=3, b=3), dict(a=5, b=4)])  
    def test_equals(a, b):
        assert a == b

### `@py.test.urls`

Provides the ability to change the URLconf for this test, similar to the 
`urls` attribute on Django's `TestCase`.

Example:
    
    @py.test.urls('myapp.test_urls')
    def test_something(client):
        assert 'Success!' in client.get('/some_path/')

Fixtures
--------

Fixtures can be loaded with `py.test.load_fixture(name)`. For example:

    def pytest_funcarg__articles(request):
        py.test.load_fixture('test_articles')
        return Article.objects.all()


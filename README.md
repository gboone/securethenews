# Secure the News

[![Build Status](https://travis-ci.org/freedomofpress/securethenews.svg?branch=master)](https://travis-ci.org/freedomofpress/securethenews)

## Getting Started with the Development Environment

The installation instructions below assume you have the following software on your machine:

* [virtualenv](http://www.virtualenv.org/en/latest/virtualenv.html#installation)
* [docker](https://docs.docker.com/engine/installation/)

Create a Python2 virtualenv and install `molecule/requirements.txt`/. From the
checkout directory, run:

```bash
virtualenv env/
source env/bin/activate
pip install -r molecule/requirements.txt
```

Then run:

```bash
make dev-go
```

If this command completes successfully, your development site will be available
at: http://localhost:8000

To import the example data, run:

```bash
make dev-createdevdata
```

This will also create an admin user for the web interface at
http://localhost:8000/admin/ (username: test, password: test).

If you want to start the TLS scan for all the news sites in your development
environment, run:

```bash
make dev-scan
```

For a full list of all helper commands in the Makefile, run `make help`. And,
of course, you can obtain a shell directly into any of the containers, e.g.:

```bash
docker exec -it stn_django bash
```

### Updating Python dependencies

New requirements should be added to ``*requirements.in`` files, for use with ``pip-compile``.
There are two Python requirements files:

* ``requirements.in`` production application dependencies
* ``molecule/requirements.in`` local testing and CI requirements (e.g. molecule, safety)

Add the desired dependency to the appropriate ``.in`` file, then run:

.. code:: bash

    make update-pip-dependencies

All requirements files will be regenerated based on compatible versions. Multiple ``.in``
files can be merged into a single ``.txt`` file, for use with ``pip``. The Makefile
target handles the merging of multiple files.

### Development Fixtures

The `createdevdata` management commands loads Site and Scan data from the
fixtures in `sites/fixtures/dev.json`. If you change the schema of `sites.Site`
or `sites.Scan`, you will need to update these fixtures, **or future
invocations of `createdevdata` will fail.**

The general process for updating the development fixtures is:

1. Migrate your database to the last migration where the fixtures were updated.
2. Load the fixtures.
3. Run the migrations that you've added.
4. Export the migrated fixtures:

    ```
    $ python3 manage.py dumpdata sites.{Site,Scan} > sites/fixtures/dev.json
    ```

The test suite includes a smoke test for `createdevdata`, so you can easily
verify that the command is working without disrupting your own development
environment.

### Live reload

The default gulp `watch` task uses `gulp-livereload` to automatically trigger a
browser refresh when changes to the frontend code are detected. In order to take
advantage of this, you will need to install the [LiveReload Chrome
extension](https://chrome.google.com/webstore/detail/livereload/jnihajbhpnppcggbcgedagnkighmdlei?hl=en).

Once you've installed the extension, simply load the development site in your
browser (`localhost:8000`) and click the LiveReload extension icon to initiate
live reloading.

## API

If everything is working correctly, you should be able to find an API endpoint
at `localhost:8000/api` (it will redirect to the current API version).

The API is read-only and can be used to obtain site metadata and the latest scan
for a given site (e.g., `/api/v1/sites` will return a directory, and
`/api/v1/sites/bbc.co.uk` will return details about the BBC). Various filters
and sort options are supported; click the "filters" dialog in the UI to explore
them.

To get all scans for a given site, you can use a path like
`/api/v1/sites/bbc.co.uk/scans`. This URL can also be found in the all_scans
field for a given site result.

If you run a public site, note that read access to the API is available to any
origin via CORS.

The API is implemented using the Django REST framework; documentation for it can
be found here:

http://www.django-rest-framework.org/

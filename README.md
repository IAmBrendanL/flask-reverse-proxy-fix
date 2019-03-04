# Flask Reverse Proxy Middleware

Python Flask middleware for applications running under a reverse proxy.

## Purpose

Currently this middleware supports correcting URLs generated by `Flask.url_for()` where a common prefix needs to be 
added to all URLs.

For example: If client requests for an application are reverse proxied such that: `example.com/some-service/v1/foo` 
becomes `some-service-v1.internal/foo`, where `/foo` is a route within a Flask application `foo()`.

Without this middleware, a call to `Flask.url_for('.foo')` would give: `/foo`. If returned to the client, as a *self* 
link for example, this would cause a request to `example.com/foo`, which would be invalid as the `/some-service/v1` 
prefix is missing.

With this middleware, a call to `Flask.url_for('.foo')` would give: '/some-service/v1/foo', which will work if used by 
a client.

This middleware is compatible with both relative and absolute URLs (i.e. `Flask.url_for('.foo')` and 
`Flask.url_for('.foo', _external=True)`.

This middleware incorporates the 
[`werkzeug.contrib.fixers.ProxyFix`](http://werkzeug.pocoo.org/docs/0.14/contrib/fixers/#werkzeug.contrib.fixers.ProxyFix) 
and based on the 
[Fixing SCRIPT_NAME/url_scheme when behind reverse proxy](http://flask.pocoo.org/snippets/35/) Flask snippet.

## Installation

This package can be installed using Pip from [PyPi](https://pypi.org/project/flask-reverse-proxy-fix):

```
$ pip install flask-reverse-proxy-fix
```

## Usage

This middleware requires one parameter, a Flask config option, `REVERSE_PROXY_PATH_PREFIX`, for the path prefix value.
 
**Note:** The prefix value **SHOULD** include a preceding slash, it **SHOULD NOT** include a trailing slash (i.e. use 
`/foo` not `/foo/`).

A minimal application would look like this:

```python
from flask import Flask, url_for
from flask_reverse_proxy_fix.middleware import ReverseProxyPrefixFix

app = Flask(__name__)

app.config['REVERSE_PROXY_PATH'] = '/foo'
ReverseProxyPrefixFix(app)

@app.route('/')
def hello_world():
    return url_for('.hello_world')
```

## Developing

A docker container ran through Docker Compose is used as a development environment for this project. It includes 
development only dependencies listed in `requirements.txt`, a local Flask application in `app.py` and 
[Integration tests](#integration-tests).

Ensure classes and methods are defined within the `flask_reverse_proxy_fix` package.

Ensure [Integration tests](#integration-tests) are written for any new feature, or changes to existing features.

If you have access to the BAS GitLab instance, pull the Docker image from the BAS Docker Registry:

```shell
$ docker login docker-registry.data.bas.ac.uk
$ docker-compose pull

# To run the local Flask application using the Flask development server
$ docker-compose up

# To start a shell
$ docker-compose run app ash
```

### Code Style

PEP-8 style and formatting guidelines must be used for this project, with the exception of the 80 character line limit.

[Flake8](http://flake8.pycqa.org/) is used to ensure compliance, and is ran on each commit through 
[Continuous Integration](#continuous-integration).

To check compliance locally:

```shell
$ docker-compose run app flake8 . --ignore=E501
```

### Dependencies

Development Python dependencies should be declared in `requirements.txt` to be included in the development environment.

Runtime Python dependencies should be declared in `requirements.txt` and `setup.py` to also be installed as dependencies
of this package in other applications.

All dependencies should be periodically reviewed and update as new versions are released.

```shell
$ docker-compose run app ash
$ pip install [dependency]==
# this will display a list of available versions, add the latest to `requirements.txt` and or `setup.py`
$ exit
$ docker-compose down
$ docker-compose build
```

If you have access to the BAS GitLab instance, push the Docker image to the BAS Docker Registry:

```shell
$ docker login docker-registry.data.bas.ac.uk
$ docker-compose push
```

### Dependency vulnerability scanning

To ensure the security of this API, all dependencies are checked against 
[Snyk](https://app.snyk.io/org/antarctica/project/dd93f059-5ff6-4767-b8cb-f1090fef43c4) for vulnerabilities. 

**Warning:** Snyk relies on known vulnerabilities and can't check for issues that are not in it's database. As with all 
security tools, Snyk is an aid for spotting common mistakes, not a guarantee of secure code.

Some vulnerabilities have been ignored in this project, see `.snyk` for definitions and the 
[Dependency exceptions](#dependency-vulnerability-exceptions) section for more information.

Through [Continuous Integration](#continuous-integration), on each commit current dependencies are tested and a snapshot
uploaded to Snyk. This snapshot is then monitored for vulnerabilities.

#### Dependency vulnerability exceptions

This project contains known vulnerabilities that have been ignored for a specific reason.

* [Py-Yaml `yaml.load()` function allows Arbitrary Code Execution](https://snyk.io/vuln/SNYK-PYTHON-PYYAML-42159)
    * currently no known or planned resolution
    * indirect dependency, required through the `bandit` package
    * severity is rated *high*
    * risk judged to be *low* as we don't use the Yaml module in this application
    * ignored for 1 year for re-review

### Static security scanning

To ensure the security of this API, source code is checked against [Bandit](https://github.com/PyCQA/bandit) for issues 
such as not sanitising user inputs or using weak cryptography. 

**Warning:** Bandit is a static analysis tool and can't check for issues that are only be detectable when running the 
application. As with all security tools, Bandit is an aid for spotting common mistakes, not a guarantee of secure code.

Through [Continuous Integration](#continuous-integration), each commit is tested.

To check locally:

```shell
$ docker-compose run app bandit -r .
```

## Testing

### Integration tests

This project uses integration tests to ensure features work as expected and to guard against regressions and 
vulnerabilities.

The Python [UnitTest](https://docs.python.org/3/library/unittest.html) library is used for running tests using Flask's 
test framework. Test cases are defined in files within `tests/` and are automatically loaded when using the 
`test` Flask CLI command included in the local Flask application in the development environment.

Tests are automatically ran on each commit through [Continuous Integration](#continuous-integration).

To run tests manually:

```shell
$ docker-compose run -e FLASK_ENV=testing app flask test
```

To run tests using PyCharm:

* *Run* -> *Edit Configurations*
* *Add New Configuration* -> *Python Tests* -> *Unittests*

In *Configuration* tab:

* Script path: `[absolute path to project]/tests`
* Python interpreter: *Project interpreter* (*app* service in project Docker Compose)
* Working directory: `[absolute path to project]`
* Path mappings: `[absolute path to project]=/usr/src/app`

**Note:** This configuration can be also be used to debug tests (by choosing *debug* instead of *run*).

### Continuous Integration

All commits will trigger a Continuous Integration process using GitLab's CI/CD platform, configured in `.gitlab-ci.yml`.

This process will run the application [Integration tests](#integration-tests).

Pip dependencies are also [checked and monitored for vulnerabilities](#dependency-vulnerability-scanning).

## Distribution
 
Both source and binary versions of the package are build using [SetupTools](https://setuptools.readthedocs.io), which 
can then be published to the [Python package index](https://pypi.org/project/flask-reverse-proxy-fix/) for use in other 
applications. Package settings are defined in `setup.py`.

This project is built and published to PyPi automatically through [Continuous Deployment](#continuous-deployment).

To build the source and binary artefacts for this project manually:

```shell
$ docker-compose run app ash
# build package to /build, /dist and /flask_reverse_proxy_fix.egg-info
$ python setup.py sdist bdist_wheel
$ exit
$ docker-compose down
```

To publish built artefacts for this project manually to [PyPi testing](https://test.pypi.org):

```shell
$ docker-compose run app ash
$ python -m twine upload --repository-url https://test.pypi.org/legacy/ dist/*
# project then available at: https://test.pypi.org/project/flask-reverse-proxy-fix/
$ exit
$ docker-compose down
```

To publish manually to [PyPi](https://pypi.org):

```shell
$ docker-compose run app ash
$ python -m twine upload --repository-url https://pypi.org/legacy/ dist/*
# project then available at: https://pypi.org/project/flask-reverse-proxy-fix/
$ exit
$ docker-compose down
```

### Continuous Deployment

A Continuous Deployment process using GitLab's CI/CD platform is configured in `.gitlab-ci.yml`. This will:

* build the source and binary artefacts for this project
* publish built artefacts for this project to the relevant PyPi repository

This process will deploy changes to [PyPi testing](https://test.pypi.org) on all commits to the *master* branch.

This process will deploy changes to [PyPi](https://pypi.org) on all tagged commits.

## Release procedure

### At release

1. create a `release` branch
2. ensure the version is bumped as per semver in `setup.py` 
3. close release in `CHANGELOG.md`
4. push changes, merge the `release` branch into `master` and tag with version

The project will be built and published to PyPi automatically through [Continuous Deployment](#continuous-deployment).

## Feedback

The maintainer of this project is the BAS Web & Applications Team, they can be contacted at: 
[servicedesk@bas.ac.uk](mailto:servicedesk@bas.ac.uk).

## Issue tracking

This project uses issue tracking, see the 
[Issue tracker](https://gitlab.data.bas.ac.uk/web-apps/flask-middleware/flask-reverse-proxy/issues) for more 
information.

**Note:** Read & write access to this issue tracker is restricted. Contact the project maintainer to request access.

## License

© UK Research and Innovation (UKRI), 2019, British Antarctic Survey.

You may use and re-use this software and associated documentation files free of charge in any format or medium, under 
the terms of the Open Government Licence v3.0.

You may obtain a copy of the Open Government Licence at http://www.nationalarchives.gov.uk/doc/open-government-licence/

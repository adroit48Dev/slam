=================
Command Reference
=================

slam
====

The command ``slam`` provides access to all the features of this package thorugh
subcommands. To find the list of available subcommands, use ``slam --help``, and
to find options available to a specific subcommand, use
``slam <subcommand> --help``.

.. program-output:: slam --help

Common arguments
----------------

The following command-line arguments are available to all subcommands, and when
given, must appear before the subcommand name:

- ``--config-file CONFIG_FILE`` or ``-c CONFIG_FILE``

  Specify a custom configuration file. If this option is not given, the
  configuration is loaded from file *slam.yaml* in the current directory.

slam init
=========

The ``slam init`` command creates a brand new configuration file.

.. program-output:: slam init --help

Required arguments
------------------

- ``wsgi_app``

  A reference to the project's WSGI application callable. This argument must be
  in the format ``<module>:<app>``, where ``module`` is the module or package
  name where the WSGI application callable is located, and ``app`` is the
  name of the variable that holds it.

Optional arguments
------------------

- ``--name NAME``

  The name of the project. If this argument is not given, the WSGI module is
  used as the project name.

- ``--description DESCRIPTION``

  A short project description.

- ``--bucket BUCKET``

  The name of an S3 bucket to use as storage for Lambda packages. If this
  argument is not given, the project name is used as bucket name.

- ``--timeout TIMEOUT``

  The timeout to configure on the Lambda function, in seconds. The default is
  10 seconds.

- ``--memory MEMORY``

  The amount of memory to provision for the Lambda function, in megabytes. The
  default is 128 MB.

- ``--stages STAGES``

  A comma-separated list of stage names to create as part of the deployment. If
  this argument is not provided, a single stage named ``dev`` is created.

- ``--requirements REQUIREMENTS``

  The name of the Python requirements file that contains the project
  dependencies. If this argument is not given, slam looks for a
  *requirements.txt* file in the project's root directory.

- ``--runtime RUNTIME``

  The name of the Lambda runtime to use for the function. This can be either
  ``"python2.7"`` or ``"python3.6"``. If this argument is not provided, the
  runtime is guessed from the version of python that is being used.

- ``--dynamodb-tables DYNAMODB_TABLES``

  A comma-separated list of DynamoDB table names to create for each stage. Once
  these tables are created, they will be named using the format
  ``<stage>.<table_name>``, so that each stage has a unique table name.

Examples
--------

::

    $ slam init fizzbuzz:fizzbuzz --stages dev,prod
    The configuration file for your project has been generated. Remember to add slam.yaml to source control.

    $ slam init tasks_api:app --wsgi --stages dev,staging,prod --dynamodb-tables users,tasks
    The configuration file for your project has been generated. Remember to add slam.yaml to source control.

slam build
==========

The ``slam build`` command builds a Lambda package, without deploying it.

.. program-output:: slam build --help

Required arguments
------------------

None.

Optional arguments
------------------

- ``--rebuild-deps``

  To speed up the build process, this command reuses dependencies from a
  previous build (installing any requirement changes on top). If this option
  is given, old requirements are deleted and everything is installed from
  scratch.

Example
-------

::

    $ slam build
    lambda_package.20170112_143002.zip has been built successfully.

slam deploy
===========

The ``slam deploy`` command deploys your project to a stage on AWS.

.. program-output:: slam deploy --help

Required arguments
------------------

None.

Optional arguments
------------------

- ``--rebuild-deps``

  To speed up the deployment process, this command reuses dependencies from a
  previous deploy (installing any requirement changes on top). If this option
  is given, old requirements are deleted and everything is installed from
  scratch.

- ``--no-lambda``

  Skip a deployment of a new lambda package. This can be used when a deployment
  has been updated, but the code has not. A typical example of when this is
  convenient is when the configuration file is edited to add or remove stages
  or database tables.

- ``--lambda-package LAMBDA_PACKAGE``

  Instead of building a new lambda package, use the one provided. The given
  package must be a zip file in the format required by AWS Lambda. The zip
  files produced by the ``slam build`` command can be used here.

- ``--stage STAGE``

  The stage that receives the updated Lambda function. By default this is the
  stage that is marked as the development stage in the configuration. The stage
  that receives the deployment will be updated to the latest version of the
  Lambda function as part of the deployment.

Example
-------

::

    $ slam deploy
    Building lambda package...
    Deploying simple-api...
    simple-api is deployed!
      Function name: simple-api-Function-1XARPP7W4H3KR
      Stages:
        dev:$LATEST: https://ukhhy78b6a.execute-api.us-west-2.amazonaws.com/dev
        prod:31: https://ukhhy78b6a.execute-api.us-west-2.amazonaws.com/prod
        staging:30: https://ukhhy78b6a.execute-api.us-west-2.amazonaws.com/staging

slam publish
============

The ``slam publish`` command makes a version of your project available on a
stage with a persistent version number.

.. program-output:: slam publish --help

Required arguments
------------------

- ``stage``

  The stage that receives the published version of the project.

Optional arguments
------------------

- ``--version VERSION``

  Publish a specific Lambda version. The given version can be a number, or a
  stage name. When a stage name is given, the version of the project stored in
  that stage is published.

Examples
--------

Assuming a project that has three stages named ``dev``, ``staging`` and
``prod``, new code versions in the ``dev`` stage can be published to
``staging`` with this command::

    $ slam publish staging
    Publishing simple-api:dev to staging...
    simple-api is deployed!
      Function name: simple-api-Function-1XARPP7W4H3KR
      Stages:
        dev:$LATEST: https://ukhhy78b6a.execute-api.us-west-2.amazonaws.com/dev
        prod:1: https://ukhhy78b6a.execute-api.us-west-2.amazonaws.com/prod
        staging:2: https://ukhhy78b6a.execute-api.us-west-2.amazonaws.com/staging

Later a version running on staging can be published to ``prod`` with::

    $ slam publish prod --version staging
    Publishing simple-api:staging to prod...
    simple-api is deployed!
      Function name: simple-api-Function-1XARPP7W4H3KR
      Stages:
        dev:$LATEST: https://ukhhy78b6a.execute-api.us-west-2.amazonaws.com/dev
        prod:2: https://ukhhy78b6a.execute-api.us-west-2.amazonaws.com/prod
        staging:2: https://ukhhy78b6a.execute-api.us-west-2.amazonaws.com/staging

slam status
===========

The ``slam status`` command shows the current deployment status of your
project.

.. program-output:: slam status --help

Required arguments
------------------

None.

Optional arguments
------------------

None.

Example
-------

::

    $ slam status
    simple-api is deployed!
      Function name: simple-api-Function-1XARPP7W4H3KR
      Stages:
        dev:$LATEST: https://ukhhy78b6a.execute-api.us-west-2.amazonaws.com/dev
        prod:4: https://ukhhy78b6a.execute-api.us-west-2.amazonaws.com/prod
        staging:3: https://ukhhy78b6a.execute-api.us-west-2.amazonaws.com/staging

slam invoke
===========

The ``slam invoke`` command invokes the Lambda function.

.. program-output:: slam invoke --help

Required arguments
------------------

None.

Optional arguments
------------------

- ``--stage STAGE``

  The stage on which to run the function. Defaults to the development stage.

- ``--nowait``

  Invoke the function, but don't wait for it to run.

- ``--dry-run``

  Do not invoke the function, just check that the current user is allowed to
  invoke it.

- ``args [args ...]``

  Input arguments to pass to the function. To pass a string argument, use
  ``argument=value``. To pass a non-string argument, use ``argument:=value``,
  where ``value`` is a number, boolean (``true`` or ``false``) or raw JSON
  string.

Example
-------

::

  $ slam invoke number:=15
  fizzbuzz

  $ slam invoke name=john age:=34
  OK

slam template
=============

The ``slam template`` command dumps the slam Cloudformation template to the
console.

.. program-output:: slam template --help

Required arguments
------------------

None.

Optional arguments
------------------

None.

Example
-------

::

    $ slam template
    <template output dumped to the console>

slam logs
=========

The ``slam logs`` command dumps logs to the console.

.. program-output:: slam logs --help

Required arguments
------------------

None.

Optional arguments
------------------

- ``--stage STAGE``

  The stage to dump logs for.

- ``--period PERIOD``

  How far back to start the log listing. The period can be given in weeks (1w),
  days (2d), hours (3h), minutes (4m) or seconds (5s). The default is 1 minute.

- ``--tail``

  Dump new logs as they appear.

Example
-------

::

    $ slam logs
    <log output dumped to the console>

slam delete
===========

The ``slam delete`` command completely removes a deployment from AWS.

.. program-output:: slam delete --help

Required arguments
------------------

None.

Optional arguments
------------------

- ``--no-logs``

  Do not delete the project logs.

Example
-------

::

    $ slam delete
    Deleting api...
    Deleting logs...
    Deleting files...

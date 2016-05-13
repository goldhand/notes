Setting up a new application on EB
==================================
A guide for setting up a new project with Elastic Beanstalk

1. Create your eb config files
------------------------------

You will need two add two directories to your project root with the following files:

    .ebextensions/
        01_packages.config
        02_app.config
    .elasticbeanstalk/
        config.global.yml
        default.cfg.yml

_** note that the `.ebextensions/` files can be whatever you want them to be. EB will read them in order of their names and execute the instructions in the file_

Example `01_packages.config`:

    packages:
      yum:
        postgresql94-devel: []
        libjpeg-turbo-devel: []
        memcached: []

Example `02_app.config`:

    container_commands:

      01_install_req:
        command: "/opt/python/run/venv/bin/pip install -r ./requirements/production.txt"

      02_migrate:
        command: "source /opt/python/run/venv/bin/activate && python ./manage.py migrate --noinput"

      03_buildstatic:
        command: "source /opt/python/run/venv/bin/activate && python ./manage.py collectstatic --noinput"

    services:
      sysvinit:
        memcached:
          enabled: true
          ensureRunning: true

    option_settings:
      "aws:elasticbeanstalk:application:environment":
        DJANGO_SETTINGS_MODULE: "config.settings.production"
        PYTHONPATH: "/opt/python/current/app:$PYTHONPATH"
      "aws:elasticbeanstalk:container:python":
        WSGIPath: "./config/wsgi.py"

Example `config.global.yml`:

    branch-defaults:
      master:
        environment: project_name-production
      develop:
        environment: project_name-develop
    global:
      application_name: project_name
      default_ec2_keyname: VPC1-EB
      default_platform: Python 2.7
      default_region: us-east-1
      profile: null
      sc: git

Deployments from the `master` branch will be made to the `project_name-production` environment and deployments from the `develop` branch will be made to the `project_name-develop` environment.

Example `default.cfg.yml`:

    EnvironmentConfigurationMetadata:
      Description: Configuration created from the EB CLI using "eb config save".
      DateModified: '1440049760000'
      DateCreated: '1440049760000'
    AWSConfigurationTemplateVersion: 1.1.0.0
    EnvironmentTier:
      Name: WebServer
      Type: Standard
    SolutionStack: 64bit Amazon Linux 2015.09 v2.0.6 running Python 2.7

There should be more info here but its application specific


2. Create your S3 bucket
------------------------

Get Moh to make your S3 bucket. If you hash all your static and media files you only need one.

You should note your bucket name, access key, and secret key for use later.

Once you have your bucket (it should be named `www-project_name-com` create a new cors.json file and upload the config to s3 by running:

    $ aws s3api put-bucket-cors --bucket www-project_name-com --cors-configuration file://cors.json

Example `cors.json`:

    {
      "CORSRules": [
        {
          "AllowedOrigins": ["https://project_name-develop.us-east-1.elasticbeanstalk.com", "https://www.project_name.com"],
          "AllowedMethods": ["GET"],
          "AllowedHeaders": ["Authorization"],
          "MaxAgeSeconds": 3000
        }
      ]
    }


3. Create your database
-----------------------

Create your database on rds however you want. Just note your database user and password and location for use later.


4. Create your application and environment
------------------------------------------

From your project root run:

    $ eb init  # creates new application

    $ eb create  # creates new environment and deploys application

Depending on which branch you are in, you will create the environment specified by your `config.global.yml` file. You can run `$ eb create` multiple times from different branches to create multiple environments for your project.

Once you run `$ eb create` EB will deploy your project but there will be errors. The environment variables aren't up yet. No worries though, we'll get to that.


5. Set up the environment
-------------------------

Save the current environment to a local `project_name-develop-sc.yml` file:

    $ eb config save

You will be prompted to enter a name for the environment. You can use the default which should be the name of the environment you created with `-sc` at the end.

Update the saved file with your project's configuration variables. In the previously created `project_name-develop-sc.yml` file, look for `OptionSettings:` then add `aws:elasticbeanstalk:application:environment:` inside that. Add all your environment configuration variables here. This is where your s3 keys and database info will be needed.

__* Warning: Don't commit this file, it has secrets__ add `.elasticbeanstalk/saved_configs/` to your `.gitignore`

Example of your `project_name-develop-sc.yml` file with some fake info:

    EnvironmentConfigurationMetadata:
      Description: Configuration created from the EB CLI using "eb config save".
      DateCreated: '1463088683000'
      DateModified: '1463088683000'
    SolutionStack: 64bit Amazon Linux 2015.09 v2.0.6 running Python 2.7
    OptionSettings:
      aws:elasticbeanstalk:sns:topics:
        Notification Endpoint: ''
      aws:elasticbeanstalk:application:environment:
        DJANGO_SECRET_KEY: asdfhjasdfkjhsakjdfkjhsadhjkadsjkhsdfhjk
        DJANGO_SETTINGS_MODULE: config.settings.production
        DJANGO_DEBUG: true
        PIP_TRUSTED_HOST: pypi.hzdg.com
        DATABASE_URL: postgres://user:pass@dburl/dbname
        PIP_EXTRA_INDEX_URL: https://pypi.python.org/simple/
        DJANGO_AWS_STORAGE_BUCKET_NAME: www-project_name-com
        DJANGO_AWS_ACCESS_KEY_ID: KLJLKJLKJLKJLK
        DJANGO_AWS_SECRET_ACCESS_KEY: LKJLDFKJLSDFJLSDKJFLKDJSF
        PIP_INDEX_URL: http://324234234:33234234243@pypi.hzdg.com/simple/
        DJANGO_SECURE_HSTS_SECONDS: 60
        DJANGO_SECURE_CONTENT_TYPE_NOSNIFF: false
        DJANGO_SECURE_SSL_REDIRECT: false
        DJANGO_SECURE_FRAME_DENY: false
      aws:elasticbeanstalk:environment:
        ServiceRole: aws-elasticbeanstalk-service-role
        EnvironmentType: SingleInstance
    EnvironmentTier:
      Type: Standard
      Name: WebServer
    AWSConfigurationTemplateVersion: 1.1.0.0

Just take note of the `aws:elasticbeanstalk:application:environment:` and all the nested variables there.

After your are done editing your environment, upload it to your saved environments:

    $ eb config put project_name-develop-sc  # store this environment

Then set it as your project's environment:

    $ eb config project_name-develop --cfg project_name-develop-sc

This will set the configuration of your environment called `project_name-develop` to the saved configuration called `project_name-develop-sc`. Just note the `-sc` that distinguishes the two.


5. Deploy your project
----------------------

Deploy your project again and you should be able to see it live:

    $ eb deploy

If you don't know your url just run:

    $ eb status

and look for the `CNAME`.

You can run `$ eb deploy` for your deployments in the future. Eb will deploy your repository and follow your `.gitignore`. If you want to change what eb ignores you can add a custom `.ebignore` file.

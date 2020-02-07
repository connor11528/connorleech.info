---
layout: post
title: Deploy Django to AWS (Part 1)
date: "2015-10-24"
category: Cloud
description: "In this tutorial we set up Elastic Beanstalk for our Django application to deploy it live to the internet with a Postgres database. We use the Elasic Beanstalk CLI to push our changes to AWS infrastructure."
author: Connor Leech
published: true
---

Following this [tutorial](https://realpython.com/blog/python/deploying-a-django-app-to-aws-elastic-beanstalk/).


```
$ git clone https://github.com/realpython/image-of-the-day.git
$ cd image-of-the-day
$ mkvirtualenv cptmusicblog
$ pip install -r requirements.txt
```

If the database is not running run:
```
$ postgres -D /usr/local/var/postgres
$ psql postgres
postgres=# create database cptmusicblog;
postgres=# \l
```

You should see a database named `cptmusicblog` in the output. `control + D` stops the postgres on Mac.

Modify `iotd/iotd/settings.py` to your local database settings:

```
if 'RDS_DB_NAME' in os.environ:
    DATABASES = {
       'default': {
             'ENGINE':'django.db.backends.postgresql_psycopg2',
             'NAME': os.environ['RDS_DB_NAME'],
             'USER': os.environ['RDS_USERNAME'],
             'PASSWORD': os.environ['RDS_PASSWORD'],
             'HOST': os.environ['RDS_HOSTNAME'],
             'PORT': os.environ['RDS_PORT'],
             }
          }
else:
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.postgresql_psycopg2',
            'NAME': 'cptmusicblog',
            'USER': '',
            'PASSWORD': '',
            'HOST' : 'localhost',
            'PORT' : '5432',
        }
    } 
```

Run some django commands to set up the database and create an admin user. The credentials you choose will allow you to log in to the admin section of the site.

```
$ python manage.py migrate
$ python manage.py createsuperuser
$ python manage.py runserver
```

Go to `http://localhost:8000/admin/` and log in with credentials. If you go to the root (`http://localhost:8000/`) you will see a django error. Now go add an image in the admin section under "Featured images". Once you upload you'll see your image on the homepage `/`.

![](/content/images/2015/06/Screen-Shot-2015-06-18-at-11-26-12-AM.png)


### CLI for AWS Elastic Beanstalk

Install this to your virtualenv:

```
$ pip install awsebcli
$ eb --version
```

Then sign in to [AWS](http://aws.amazon.com/):

![](/content/images/2015/06/Screen-Shot-2015-06-18-at-11-32-00-AM.png)

Okay then set up an elastic beanstalk environment:

```
$ eb init
```

![](/content/images/2015/06/Screen-Shot-2015-06-18-at-11-35-51-AM.png)

![](/content/images/2015/06/Screen-Shot-2015-06-18-at-11-38-50-AM.png)

I created a new ssh key for this named aws-eb. Once that command is done there is now a `.elasticbeanstalk` directory in your project.

Run `$ eb console` and your browser will open up


### Gif break

This is us right now:

![](http://media.giphy.com/media/vgxBQVRkJ5YBO/giphy.gif)

This is where we want to be: 

![](http://media.giphy.com/media/rjLBUukT0w1AA/giphy.gif)

This is going to happen first:

![](http://media.giphy.com/media/X5nQfDQnaTSvu/giphy.gif)

Don't do this:

![](http://media.giphy.com/media/2yU3Ex75PRjeE/giphy.gif)


### Configure environment

We want to have a development and production environment.

Run `$ eb create`

![](/content/images/2015/06/Screen-Shot-2015-06-18-at-11-47-54-AM.png)

So that looks good. We want to host our static files in an S3 bucket.

Elastic beanstalk created an environment and tried to deploy our app...

In the console (`eb console`) you can view the environment:

![](/content/images/2015/06/Screen-Shot-2015-06-18-at-11-50-59-AM.png)

Now create a database in the AWS console:

1. Click the “Configuration” link.
2. Scroll all the way to the bottom of the page, and then under the “Data Tier” section, click the link “create a new RDS database”.
3. On the RDS setup page change the “DB Engine” to “postgres”.
4. Add a “Master Username” and “Master Password”.
5. Save the changes

I saw this for a while:

![](/content/images/2015/06/loading-1.gif)


Create `.ebextensions/01_packages.config`:

```
packages:
  yum:
    git: []
    postgresql93-devel: []
```

### Fix mistrake

Actually `.elasticbeanstalk` goes in the root of the directory, so we have to move it, so from the `iotd` directory (where it is and not supposed to be).

Move the file:
```
$ mv .elasticbeanstalk ..   
$ cd ..
$ eb list
$ eb config cptmusicblog-dev
```

### change WSGIPath config
This pulls down a config file from AWS. After we edit the config the app will redeploy. We have to edit this part of the file:

```
aws:elasticbeanstalk:container:python:
  NumProcesses: '1'
  NumThreads: '15'
  StaticFiles: /static/=static/
  WSGIPath: application.py
```

Change the last line so that it reads:

```
 aws:elasticbeanstalk:container:python:
   NumProcesses: '1'
   NumThreads: '15'
   StaticFiles: /static/=static/
   WSGIPath: iotd/iotd/wsgi.py
```

Then run `$ eb deploy` after you save and exit the changes.

You can now go to the website (http://cptmusicblog-dev.elasticbeanstalk.com/ in my case) and see a nice friendly django error:

![](/content/images/2015/06/Screen-Shot-2015-06-18-at-1-05-32-PM.png)

This is good! We got the same thing when running locally before we uploaded the doge image. Go to `<your_eb_url>/admin` (so in my case `http://cptmusicblog-dev.elasticbeanstalk.com/admin`) and you will see the django admin login panel. Hooray!!

### Logging in to the admin

In `.elasticbeanstalk/02_python.config` we specify settings for AWS:

```
container_commands:
    01_migrate:
	    command: "source /opt/python/run/venv/bin/activate && python iotd/manage.py migrate --noinput"
	    leader_only: true
    02_createsu:
	  command: "source /opt/python/run/venv/bin/activate && python iotd/manage.py createsu"
	  leader_only: true
    03_collectstatic:
	  command: "source /opt/python/run/venv/bin/activate && python iotd/manage.py collectstatic --noinput"

option_settings:
    "aws:elasticbeanstalk:application:environment":
        DJANGO_SETTINGS_MODULE: "iotd.settings"
        "PYTHONPATH": "/opt/python/current/app/iotd:$PYTHONPATH"
        "ALLOWED_HOSTS": ".elasticbeanstalk.com"
    "aws:elasticbeanstalk:container:python":
        WSGIPath: iotd/iotd/wsgi.py
        NumProcesses: 3
        NumThreads: 20
    "aws:elasticbeanstalk:container:python:staticfiles":
        "/static/": "www/static/"
```

Notice the second step creates a superuser. The directory we cloned has a script in `iotd/images/management/commands/createsu.py`:

```
import os

from django.core.management.base import BaseCommand
from django.contrib.auth.models import User

class Command(BaseCommand):

    def handle(self, *args, **options):
        if not User.objects.filter(username="admin").exists():
            User.objects.create_superuser("admin", "admin@admin.com", "admin")
```

So when you go to `/admin` log in with `username: admin, password: admin`. Then you can upload an image and you will see doge again!

![](/content/images/2015/06/Screen-Shot-2015-06-18-at-1-27-39-PM.png)

### Changing where the images are hosted

So we are almost good. The problem now is that the images are hosted on the EC2 instance itself in the static files directory. We don't want that. Every time we run `eb deploy` we will create a brand new instance with a clean static files directory. We don't want to wipe our data every single time we deploy. 

Instead we will to host our data in an [S3 bucket](http://aws.amazon.com/s3/). The instances will talk to the bucket and render the data. That way we can deploy all day long and not erase the data we've uploded.

Try it! Run `eb deploy` and the image will disappear:

![](/content/images/2015/06/Screen-Shot-2015-06-18-at-1-40-09-PM.png)

The text will be stored in the database we set up but the static_files will be wiped.

### Gif break #2

This is us if we host our images on EC2 instances instead of S3 buckets:

![](http://media.giphy.com/media/zemslkaIiA2oE/giphy.gif)

I will continue on how to set up and configure S3 bucket for our app in part 2:

[deploy django to aws part 2: hosting files on S3](http://connorleech.info/deploy-django-to-aws-part-2-hosting-files-on-s3/)

If you don't need file upload and only a server and a database you are good to go. Otherwise see you "after the jump".


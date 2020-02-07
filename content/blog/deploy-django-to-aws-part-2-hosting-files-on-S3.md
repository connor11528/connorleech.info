---
layout: post
title: Deploy Django to AWS part 2 - Hosting Files on S3
date: "2015-10-24"
category: Cloud
lede: "This tutorial walks through the rudimentals of uploading a file to an S3 bucket using Django. We go through setting bucket permissions and navigating the AWS console."
author: Connor Leech
published: true
---

Continued from part 1: [deploy-django-to-aws](http://connorleech.ghost.io/deploy-django-to-aws/).

I'm learning from [this tutorial](https://www.caktusgroup.com/blog/2014/11/10/Using-Amazon-S3-to-store-your-Django-sites-static-and-media-files/).

### Set up s3 bucket

Go to https://console.aws.amazon.com/s3/home and click "Create Bucket"

![](/content/images/2015/06/Screen-Shot-2015-06-18-at-2-07-17-PM.png)


Once your bucket is created click on "Permissions":

![](/content/images/2015/06/Screen-Shot-2015-06-18-at-2-27-07-PM.png)

Then hit "Edit bucket policy". This is our bucket policy:

```
{
  "Statement": [
    {
      "Principal": {
          "AWS": "*"
      },
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": ["arn:aws:s3:::bucket-name/*", "arn:aws:s3:::bucket-name"]
    }
  ]
}
```

I got the policy from this [Stack Overflow post](http://stackoverflow.com/questions/10854095/boto-exception-s3responseerror-s3responseerror-403-forbidden).

Where `cptmusicblog` is the name of my bucket. Add a file and double click on it. You should be able to see it publicly. Here is a url for an image I uploaded:

https://s3-us-west-2.amazonaws.com/cptmusicblog/splash.png

### Set up django to get static files from s3

Install some packages and add them to `requirements.txt`:

```
$ pip install django-storages boto
$ pip freeze > requirements.txt
```

Add "storages" to INSTALLED_APPS in `iotd/iotd/settings.py`:

```
INSTALLED_APPS = (
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'images',
    'storages',
)
```

Then add more to settings.py:

```
AWS_HEADERS = {
    'Expires': 'Thu, 31 Dec 2099 20:00:00 GMT',
    'Cache-Control': 'max-age=94608000',
}

AWS_STORAGE_BUCKET_NAME = 'BUCKET_NAME'
AWS_ACCESS_KEY_ID = 'xxxxxxxxxxxxxxxxxxxx'
AWS_SECRET_ACCESS_KEY = 'yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy'
```

The AWS_HEADERS means that AWS tells browsers they can cache the files until 2099

![](http://media.giphy.com/media/3jmLczk5BbfZC/giphy.gif)

which happens to be in

![](http://media.giphy.com/media/11fp850173Eoyk/giphy.gif)

Get your access credentials by clicking your username dropdown > "Security credentials" > "Users".

If you haven't created a user you have to do that. Then click `Manage Access Keys`. Then `Create Access Key`. We can only view our private key once. Download the keys as a CSV file. Keep these secret so no one exploits them by mining bitcoin on your account with your credentials (do not push them to github).

Also add these lines to `settings.py`:

```
AWS_S3_CUSTOM_DOMAIN = '%s.s3.amazonaws.com' % AWS_STORAGE_BUCKET_NAME
STATIC_URL = "https://%s/" % AWS_S3_CUSTOM_DOMAIN
STATICFILES_STORAGE = 'storages.backends.s3boto.S3BotoStorage'
```


### CORS configuration

Go to S3 bucket preferences and under "edit CORS configuration" paste this in:

```
<CORSConfiguration>
        <CORSRule>
            <AllowedOrigin>*</AllowedOrigin>
            <AllowedMethod>GET</AllowedMethod>
            <MaxAgeSeconds>3000</MaxAgeSeconds>
            <AllowedHeader>Authorization</AllowedHeader>
        </CORSRule>
    </CORSConfiguration>
```

### Try it

```
$ python iotd/manage.py collectstatic
$ eb deploy cptmusicblog-dev
```
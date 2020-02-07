---
layout: post
title: Make a deck of cards with Django
date: "2015-10-23"
category: Programming
description: "In this tutorial we set up a Python Django project with a virtual environment, create a Cards model and then render the data to the HTML view."
author: Connor Leech
published: true
---

Learn you Django by making a deck of cards out of internets.

For the impatient, [source code is here](https://github.com/connor11528/django_cards)

Start the project

```
$ django-admin startproject django_cards
$ cd django_cards 
```

create a virtual environment and activate it

```
$ virtualenv venv
New python executable in venv/bin/python2.7
Also creating executable in venv/bin/python
Installing setuptools, pip...done.

$ . venv/bin/activate
(venv) $
```

venv is a convention. It creates a new environment for our project where we'll install django and other packages. A virtual environment is like a virtual machine with way less overhead. Add venv to .gitignore for version control.

Install django and generate a requirements.txt file. This is where all the packages that our app requires will be defined. It's like a gemfile in ruby or package.json in node.

```
$ pip install django
$ pip freeze > requirements.txt
```

This creates our project directory. Now we create a new app called cards where we will have all of our application logic.

```
$ django-admin.py startapp cards
```

cards is now an app we'll use within the django\_cards project. In django\_cards/settings.py add cards to our installed apps:

```

INSTALLED_APPS = (
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'cards',
)
```

Create a card model in cards/models.py. This will define the columns in our

```
from django.db import models

class Card(models.Model):
    SPADE = 0
    CLUB = 1
    DIAMOND = 2
    HEART = 3
    SUITS = (
        (SPADE, "spade"),
        (CLUB, "club"),
        (DIAMOND, "diamond"),
        (HEART, "heart")
    )
    suit = models.PositiveSmallIntegerField(choices=SUITS)
    rank = models.CharField(max_length=5)
```

Add to a new file cards/utils.py:

```
from models import Card

def create_deck():
    """
    Create a list of playing cards in our database
    """
    suits = [0, 1, 2, 3]
    ranks = ['two', 'three', 'four', 'five', 'six', 'seven', 'eight', 'nine', 'ten', 'jack', 'queen', 'king', 'ace']
    cards = [Card(suit=suit, rank=rank) for rank in ranks for suit in suits]
    Card.objects.bulk_create(cards)
```

Now we're going to set up our database. Make the migrations and migrate the database. We're also going to run the create deck function.

```
$ python manage.py makemigrations
$ python manage.py migrate
$ python manage.py shell
>> from cards.utils import create_deck
>> create_deck()
```

Next we'll set up routing. In the project directory (django_cards) add a new route: 

```
url(r'^$', 'cards.views.home', name='home'),
```

This regex matches our index route and will use the view as defined in the views.py file of our card app. The last parameter gives our route a name.

Define the home view:

```
from models import Card

def home(request):
	data = {
		'cards': Card.objects.all()
	}
	return render(request, 'cards.html', data)
```

This provides all the card data to our template. Inside of the cards directory add a templates/cards.html. This will be our homepage. Django automatically knows to search for cards.html inside of a templates directory.

Add a basic loop to templates/cards.html:

```
<div>
    {% for card in cards %}
        <p>Suit: {{ card.suit }}, Rank: {{ card.rank }}</p>
    {% endfor %}
</div>
```

Then run the server and see the card data displayed on localhost:8000

```
$ python manage.py runserver
```

We'll make our template a bit more snazzy:

```
We have {{ cards | length }} cards!

{% for card in cards %}
    <div>
        <p>
            Capitalized Suit: {{ card.get_suit_display | capfirst }} <br>
            Uppercased Rank: {{ card.rank | upper }}
        </p>
    </div>
{% endfor %}
```

We now use get\_FIELD_display to show the suit name. This is a bit magical to me, had to [investigate on SO](http://stackoverflow.com/questions/5730211/how-does-get-field-display-in-django-work).


In future tutorials we will build up actual functionality to play cards. Keep an eye on the [SOURCE code](https://github.com/connor11528/django_cards) for this tutorial!
---
layout: post
title: How to process payments with Node.js, Vue 2 and Stripe
date: "2017-08-19"
category: Javascript
description: "This post goes through building a website for selling products. We set up a Vue client side application and Node.js server side app. The apps use Stripe Javascript libraries for payment processing. We deploy the static app to Amazon S3 and deploy the server to the cloud using Heroku"
author: Connor Leech
published: true
---

In this tutorial we are going to build a website for selling artisan bundles of sticks online. It is going to be called Stickly! In order to make this site we're going to need a Product display page, an Order page and a Confirmation details page. We'll accept payments using Stripe's client side and server side Javascript libraries.

To build the frontend we'll use Vue 2 with [vue-router](https://github.com/vuejs/vue-router) for client side routing.  The frontend application will be responsible for collecting user information and sending a request to Stripe for a special token. Once we have that token from Stripe, our client side code will forward the token to our server in order to initiate the charge.

The server will be made from Node.js using express and its associated utility libraries. There will be two endpoints. One for creating a new charge and the other for returning details about a particular charge, based on a unique identifier that we get from Stripe.

In order to build this application and start selling online, we are going to utilize third party services. To build a real world website that accept actual payments on the internet you'll need to sign up for the following services and locate your special API keys.

- [Stripe](https://stripe.com/): Stripe is the current standard for developers accepting payments in the US. There are many competitors out there but Stripe's extensive documentation and ease of use has made it the de facto choice for payment processing online. We are going to use their [client side](https://stripe.com/docs/stripe.js?) Javascript library and their server side [Node.js package](https://stripe.com/docs/api/node).
- [Amazon Simple Storage Service (S3)](https://aws.amazon.com/s3/): Amazon S3 is part of Amazon Web Services, the infastructure that powers a very large portion of the internet. The Simple Storage Solution (S3) provides a way to host static assets like webpages, images or videos and make them globally accessible. You can create a free AWS account [here](https://aws.amazon.com/free).
- [Heroku](https://www.heroku.com/): Heroku is a cloud application platform that was acquired by Salesforce. It simplifies the task of deploying and maintaining cloud infastructure. We are going to use Heroku [Command Line Interface (CLI) ](https://devcenter.heroku.com/articles/getting-started-with-nodejs#set-up) to deploy our Node.js server to the internet.

![](https://cdn.scotch.io/2842/b7yhhuUPSGO1fEkMHD6P_sticks.jpeg)

- Demo: http://stickly.io/#/
- Frontend code: https://github.com/connor11528/stickly-frontend
- Backend code: https://github.com/connor11528/stickly-backend

## Frontend Application

First we are going to build the frontend Vue.js application. This is the website portion that the user will see. Ultimately it will be hosted using Amazon S3, but first we have to set everything up.

The Vue.js core team maintains a collection of generators for scaffolding new projects. For this tutorial we are going to use the [webpack-simple template](https://github.com/vuejs-templates/webpack-simple) to quickly get a client side Vue.js up and running. This will include a custom Webpack setup and specific `.vue` files that contain our HTML, CSS and Javascript for dedicated components.

If you have not used Vue before you are in for a treat! If you are experienced with Vue this setup should be very straightforward to you. 

We are going to use [Bulma](http://bulma.io/) for styling, which is a Flexbox based CSS library. If you are new to using CSS Flexbox you can check out the Scotch.io article [here](https://scotch.io/tutorials/a-visual-guide-to-css3-flexbox-properties). In depth knowledge of Flexbox is not a prerequisite for this tutorial though!

### Create the Vue.js app
Create the Vue.js static website app with the following commands in your command line interface.

```sh
$ vue init webpack-simple YOUR_APP_NAME
$ cd YOUR_APP_NAME
$ npm install
$ npm install vue-router moment --save
```

This generates our new project and includes the additional dependecies for client side routing and date formating. 

### Instatiate the application
Our *src/main.js* file is going to be the nerve center for our application. This is the first file to get called and it will call all of our components and link our Javascript application to the index.html page. Let's import all of the components, libraries and setup we need for our application within this file.

**./src/main.js**
```js
// Environment configuration
if (process.env.NODE_ENV === 'production') {
    window.endpoint = 'https://stickly.herokuapp.com';
} else {
    window.endpoint = 'http://localhost:3000';
}

// Global Variables
window.moment = require('moment');
window.axios = require('axios');
window.axios.defaults.headers.common['X-Requested-With'] = 'XMLHttpRequest';

import Vue from 'vue'
import VueRouter from 'vue-router'
import App from './components/App.vue'
import Product from './components/Product.vue'
import Order from './components/Order.vue'
import Complete from './components/Complete.vue'

Vue.use(VueRouter);

// Register routes
const routes = [
  { name: 'home', path: '/', component: Product },
  { name: 'order', path: '/order', component: Order },
  { name: 'order-complete', path: '/order-complete/:id', component: Complete }
];

const router = new VueRouter({
  routes
});

// Instantiate application to the DOM
new Vue({
    router,
    el: '#app',
    data(){
        return {
            order_details: {}
        }
    },
    render: h => h(App)
});
```

The first thing we do here is determine whether we are in our production or development environment. This value will be set by webpack when we run `npm run dev` (for development) or `npm run build` (for production). Based on the environment we set a global variable attached to the browser's window object that determines which URL our Vue app will make XHR (ajax) requests to. For making asynchronous requests from the background we are going to use [axios](https://github.com/mzabriskie/axios).

Next we register the components we will need and bind them to specific routes. This is the pattern required for setting up client side routing with [vue-router](https://github.com/vuejs/vue-router), the official Vue.js routing package maintained by the Vue.js core team.

We will also need to modify our *index.html* page to include the Stripe javascript library, the Bulma CSS framework, style classes and Font Awesome.

**./index.html**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">

    <title>Stickly</title>

    <!-- Styles -->
    <link rel='stylesheet' href='https://maxcdn.bootstrapcdn.com/font-awesome/4.7.0/css/font-awesome.min.css'>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/bulma/0.5.1/css/bulma.css">
    <link rel="stylesheet" href="./css/main.css">

</head>
<body>
  <div class="hero is-fullheight">
    <div id='app'></div>
  </div>

  <!-- Scripts -->
  <script type="text/javascript" src="https://js.stripe.com/v2/"></script>
  <script src="/dist/build.js"></script>
</body>
</html>
```

Our Vue.js application will be bound to *index.html* through the "app" id tag where it will mount our App component. This app component will display most of our content and our Vue.js application logic. It displays a navigation section and the special `<router-view></router-view>` component. This component tells vue-router where to mount our views. The router takes the current URL and mounts the component to this space accordingly. 

```html
<template>
  <div id="app">
    <!-- Nav -->
        <div class="hero-head">
          <div class="container">
            <nav class="nav">
              <div class="container">
                <div class="nav-left">
                  <router-link to="/" class="nav-item">Stickly</router-link>
                </div>
                <span class="nav-toggle">
                  <span></span>
                  <span></span>
                  <span></span>
                </span>
                <div class="nav-right nav-menu">
                    <a class="nav-item" href='mailto:support@stickly.com'>
                      Contact
                    </a>
                </div>
              </div>
            </nav>
          </div>
        </div>

    <router-view></router-view>
  </div>
</template>

<script>
export default {
  name: 'app',
  data () {
    return {}
  }
}
</script>
```

The App componet is the heart of our application. All of our other Vue components will render where the router-view component is. This way we do not need to duplicate code for the Nav bar or HTML headers on every page. 

We defined in our *main.js* file that when the user hits the root of our application the Product component should render. Let's define that now!

### Create the Product page

The product page is the landing page that entices the user to buy our product, in this case a bundle of sticks. There is an open source template called [dansup/bulma-templates](https://github.com/dansup/bulma-templates) that we can use to quickly style the product page. There are lots of example templates for Bulma that you can browse [here](https://dansup.github.io/bulma-templates/) and [here](http://bulma.io/expo/), depending on what you are building. The product page template is a good starting point, though I have made some minor modifications.

![](https://cdn.scotch.io/2842/QVDxcSwcSJ2rv5yjhZqf_Screen%20Shot%202017-08-16%20at%2012.38.58%20PM.png)

**./src/components/Product.vue**

```html
<template>
    <div class="section">
        <div class="container">
            <div class="columns" style='min-height:800px;'>
                <div class="column is-6">
                    <div class="image is-2by2">
                        <img src="https://cdn.scotch.io/2842/b7yhhuUPSGO1fEkMHD6P_sticks.jpeg">
                    </div>
                </div>
                <div class="column is-5 is-offset-1">
                    <div class="title is-2">Bundle of Sticks</div>
                    <p class="title is-3 has-text-muted">$ 15</p>
                    <hr>
                    <br>
                    <p>This bundle of sticks provides an earthy fragrance and aesthetic natural appeal. Your bundle of sticks will come with a personalized note. The perfect natural gift for any time of year!</p>
                    <br>
                    <br>
                    <p class='level'>
                        <router-link to="/order" class="button is-large is-primary level-item has-text-centered">Buy Now</router-link>
                    </p>
                    <br>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {}
</script>
```

As you can see this component is all plain HTML with Bulma's specific classes for styling. The only Vue specific part of this file is the `<router-link></router-link>` component. This automatically generates an anchor tag and connects the route to the routes we defined for vue-router in our *main.js* file. 

When the user clicks the Order Now button they will be redirected to the Order page where they can fill out their details and submit their information for payment processing. 

### The Order page
The order page displays fields for the user's contact details (name and email), their shipping information, a personalized note for the bundle of sticks and fields for their credit card details. Additionally, we do checks to make sure that the credit card fields are filled out correctly with some Vue.js form validation techniques in the `validate` method.

**./src/components/Order.vue** (the html portion)
```html
<template>
    <div class='container'>
        <div class='columns'>
            <div class='column is-4'>
                <img src="https://cdn.scotch.io/2842/b7yhhuUPSGO1fEkMHD6P_sticks.jpeg" alt="A bundle of sticks">
            </div>
            <div class='column is-4'>            
                <div class='field'>
                    <label>Name</label>
                    <div class="control has-icons-left has-icons-right">
                        <input class="input" type="text" placeholder="First and Last" v-model='name'>
                        <span class="icon is-small is-left">
                          <i class="fa fa-user"></i>
                        </span>
                    </div>
                </div>
                <div class="field">
                    <label>Email</label>
                    <div class="control has-icons-left has-icons-right">
                        <input class="input" type="text" placeholder="Email address" v-model='email'>
                        <span class="icon is-small is-left">
                            <i class="fa fa-envelope"></i>
                        </span>
                    </div>
                </div>

                <hr id='left-line'>
                                
                <div class="field">
                    <label for="card_number">Card Number</label>
                    <input id="card_number" v-model="card.number" type="text" :class="['is-danger' ? cardNumberError : '', 'input']" placeholder='4242424242424242'>
                    <span class="help is-danger" v-show="cardNumberError">
                        {{ cardNumberError }}
                    </span>
                </div>

                <div class="field">
                    <label for="cvc">CVC</label>
                    <input id="cvc" v-model="card.cvc" type="text" class="input" placeholder='123'>
                    <span class="help is-danger" v-show="cardCvcError">
                        {{ cardCvcError }}
                    </span>
                </div>

                <div class="field">
                    <label for="exp_month">Expiry Month</label>
                    
                    <input id="exp_month" v-model="card.exp_month" type="text" :class="['is-danger' ? cardMonthError : '', 'input']" placeholder="MM">
                    <span class="help is-danger" v-show="cardMonthError">
                        {{ cardMonthError }}
                    </span>
                </div>

                <div class="field">
                    <label for="exp_month">Expiry Year</label>
                    <input id="exp_year" v-model="card.exp_year" type="text" :class="['is-danger' ? cardYearError : '', 'input']" placeholder="YY">
                    <span class="help is-danger" v-show="cardYearError">
                        {{ cardYearError }}
                    </span>
                </div>
                    
                <div class="help is-danger" v-if="cardCheckError">
                    <span>{{ cardCheckErrorMessage }}</span>
                </div> 
            </div>
            <div class='column is-4'>         
                <label>Special Note</label>
                <textarea class="textarea" placeholder="What would you like the note to say?" v-model='specialNote'></textarea>
                
                <hr>
                
                <div class="field">
                    <label>Address</label>
                    <input type='text' class='input' v-model='address.street' placeholder='123 Fake St #303'>
                </div>
                <div class="field">
                    <label>City</label>
                    <input type='text' class='input' v-model='address.city' placeholder='San Francisco'>
                </div>
                <div class="field">
                <label>State</label>
                    <input type='text' class='input' v-model='address.state' placeholder='CA'>
                </div>
                <div class="field">
                    <label>Zip</label>
                    <input type='text' class='input' v-model='address.zip' placeholder='94607'>
                </div>
            </div>
        </div>
        <div class="columns">
            <div class='column is-12'>
                <button type="submit" class="button is-primary is-large is-pulled-right" @click.prevent="validate" :disabled="cardCheckSending">
                    <span v-if="cardCheckSending">
                        <i class="fa fa-btn fa-spinner fa-spin"></i> 
                        Ordering...
                    </span>
                    <span v-else>
                        Place Order
                    </span>
                </button>
            </div>
        </div>
    </div>
</template>
<style>
h2 { text-decoration: underline; }
.textarea:not([rows]) { max-height: 110px; min-height: 110px; }
.container { margin-bottom: 30px; }
.column > img { margin-top: 60px; }
.button-field { display: flex; justify-content: center; }
#left-line { margin-top:27px; }
</style>
```
In the above template code we style the page and provide an image of the product the customer is about to buy. Additionally, we bind the input for these form fields using `v-model`. We'll have to define these fields in the vue object's data property, featured below. We also provide helper classes bound to specific properties to show red error messages when forms are not filled out properly or left blank.

The Javascript logic for this component lives in the same portion and is featured below. This code is responsible for binding data to the form fields, obtaining a token from Stripe and sending the payment request to our Node.js server.

**./src/components/Order.vue** (the JS portion)
```js
<script>
import axios from 'axios';

export default {
    data(){
        return {
            stripeKey: 'YOUR_STRIPE_PUBLIC_KEY',

            // fields
            name: 'Connor Leech',
            email: 'connor@employbl.com',
            specialNote: 'This is the text to put on the bundle of sticks',
            address: {
                street: '123 Something Lane',
                city: 'San Francisco',
                state: 'CA',
                zip: '94607'
            },

            card: {
                number: '4242424242424242',
                cvc: '123',
                exp_month: '01',
                exp_year: '19'
            },

            // validation
            cardNumberError: null,
            cardCvcError: null,
            cardMonthError: null,
            cardYearError: null,
            cardCheckSending: false,
            cardCheckError: false,
            cardCheckErrorMessage: ''
        }
    },
    methods: {
        validate(){
            this.clearCardErrors();
            let valid = true;
            if(!this.card.number){ valid = false; this.cardNumberError = "Card Number is Required"; }
            if(!this.card.cvc){ valid = false; this.cardCvcError = "CVC is Required"; }
            if(!this.card.exp_month){ valid = false; this.cardMonthError = "Month is Required"; }
            if(!this.card.exp_year){ valid = false; this.cardYearError = "Year is Required"; }
            if(valid){
                this.createToken();
            }
        },
        clearCardErrors(){
            this.cardNumberError = null;
            this.cardCvcError = null;
            this.cardMonthError = null;
            this.cardYearError = null;
        },
        createToken() {
            this.cardCheckError = false;
            window.Stripe.setPublishableKey(this.stripeKey);
            window.Stripe.createToken(this.card, $.proxy(this.stripeResponseHandler, this));
            this.cardCheckSending = true;
        },
        stripeResponseHandler(status, response) {
            this.cardCheckSending = false;
            if (response.error) {
                this.cardCheckErrorMessage = response.error.message;
                this.cardCheckError = true;

                console.error(response.error);
            } else {

                // token to create a charge on our server 
                var token_from_stripe = response.id;

                var request = {
                    name: this.name,
                    email: this.email,
                    engravingText: this.engravingText,
                    address: this.address,
                    card: this.card,
                    token_from_stripe: token_from_stripe
                };

                // Send to our server
                axios.post(`${window.endpoint}/charge`, request)
                    .then((res) => {
                        var error = res.data.error;
                        var charge = res.data.charge;
                        if (error){
                            console.error(error);
                        } else {
                            this.$router.push({ path: `order-complete/${charge.id}` })
                        }
                    });
            }
        }
    }
}
</script>
```

The `createToken()` function uses the Stripe.js library to create a unique token for our particular charge request. Once we have that token, we trigger the `stripeResponseHandler` method. This checks that there are no errors and submits the request to our server for processing. If we successfully make a charge on our Node.js server using the token provided to us by Stripe.js we redirect the user to the Order Complete page. There is additional logic here for rendering the validation errors as well.

### Order Complete page

And finally, we get to the order complete page. This page will be rendered after we successfully create the charge on our server. The URL will include a unique id associated with the particular charge we created. This URL will be persistent so the user can browse back to it at any time in order to view their order details.

**./src/components/Complete.vue**
```html
<template>
    <div class='container content'>
        <br>
        <h3 style='color: white;'>Order complete!</h3>

        <p>Congratulations! Your order for Sticks will be shipped out within 1-2 business days.  <a href=''>support@stickly.com</a>. We sent you a confirmation email for your records. Thanks so much!</p>

        <div v-if='orderDetails'>
            <dl>
                <dt>Order Number</dt>
                <dd>{{ orderDetails.id }}</dd>
                <dt>Order Created</dt>
                <dd>{{ orderDetails.created | moment }}</dd>
                <dt>Payment Amount</dt>
                <dd>{{ orderDetails.amount | currency }}</dd>
                <dt>Shipping Address</dt>
                <dd>{{ orderDetails.shipping.address.line1 }}, {{ orderDetails.shipping.address.city }}, {{ orderDetails.shipping.address.state }} {{ orderDetails.shipping.address.postal_code }}</dd>
                <dt>Engraving Text</dt>
                <dd>{{ orderDetails.description }}</dd>
                <dt>Email</dt>
                <dd>{{ orderDetails.receipt_email }}</dd>
            </dl>
        </div>
    </div>
</template>
<style>
    dt { font-weight: bold; }
</style>
<script>
import axios from 'axios';
import moment from 'moment';

export default {
    data(){
        return {
            orderDetails: false
        };
    },
    created(){
        var charge_id = this.$route.params.id;
        axios.get(`${window.endpoint}/charge/${charge_id}`)
            .then((res)=>{
                this.orderDetails = res.data.charge;
            });
    },
    filters: {
        moment(date) {
            return moment.unix(date).format('MMMM Do, YYYY - h:mm a');
        },
        currency(amount){
            return `$${(amount/100).toFixed( 2 )}`;
        }
    }
}
</script>
```

In this file we hit our backend again to grab the details for the order from the identification number in the URL. We then bind that data to render on the screen using Vue.js. We also define moment and currency [output filters](https://vuejs.org/v2/guide/migration.html#Filters) to format how the currency and dates are rendered to the screen.

### Testing the Frontend

In your console run `npm run dev` and the frontend page of our website will be rendered out to http://localhost:8080/#/ in a new tab. You should see the following screens for viewing the product and filling out order details.

![](https://cdn.scotch.io/2842/J21r8hIiTPWaTU33Ga2Q_Screen%20Shot%202017-08-18%20at%201.29.30%20PM.png)

![](https://cdn.scotch.io/2842/Px9w67X8RVaiDz1fWuy6_Screen%20Shot%202017-08-18%20at%201.30.30%20PM.png)

In the next section we will set up the Node.js server to create the actual charges.

Here is the source code repository for the frontend application code: https://github.com/connor11528/stickly-frontend

Let's set up the server!

## Backend Application

Our server will be responsible for charging the user and providing the details about the order back to the customer. We are going to create a brand new project directory to keep our Node.js application. This will be a separate code repo from the frontend and deployed separately. 

First let's create our Node.js server projec and check the project into Git for version control. We will use git later on in order to deploy our app to the internet with Heroku.

```sh
$ mkdir stickly-backend
$ cd stickly-backend
$ npm init
$ touch server.js .env .gitignore
$ git init
$ git add -A
$ git commit -m 'initial commit'
```

This will set up an empty skeleton for our Node.js project and check the changes into git for version control. Make sure to add a `.gitignore` file that includes `.env` so that you do not accidentally push your secret keys to the internet. That file looks like this:

**./.env**
```
STRIPE_SECRET_KEY=sk_test_XXXXXXXXXXXXXXXXXXXXXXX
```

There are a few npm packages that we need. These packages will be added to the *package.json* file due to the `--save` flag. The `npm init` command created our *package.json* file. To install additional Node.js packages with npm run:

```
$ npm install express body-parser cors dotenv method-override stripe --save
```
Most of these are helper packages that help the express Node.js web server accept and handle incoming requests from the internet. The Stripe package is for sending and processing payments.

We're going to define our application and routes in a *server.js* file like so:

**./server.js**

```js
// Grabs our environment variables from the .env file
require('dotenv').config();

var express = require('express'),
    bodyParser = require('body-parser'),
    methodOverride = require('method-override'),
    cors = require('cors'),
    app = express();

// Environment configuration
var env = process.env.NODE_ENV = process.env.NODE_ENV || 'development';
var port = process.env.PORT || 3000;

// Create a server side router
var router = express.Router();

// Configure express to handle HTTP requests
app.use(cors());
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({
  extended: true
})); 
app.use(methodOverride());

// Configure our Stripe secret key and object
var stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);

// Define the route endpoints for our application

// Create a new charge
router.post('/charge', function(req, res){
    // Create the charge object with data from the Vue.js client
    var newCharge = {
        amount: 23500,
        currency: "usd",
        source: req.body.token_from_stripe, // obtained with Stripe.js on the client side
        description: req.body.specialNote,
        receipt_email: req.body.email,
        shipping: {
            name: req.body.name,
            address: {
                line1: req.body.address.street,
                city: req.body.address.city,
                state: req.body.address.state,
                postal_code: req.body.address.zip,
                country: 'US'
            }
        }
    };

    // Call the stripe objects helper functions to trigger a new charge
    stripe.charges.create(newCharge, function(err, charge) {
        // send response
        if (err){
            console.error(err);
            res.json({ error: err, charge: false });
        } else {
            // send response with charge data
            res.json({ error: false, charge: charge });
        }
    });
});

// Route to get the data for a charge filtered by the charge's id
router.get('/charge/:id', function(req, res){
    stripe.charges.retrieve(req.params.id, function(err, charge) {
        if (err){
            res.json({ error: err, charge: false });
        } else {
            res.json({ error: false, charge: charge });
        }
    });
});

// Register the router
app.use('/', router);

// Start the server
app.listen(port, function(){
  console.log('Server listening on port ' + port)
});
```

Congrats! We have our application set up in our local environment. To start the Vue application run `npm run dev` in the command line. Open a separate window and run `node server`. We can then trigger charges and navigate to the various web pages and hit the URL endpoints. In the next section we are going to deploy these applications to the internet.

## Deployment

Our frontend application is going to be hosted on Amazon S3. Our Node.js server will be hosted on Heroku.

### Deploy Vue.js app to S3
First, we are going to deploy our static website to Amazon Simple Storage Service (S3). If you are not familiar with S3 it is also possible to deploy to other static hosting services such as Github Pages or Netlify. S3 is very affordable and has the added benefit of scaling infinitely, in case A LOT of people want to buy sticks.

Head into the AWS Consle and select **S3**. Once there, select **Create Bucket**.

![](https://cdn.scotch.io/2842/D1CmkEHsSiu1eBTjMqdI_Screen%20Shot%202017-08-18%20at%202.01.50%20PM.png)

The bucket name must be globally unique so you will not be able to use the same bucketname as me. Since this server will be hosting our code we want it publically accessble to everyone. Select **Grant public read access**.

![](https://cdn.scotch.io/2842/K38XsXD3S1K0qGWgYi9O_Screen%20Shot%202017-08-18%20at%202.03.53%20PM.png)

After you created the bucket, click into it and select **Properties** and then go to **Static Website Hosting**. This will serve up our **index.html** file from the bucket's unique URL.

Congrats! You have set up an S3 bucket where we can host our website. To build our Vue.js app for production, head into the frontend repo and run:

```
$ npm run build
```
This will generate a `dist` folder of our bundled Javascript files ready for production. We are going to then upload this folder and index.html to our S3 bucket. Go to your bucket and select **Upload**. Drag and drop our Vue.js app into the bucket.

![](https://cdn.scotch.io/2842/FUTk5EZOQHSVv7aH0Wqx_Screen%20Shot%202017-08-18%20at%202.11.36%20PM.png)

Click **Next**. On the following page, select **Grant public read access to this object(s)**. Everything else can be kept as is. 

Now if you head to http://YOUR_BUCKET_NAME.s3-website-REGION.amazonaws.com/#/ you will be able to see your site live on the internet. For example, the link for my bucket is:

http://stickly.io.s3-website-us-west-1.amazonaws.com/#/

We will sort out a nicer domain name (optional step), but first, let's deploy our server using Heroku.

### Deploy Node.js app to Heroku
We need to add one file called a Procfile that tells Heroku how to start our server. It looks like this:

**Procfile**
```
web: node server.js
```

Head into our server-side repo and make sure to install the heroku Command Line Interface (CLI). Check the project into git and create a new heroku application with the following commands:

```
$ git init 
$ git add -A
$ git commit -m 'initial commit'
$ heroku create MY_APP_NAME
```

We also have to set our environment variables directly with Heroku. Run the following additional commands:

```
$ heroku config:set STRIPE_SECRET_KEY=sk_test_XXXXXXXXXXXXXXXXXXXXXXX
```

To deploy our server and view it in the browser run:

```
$ git push heroku master
$heroku open 
```

Our backend is live! Now when you navigate to the live frontend site our Vue.js app will call route functions on our Heroku server. Make sure that you update *src/main.js* environment configuration with you Heroku app name. This way in development we will make XHR (ajax) requests to localhost:3000 but in production requests go to our cloud server.

To accept real money you can update your Stripe environment variables with your production keys.

Congrats on making it through this tutorial! You can view the source code for these applications below:

- Demo: http://stickly.io/#/
- Frontend code: https://github.com/connor11528/stickly-frontend
- Backend code: https://github.com/connor11528/stickly-backend

Thanks for reading!

<hr>

Also published on Medium as [Standing on the shoulders of Giants — Node.js, Vue 2, Stripe, Heroku and Amazon S3](https://medium.com/@connorleech/standing-on-the-shoulders-of-giants-node-js-vue-2-stripe-heroku-and-amazon-s3-c6fe03ee1118)







---
layout: post
title: Build Google Maps Typeahead Functionality with Vue.js and Laravel 5.3
date: "2016-12-28"
category: Javascript
lede: "Use the Google Places API to prepopulate an address side on the client side and integrate it into a Laravel/Vue.js application"
author: Connor Leech
published: true
---

![](https://cdn-images-1.medium.com/max/800/1*L8eYtCAprc3LGoVKhTTcWA.png)
<span class="figcaption_hack">What we’ll be building!</span>

In [my last
post](https://medium.com/@connorleech/generate-authentication-for-a-laravel-5-3-web-app-384781a5529f#.yvtkouxeh)
we created a Laravel 5.3 application and added authentication. In this tutorial
we are going to add Google Maps API typeahead functionality for selecting
addresses.

In this app, we want users to be able to see places as they type. For this we’ll
[GuillaumeLeclerc](https://github.com/GuillaumeLeclerc)/[vue-google-maps](https://github.com/GuillaumeLeclerc/vue-google-maps)
component, specifically the placesInput component.

    $ npm i vue-google-maps --save

Rename **resources/assets/js/components/Example.vue** to
**resources/assets/js/components/LocationInput.vue. **In that file we will
create our custom Vue component.

    <template>

    <place-input
                :place.sync="placeInput.place"
                :types.sync="placeInput.types"
                :component-restrictions.sync="placeInput.restrictions"
                class='form-control'
        ></place-input>

    <pre>{{ placeInput.place | json }}</pre>

    </template>

    <script>
        import { PlaceInput, Map } from 'vue-google-maps'

        export default {
           data() {
                return {
                    placeInput: {
                        place: {
                            name: ''
                        },
                        types: [],
                        restrictions: {'country': 'usa'}
                    }
                }
            },
            components: {
                PlaceInput
            }
        }
    </script>
    <style>
        label { display: block; }
    </style>

We also have to create a Google Maps API key at
[console.developers.google.com](http://console.developers.google.com/). Then
register the component and initialize Google Maps in
**resources/assets/js/app.js.**

    import { load } from 'vue-google-maps'

    load({
      key: 'YOUR_API_KEY',
      v: '3.24',                // Google Maps API version
      libraries: 'places',   // If you want to use places input
    });

    Vue.component('locationInput', require('./components/LocationInput.vue'));

    const app = new Vue({
        el: 'body'
    });

To use our component drop it into **resources/views/home.blade.php**:

    ('layouts.app')

    ('content')
    <div class="container">
        <div class="row">
            <div class="col-md-8 col-md-offset-2">
                <location-input></location-input>
            </div>
        </div>
    </div>

To make the app run and have webpack watch for file changes open up two terminal
windows and run:

    $ php artisan serve
    $ gulp watch

Source code:
[https://github.com/connor11528/laravel-5.3-app](https://github.com/connor11528/laravel-5.3-app)

If you enjoyed this tutorial give it a recommend or share [on
twitter](https://twitter.com/Connor11528). Thanks for reading!


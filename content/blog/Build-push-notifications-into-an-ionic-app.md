---
layout: post
title: Build push notifications into an Ionic 1.x app
date: "2015-10-23"
category: Javascript
lede: "Ionic is a framework for using Angular and Javascript to build native mobile applications. This post goes through adding push messaging using Ionic 1.x and Angular 1.x. It uses Phonegap under the hood."
author: Connor Leech
published: true
---

sample code on [github here](https://github.com/connor11528/ionic-push-starter).

<!-- more -->
```
$ ionic start ionic-push-starter io
$ cd ionic-push-starter
$ ionic upload
```

Your app should be available at [ionic.io](https://apps.ionic.io/apps). Theoretically you can view your apps with Ionic View, a mobile app. I'm running android 4.1.2 and it no workey.

Add a plugin for connecting to the internet with android:

```
$ cordova plugin add cordova-plugin-whitelist
```

With my testing my other ionic apps missing this plugin was driving me crazy [this blog post](https://blog.nraboy.com/2015/05/whitelist-external-resources-for-use-in-ionic-framework/) helped me out. The author has a lot of good tutorials.

Paste in this code to `www/js/app.js`:

```
.config(['$ionicAppProvider', function($ionicAppProvider) {
  // Identify app
  $ionicAppProvider.identify({
    // The App ID (from apps.ionic.io) for the server
    app_id: 'YOUR_APP_ID',
    // The public API key all services will use for this app
    api_key: 'YOUR_PUBLIC_API_KEY',
    // The GCM project ID (project number) from your Google Developer Console (un-comment if used)
    // gcm_id: 'YOUR_GCM_ID'
  });
}])
```

The `app_id` comes from https://apps.ionic.io/apps and look your at your app's id. `api_key` comes from your apps page > Settings > Keys > Public Key.

```
$ ionic platform add android
$ ionic build android
```

This will build your android application. I email this file: `ionic-push-starter/platforms/android/build/outputs/apk/android-debug.apk` to myself and download my app on my phone. `.apk` is the filetype of your android app. Ionic converts your html, css and js into native code.

Email the above file to yourself. You may have to disable some security settings on your android phone.

Note:
```
$ ionic emulate android
$ ionic serve
```

Are the ways to test in the emulator and browser. We don't have push notification functionality there. To use the android emulator you must have the android sdk installed. Push notifications must be tested on a physical device. Open the app, go to the push tab, you'll see Ionic's note on that.

#### set up Google Cloud Messaging 

This is all pretty much from the [docs](http://docs.ionic.io/v1.0/docs/push-android-setup).

Click "Create Project" on [google developer console](https://console.developers.google.com/project).

Copy in the top bar your **Project ID** and **Project Number**. The Project Number will later become your **GCM sender ID**. 

Turn on **Google Cloud Messaging for Android** under Mobile APIs.

![](/content/images/2015/06/Screen-Shot-2015-06-12-at-6-54-09-PM.png)

Then click "Enable API". It is taking a bit of time for me

![](/content/images/2015/06/Screen-Shot-2015-06-12-at-6-54-33-PM.png)

Then create API Public Access Key. Click **Create new Key**.

![](/content/images/2015/06/Screen-Shot-2015-06-12-at-7-04-28-PM.png)

From the ionic docs

>In the Create a new key dialog, click Server key.

> In the resulting configuration dialog leave the box blank to allow connections from any IP

Save the **API Key** that was generated.

Send the google key to the ionic push service. They will be taking care of this for us.

```
$ ionic push --google-api-key your-google-api-key
Google API Key Saved
```

#### Back to the code
In `www/js/app.js` go into the `.config` block we copied in earlier. Uncomment the `gcm_id` property and add the **Project Number** that's on the homepage of your google app (click Overview).

Add the plugin:

```
$ ionic plugin add https://github.com/phonegap-build/PushPlugin.git
```

Now email the app to yourself, download it on your phone. Click register and an alert box with the token will appear

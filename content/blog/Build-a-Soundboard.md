---
layout: post
title: Play recorded sounds with Sound.js
date: "2015-10-23"
category: Javascript
lede: "This tutorial builds an application that plays a sound when you click a button. Built this during RocketU coding bootcamp"
author: Connor Leech
published: true
---

We're going to have a button and when we click it we're going to play a sound.

[Source code](https://github.com/connor11528/soundboard) on github

[Demo](http://connorleech.info/soundboard/) hosted with gh-pages

###### Make sound button

<b>index.html</b>
```
<div class='row text-center' style='margin-top: 50px;'>
    <div id="1" class="gridBox btn btn-lg btn-primary">Hey everybody</div>
</div>
```

I'm using [sound.js](http://createjs.com/SoundJS) and for starters using code provided from their website, slightly modified for styling

<b>main.js</b>
```
$(document).ready(function(){
	var preload;

	function init() {
		if (!createjs.Sound.initializeDefaultPlugins()) {
			return;
		}

		var assetsPath = "";
		var sounds = [
			{src: "hey_everybody.m4a", id: 1}
		];

		createjs.Sound.alternateExtensions = ["mp3"];	// add other extensions to try loading if the src file extension is not supported
		createjs.Sound.addEventListener("fileload", createjs.proxy(soundLoaded, this)); // add an event listener for when load is completed
		createjs.Sound.registerSounds(sounds, assetsPath);
	}

	function soundLoaded(event) {
		//examples.hideDistractor();
		var div = document.getElementById(event.id);
	}

	function stop() {
		if (preload != null) {
			preload.close();
		}
		createjs.Sound.stop();
	}

	function playSound(target) {
		//Play the sound: play (src, interrupt, delay, offset, loop, volume, pan)
		var instance = createjs.Sound.play(target.id, createjs.Sound.INTERRUPT_NONE, 0, 0, false, 1);
		if (instance == null || instance.playState == createjs.Sound.PLAY_FAILED) {
			return;
		}
		instance.addEventListener("complete", function (instance) {

		});
	}

	init();
	$('.gridBox').on('click', function(){
		playSound(this);
		setTimeout(function(){
			stop()
		}, 2000);
	});

});
```

At the end of the function we call init and listen for clicks. Stop the sound after 2 seconds.
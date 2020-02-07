---
layout: post
title: Handle a git merge conflict
date: "2015-10-20"
category: Programming
description: "At the beginning of the RocketU we learned how to fix git merge conflicts. Though I had been working with Git for some time i had not worked extensively in teams so handling merge conflicts was new territory for me. Since I hadn't learning about it beofre I decided to write a short tutorial."
author: Connor Leech
published: true
---

I'm starting to work with other people using git and github. Though I'm familiar with pushing, pulling, branching and cloning, I had not regularly encountered  merge conflicts until I began RocketU.

```
mkdir mergeConflicting
cd mergeConflicting
touch play.js
```

add some random code. This creates a gulp web server (after you run npm install):

```
var gulp = require('gulp'),
  connect = require('gulp-connect');

gulp.task('webserver', function() {
  connect.server({
    livereload: true
  });
});
 
gulp.task('default', ['webserver']);
```

Then add the files to .git and commit the changes:

```
git add -A
git commit -m 'add gulp process'
```

All of this is standard. I've learned not to work on the master branch. So to create a new branch named develop and start working on it run:

```
git checkout -b develop
```

Now we're on the develop branch and we'll make some changes. We'll make some changes to master. When we try to merge develop into master we will have a merge conflict that we will resolve.


I added some more random tasks from the gulp API docs that changes lines 5 through 11.
```
git add -A
git commit -m 'add gulp task on develop branch'
```

Now I'll go back to master, add some different code on the same lines and simulate a merge conflict.

```
git checkout master
```
Add some new stuff
```
git add -A
git commit -m 'make changes on master branch'
git status
On branch master
nothing to commit, working directory clean
```

Cool, so everything is gravy on our respective branches. When I try to merge develop into master though we'll get a conflict! Git does not know if it should display the file as I have it on master or as I have it on develop. We changed the same lines!

```
git merge develop
Auto-merging play.js
CONFLICT (content): Merge conflict in play.js
Automatic merge failed; fix conflicts and then commit the result.
```

So that's fun. If you look at the file it is a bit scary because your code is all messy and jacked up. Git will display <<<<< and >>>>> denoting where the two branches conflict.

```
<<<<<<< HEAD
// random changes on master branch
gulp.src('./client/templates/*.jade')
  .pipe(jade())
  .pipe(gulp.dest('./build/templates'))
  .pipe(minify())
  .pipe(gulp.dest('./build/minified_templates'));

=======

gulp.src('client/js/**/*.js') // Matches 'client/js/somedir/somefile.js' and resolves `base` to `client/js/`
  .pipe(minify())
  .pipe(gulp.dest('build'));  // Writes 'build/somedir/somefile.js'

gulp.src('client/js/**/*.js', { base: 'client' })
  .pipe(minify())
  .pipe(gulp.dest('build'));  // Writes 'build/js/somedir/somefile.js'
  
>>>>>>> develop
```

Until recently I would edit this file manually and commit my changes. Today we'll use the git mergetool tool.

```
git mergetool

This message is displayed because 'merge.tool' is not configured.
See 'git mergetool --tool-help' or 'git help config' for more details.
'git mergetool' will now attempt to use one of the following tools:
opendiff tortoisemerge emerge vimdiff
Merging:
play.js

Normal merge conflict for 'play.js':
  {local}: modified file
  {remote}: modified file
Hit return to start merge resolution tool (opendiff):
```

I use opendiff cause it gives you a GUI. Another tool is called vimdiff. I always get stuck in Vim and don't like using it cause I'm afraid. Use opendiff. Hit enter.

![](/content/images/2015/04/Screen-Shot-2015-04-23-at-8-22-13-PM.png)

Now you will see the two files and how they differ. Use the arrow keys to select left or right. I will select the right. Then save (File > Save or command + S) then quit (File > Quit or command + Q). Back in the terminal you'll see some stuff.

```
git mergetool

This message is displayed because 'merge.tool' is not configured.
See 'git mergetool --tool-help' or 'git help config' for more details.
'git mergetool' will now attempt to use one of the following tools:
opendiff tortoisemerge emerge vimdiff
Merging:
play.js

Normal merge conflict for 'play.js':
  {local}: modified file
  {remote}: modified file
Hit return to start merge resolution tool (opendiff): 
2015-04-23 20:21:49.350 FileMerge[3018:303] Invalid color System, labelColor (warning given only once)
2015-04-23 20:21:49.516 FileMerge[3018:303] Unable to load platform at path /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform
2015-04-23 20:21:49.518 FileMerge[3018:303] Unable to load platform at path /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform

git status

On branch master
All conflicts fixed but you are still merging.
  (use "git commit" to conclude merge)

Changes to be committed:

	modified:   play.js

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	play.js.orig
```

Congrats you have resolved the merge conflict. We still have to commit our changes. Also notice a new (untracked) file was created. This is the same file when the merge conflict was first created, so you have both versions of the code. If multiple files conflict multiple .orig files will be generated. Delete this file and make the commit of the resolved merge.

```
rm play.js.orig
```
I made a comment to play.js. Then I committed the changes

```
git add -A
git commit -m 'merge the two branches'
```

Finally we can delete the develop branch and check that it is gone!

```
git branch -d develop 
Deleted branch develop (was fca53f8).

git branch
* master
```

Ultimately the develop commit shows up on the commit history of our master branch. There is another way to do this using `git rebase`. I'll save that fun for later

Check out the full git history here:

https://github.com/connor11528/mergeConflicting/commits/master
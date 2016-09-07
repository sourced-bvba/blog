---
layout: post
category : "java"
title: "Why I never use the Maven release plugin"
tags : [maven]
---

Just about every 6 months or so an article appears cursing Maven, attracting both proponents as opponents to Maven and Ant. While it’s real fun to watch (I really get a laugh when people start to advocate the return to Ant), most of the time it’s always the same arguments. Maven lacks flexibility, the plugin system sucks (when will people learn to use plugin versions…), you can’t use scripting and the all time favorite: the release plugin sucks. Well, I am a Maven addict and I’m happy to say: yes, I agree, the release plugin sucks. Big time. But here’s something you may have forgotten: you don’t need it! Even more: you shouldn’t use it.

The Maven release plugin tries to make releasing software a breeze. That’s where the plugin authors got it wrong to start with. Releases are not something done on a whim. They are carefully planned and orchestrated actions, preceded by countless rules and followed by more rules. Assuming you can bundle all that in a simple mvn release:release is just plain naive. Even Maven’s most fierce supporters agree on this. The Maven release plugin just tries to do too much stuff at once: build your software, tag it, build it again, deploy it, build the site (triggering yet another build in the process) and deploy the site. And whilst doing that, running the tests x times. Most of the time, you’re making candidate releases, so building the complete documentation is a complete waste of time.

Now, if you break down the release plugin into sensible steps, you’ll really save yourself a whole lot of trouble. I use these steps to release something. As a sidenote: I use git and git-flow standards (as described here). Assume the POM’s version’s currently on 1.0-SNAPSHOT.

1. Announce the release process
   Very important. As I said, you don’t release on a whim. Make sure everyone on your team knows a release is pending and has all their stuff pushed to the  development branch that needs to be included.

2. Branch the development branch into a release branch.
   Following git-flow rules, I make a release branch 1.0.

3. Update the POM version of the development branch.
   Update the version to the next release version. For example `mvn versions:set -DnewVersion=2.0-SNAPSHOT`. Commit and push. Now you can put resources developing towards the next release version.

4. Update the POM version of the release branch.
   Update the version to the standard CR version. For example `mvn versions:set -DnewVersion=1.0.CR-SNAPSHOT`. Commit and push.

5. Run tests on the release branch.
   Run all the tests. If one or more fail, fix them first.

6. Create a candidate release from the release branch.
   * Use the Maven version plugin to update your POM’s versions. For example `mvn versions:set -DnewVersion=1.0.CR1`. Commit and push.
   * Make a tag on git.
   * Use the Maven version plugin to update your POM’s versions back to the standard CR version. For example `mvn versions:set -DnewVersion=1.0.CR-SNAPSHOT`.    
   * Commit and push.
   * Checkout the new tag.
   * Do a deployment build (mvn clean deploy). Since you’ve just run your tests and fixed any failing ones, this shouldn’t fail.
   * Put deployment on QA environment.


7. Iterate until QA gives a green light on the candidate release.
   1. Fix bugs.
      Fix bugs reported on the CR releases on the release branch. Merge into development branch on regular intervals (or even better, continuous). Run tests continuously, making bug reports on failures and fixing them as you go.

   2. Create a candidate release.
      * Use the Maven version plugin to update your POM’s versions. For example `mvn versions:set -DnewVersion=1.0.CRx`. Commit and push.
      * Make a tag on git.
      * Use the Maven version plugin to update your POM’s versions back to the standard CR version. For example `mvn versions:set -DnewVersion=1.0.CR-SNAPSHOT`.    
      * Commit and push.
      * Checkout the new tag.
      * Do a deployment build (mvn clean deploy). Since you’ve run your tests continuously, this shouldn’t fail.
      * Put deployment on QA environment.


8. Once QA has signed off on the release, create a final release.
   * Check whether there are no new commits since the last release tag (if there are, slap developers as they have done stuff that wasn’t needed or asked for).
   * Use the Maven version plugin to update your POM’s versions. For example `mvn versions:set -DnewVersion=1.0`. Commit and push.
   * Tag the release branch.
   * Merge into the master branch.
   * Checkout the master branch.
   * Do a deployment build (mvn clean deploy).
   * Start production release and deployment process (in most companies, not a small feat). This can involve building the site and doing other stuff, some not even Maven related.

There’s no way in hell Maven can automate this process and if you try, you’ll bump into the many pitfalls the release plugin has to offer. The release plugin is just a combination of the versions, scm, deploy and site plugin that seriously violates the single responsibility principle. The release plugin is one of the reasons Maven has gotten a bad reputation with some people. It’s long due for an overhaul, but if you ask me, they should just remove it altogether. Releasing software is a process, not a single command on the command line.

The process I just described isn’t perfect in any way, but it works and I avoid using the release plugin as it just does too much stuff. Have fun bashing Maven, but please, keep it clean :) .
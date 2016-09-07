---
layout: post
category : article
title: "Using Animal Sniffer with Gradle"
comments: true
tags : [java]
---

When compiling code in Gradle that targets older JVM's, you need to be very careful your code doesn't use any of the classes in the newer JDK. For example, it's very easy to create a class that uses the new DateTime API in Java 8 and compile it to target Java 6. The compilation will succeed but the class won't run on a Java 6 JVM. But the changes can be harder to spot. For example, the Collection API has changed a bit in Java 8 and some classes have added methods. The result would be the same.<!--more--> 

Luckily there is a tool called Animal Sniffer that checks your code against a signature. If the classes in the signature don't match those in your code, Animal Sniffer will spit out warnings. For example, there is a signature JAR for Java 6 and if you would run Animal Sniffer on code that doesn't match Java 6 API's, it would tell you. 

Unfortunately, Animal Sniffer doesn't have any Gradle support yet. Most projects that use Animal Sniffer with Gradle use the Ant targets. I'm not a big fan of using Ant legacy in Gradle build file, so I did the only sane thing: write a plugin for Gradle. The build file for the plugin is very simple, it just adds the animal sniffer dependency.

{% highlight groovy %}
apply plugin: 'groovy'

repositories {
    jcenter()
}

dependencies {
    compile "org.codehaus.mojo:animal-sniffer:1.11"
    compile localGroovy()
    compile gradleApi()
}
{% endhighlight %}

As for the plugin, I used the functionality of the Animal Sniffer Maven plugin as a template. Currently, it should provide the same functionality as the Maven plugin, only a bit groovier. It's still a bit rough around the edges, but it does what it needs to do.

{% highlight groovy %}
import groovy.transform.Canonical
import org.gradle.api.Plugin
import org.gradle.api.Project

import org.codehaus.mojo.animal_sniffer.ClassListBuilder
import org.codehaus.mojo.animal_sniffer.SignatureChecker
import org.codehaus.mojo.animal_sniffer.logging.Logger
import org.slf4j.LoggerFactory

class AnimalSnifferPlugin implements Plugin<Project> {
    private org.slf4j.Logger logger = LoggerFactory.getLogger(this.class)

    @Override
    void apply(Project project) {
        project.configurations.maybeCreate("signature")
        project.extensions.create("animalsniffer", AnimalSnifferExtension)
        if(project.plugins.findPlugin("java")) {
            project.tasks.getByName("compileJava").doLast {
                performAnimalSniffer(project)
            }
        } else {
            logger.warn("Animal Sniffer plugin applied, but Java plugin is not applied. Skipping.")
        }
    }

    def performAnimalSniffer(Project project) {
        if (!project.animalsniffer.skip) {
            if (!project.animalsniffer.signature) {
                throw new IllegalStateException("Signature is required when using Animal Sniffer plugin")
            } else {
                project.dependencies {
                    signature project.animalsniffer.signature
                }
            }
            def logger = new GradleLogger(logger)
            def signatures = project.configurations.signature.resolvedConfiguration.resolvedArtifacts*.file
            ClassListBuilder plb = new ClassListBuilder(logger);
            plb.process(project.buildDir);
            if (project.animalsniffer.excludeDependencies) {
                def files = project.configurations.runtime.resolvedConfiguration.resolvedArtifacts*.file
                files.each {
                    plb.process(it)
                }
            }
            def ignored = plb.getPackages();
            if (project.animalsniffer.ignores) {
                project.animalsniffer.ignores.each {
                    if (it) {
                        ignored.add(it.replace('.', '/'))
                    }
                }
            }
            signatures.each {
                def checker = new SignatureChecker(it.newInputStream(), ignored, logger)
                if (project.animalsniffer.annotations) {
                    checker.setAnnotationTypes(project.animalsniffer.annotations)
                }
                def allSources = project.sourceSets*.allJava.collect { it.srcDirs }.flatten()
                checker.sourcePath = allSources
                if (project.buildDir)
                    checker?.process(project.buildDir)
                if (checker.signatureBroken)
                    throw new IllegalStateException("Signature errors found. Verify them and ignore them with the proper annotation if needed.")
            }
        }
    }
}


@Canonical
class GradleLogger implements Logger {
    @Delegate
    org.slf4j.Logger logger
}

class AnimalSnifferExtension {
    boolean excludeDependencies = true
    String signature = ""
    def ignores = []
    def annotations = []
    boolean skip = false
}
{% endhighlight %}

Using the plugin is dead easy. The plugin was configured locally with the plugin-id `animalsniffer`. After including the jar in the buildscript classpath, configuring the plugin is just a matter of applying the plugin and configuring the signature. To configure the signature just set the `signature` property of the `animalsniffer` extension in your gradle file to the dependency definition of the signature. Currently you can find signatures for all Java JVM older than 8 (1.2 to 7) in the group `org.codehaus.mojo.signature`. For example, to configure for the standard Java 6 API's, you can use the following configuration in your gradle build file.

{% highlight groovy %}
apply plugin: "animalsniffer"
animalsniffer {
    signature = "org.codehaus.mojo.signature:java16:1.0@signature"
}
{% endhighlight %}

Once again, writing a Gradle plugin is proving to be dead easy. What could be improved? Perhaps use some mapped values for signature (so you can use `signature = "1.6"` or `signature = "1.6-jrockit"`) which would make configuration a wee bit easier. But that's just syntax sugar.

UPDATE:

Just added the plugin to my Bintray account. Just add to following to your Gradle build file to use the plugin.

{% highlight groovy %}
buildscript {
    repositories {
        maven {
            url = "http://dl.bintray.com/lievendoclo/maven"
        }
    }
    dependencies {
        classpath "be.insaneprogramming.gradle:animalsniffer-gradle-plugin:1.0.0"
    }
}
{% endhighlight %}

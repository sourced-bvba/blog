---
layout: post
title: "The hell of IntelliJ and Gradle integration"
date: 2013-08-07 15:02
comments: true
categories: 
---

I love using Gradle nowadays. I love IntelliJ even more. However, integration these two is proving to be a real pain in the ass! 

For simple Java projects, everything works great. Although I have to admit I'm staying clear of the JetGradle plugin, which is at best somewhat usable. But most of the time, I'm still using the Gradle IDEA plugin. It just works better and I'm used to having a console open at all times: it's just as much work for me to type `gradle idea` than it is to synchronize a Gradle project in IDEA.

But...

The WAR integration is just awful, no, even abysmal. Both the Gradle IDEA plugin as JetGradle lack the functionality to add a web facet to my module. The Gradle guys aren't really inclined to add the functionality to their plugin, and the guys at Jetbrains seems to be blissfully unaware that an issue has been filed more than a year ago (I created another one myself to wake them up).<!--more-->

Having no automatic way to synchronize my gradle build file with my IDE for web projects is forcing me to use Maven for these projects, or to use Eclipse as my IDEA (the eclipse-wtp plugin works). I'm not happy with either of those options. 

The proposed solutions aren't great either. You can manually add the web facet to your IDEA project. This is error-prone and not something I'm looking forward to doing every time I change a dependency. The other solution is even worse and involves manipulating the IntelliJ XML project files from within the Gradle build script. How time-consuming the first solution may be, this is just plain nuts.

I'm really sad. Gradle was becoming my premier choice for building software. But unless they can provide decent integration with web projects, I'm forced to use Maven for any projects that need a war built. Gradle, don't make me regret switching over to you. 

*Update*: Thank God Gradle uses Groovy and has great XML editing possibilities. So I went for the solution that seemed nuts: manipulating the iml and ipr files. It took a bit of tinkering, but the following snippet will add the web facet to your project AND create a deployable artifact (exploded WAR) in IntelliJ. It's not pretty, but it works (for me at least, there are probably a couple of use cases that won't work).
I just wished the guys from Gradle would have done this in the first place, but maybe my code will make its way into Gradle. One can only hope.

First you have a class in your `buildSrc\src\main\groovy` folder that has all the functionality for the enrichment process (as I call it).

{% highlight groovy %}
import org.gradle.api.Project;

public class IdeaEnricher {

    def static updateWebArtifacts(Project gradleProject, Node project) {
        def artifactManager = project.component.find { it.@name == 'ArtifactManager' } as Node
        if (artifactManager) {
            Node artifact = artifactManager.artifact.find { it.@type == 'exploded-war' }
            if (artifact) {
                artifactManager.remove(artifact)
            }
        } else {
            artifactManager = project.appendNode('component', [name: 'ArtifactManager']);
        }
        def builder = new NodeBuilder();
        def artifact = builder.artifact(type: 'exploded-war', name: "${gradleProject.name}/Exploded war") {
            'output-path'("\$PROJECT_DIR\$/build/libs/${gradleProject.name}_exploded.war")
            root(id: 'root') {
                element(id: 'javaee-facet-resources', facet: "${gradleProject.name}/web/Web");
                element(id: 'directory', name: 'WEB-INF') {
                    element(id: 'directory', name: 'classes') {
                        element(id: 'module-output', name: "${gradleProject.name}")
                    }
                    element(id: 'directory', name: 'lib') {
                        gradleProject.configurations.runtime.each {
                            element(id: 'file-copy', path: it)
                        }
                    }
                }
            }
        }
        artifactManager.append artifact

    }

    def static updateWebFacet(Project gradleProject, Node module) {
        def facetManager = module.component.find { it.@name == 'FacetManager' } as Node
        if (facetManager) {
            Node webFacet = facetManager.facet.find { it.@type == 'web' }
            if (webFacet) {
                facetManager.remove(webFacet)
            }
        } else {
            facetManager = module.appendNode('component', [name: 'FacetManager']);
        }
        def builder = new NodeBuilder();
        def webFacet = builder.facet(type: "web", name: 'Web') {
            configuration {
                descriptors {
                    deploymentDescriptor(name: 'web.xml', url: 'file://$MODULE_DIR$/src/main/webapp/WEB-INF/web.xml')
                }
                webroots {
                    root(url: 'file://$MODULE_DIR$/src/main/webapp', relative: '/')
                }
                sourceRoots {
                    root(url: 'file://$MODULE_DIR$/src/main/java')
                    root(url: 'file://$MODULE_DIR$/src/main/resources')
                }
            }
        }
        facetManager.append webFacet
    }

    def static updateBuildOutputFolderForGradle(Project gradleProject, Node project) {
        def projectRootManager = project.component.find { it.@name == 'ProjectRootManager' } as Node
        Node output = projectRootManager.output.find{it.@url == 'file://$PROJECT_DIR$/out'}
        if(output) {
            output.@url = 'file://$PROJECT_DIR$/build/ide'
        }
    }
}
{% endhighlight %}  

Enabling it in your gradle build script is quite trivial then:

{% highlight groovy %}
idea {
    module {
        iml {
            withXml {
                IdeaEnricher.updateWebFacet(this.project, it.asNode())
            }
        }
    }
    project {
        ipr {
            withXml {
                IdeaEnricher.updateWebArtifacts(this.project, it.asNode())
                IdeaEnricher.updateBuildOutputFolderForGradle(this.project, it.asNode())
            }
        }
    }
}
{% endhighlight %}

I'm inclined to do a bit more tinkering on the ipr file so that the output folder is in line with gradle's build folder. Now IntelliJ creates its own `out` folder. It's a minor annoyance, but since I'm changing the configuration anyway, I might as well go all the way.

This has once again convinced me that Gradle is the right choice. Granted, it's a bit rough around the edges and not all functionality that you have with Maven is there yet. I once said that Gradle provides your with the tools to blow your entire leg off, but this time I was glad I had the entire Groovy weaponry at my disposal. The fact that you can add on additional logic in your build is something extremely powerful and something I see myself doing a lot more in the future.

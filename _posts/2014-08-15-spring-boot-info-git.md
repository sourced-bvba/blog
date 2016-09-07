---
layout: post
category : article
title: "Spring Boot's info endpoint, Git and Gradle"
comments: true
tags : [spring boot git]
---

I'm a huge fan of Spring Boot. I really like how it has raised my level of productivity and the ease of adoption. I also use Git on a day to day basis. However, the integration of information of your Git repository in Spring Boot is not that straightforward if you're using Gradle. Luckily it's not that much work, as you'll see. <!--more-->.

Spring Boot has a very handy feature called the actuator, which adds a lot of REST endpoints to your application which provide a lot of interesting information. One of those endpoints is the info endpoint. This endpoint return a JSON containing some information on your Spring Boot deployment, such as the application name and version, if you've configured them correctly in the application configuration. One of the other features of that endpoint is that when there is a git.properties file on the root of the deployment, it'll use the values in there to display some git information in that JSON as well.

However, in the documentation it's only documented how you can do this with Maven (it has a plugin for that) and only has a vague pointer to the gradle-git plugin (while mentioning it'll be a bit more work). Now, writing a Gradle plugin that generates that git.properties wasn't that hard at all. 

I haven't had the time to put this code in a standalone plugin, but it shouldn't be that hard to do so.

First, I made a buildSrc folder in my project root so I could write the plugin. Then I added a small build.gradle in there to add a dependency to GrGit, which is a great0 library that integrates Gradle with Git.

{% highlight groovy %}
apply plugin: 'groovy'

repositories {
    jcenter()
}

dependencies {
    compile("org.ajoberstar:gradle-git:0.9.0")
}
{% endhighlight %} 

Then I just had to write the plugin. I wanted to have the generation to run whenever the JavaPlugin's classes task was run, as this should cover most JVM-oriented builds (most require the Java plugin).

{% highlight groovy %}
import org.ajoberstar.grgit.Grgit
import org.gradle.api.DefaultTask
import org.gradle.api.Plugin
import org.gradle.api.Project
import org.gradle.api.Task
import org.gradle.api.plugins.BasePlugin
import org.gradle.api.plugins.JavaPlugin
import org.gradle.api.tasks.TaskAction

class GitPropertiesPlugin implements Plugin<Project> {
    @Override
    void apply(Project project) {
        def task = project.tasks.create('generateGitProperties', GenerateGitPropertiesTask)
        task.setGroup(BasePlugin.BUILD_GROUP)
        ensureTaskRunsOnJavaClassesTask(project, task)
    }

    private void ensureTaskRunsOnJavaClassesTask(Project project, Task task) {
        project.getTasks().getByName(JavaPlugin.CLASSES_TASK_NAME).dependsOn(task)
    }

    static class GenerateGitPropertiesTask extends DefaultTask {
        @TaskAction
        void generate() {
            def repo = Grgit.open(project.file('.'))
            def dir = new File(project.buildDir, "resources/main")
            def file = new File(project.buildDir, "resources/main/git.properties")
            if (!dir.exists()) {
                dir.mkdirs()
            }
            if (!file.exists()) {
                file.createNewFile()
            }
            def map = ["git.branch"                : repo.branch.current.name
                       , "git.commit.id"           : repo.head().id
                       , "git.commit.id.abbrev"    : repo.head().abbreviatedId
                       , "git.commit.user.name"    : repo.head().author.name
                       , "git.commit.user.email"   : repo.head().author.email
                       , "git.commit.message.short": repo.head().shortMessage
                       , "git.commit.message.full" : repo.head().fullMessage
                       , "git.commit.time"         : repo.head().time.toString()]
            def props = new Properties()
            props.putAll(map)
            props.store(file.newWriter(), "")
        }
    }
}
{% endhighlight %} 

Now I only need to add the plugin to the project and it'll be automatically executed when I build the project.

{% highlight groovy %}
apply plugin: GitPropertiesPlugin
{% endhighlight %} 

That really wasn't that hard...

---
layout: post
category : "java"
title: "Exploring Spring Integration 2.1"
tags : [spring]
---

At my current assignment, one of the things that we do is reading XML files from an FTP site, do some magic on the contents and send the modified data to a REST service. If you’ve read some of my earlier posts, you’ve probably guessed this is done through an IBM product (Websphere Process Server). Given the fact that WPS is just an ESB with a BPM engine, I wanted to see how difficult it would be and how much faster I would be if I’d use an open source product for the ESB part (there’s an article coming on the BPM part too). As I’m a real Spring fan, I decided to give Spring Integration (I’ll refer to it as SI from now on) a go, as other products seemed a bit too much work to start with (Mule and ServiceMix have their own server implementation for starters…). <!--more-->

Using my trusty Maven, I made a simple WAR project and added some of the SI dependencies. Why a WAR? That way I can simply deploy my ESB on a Tomcat  . As I’m trying to read files from an FTP, I’ll need the FTP component (which transitively get the file component as well). I also need the XML component for unmarshalling my content. I also included the stream component, as it has a shortcut for writing stuff on the standard Java output stream (I’m trying to write as little code as possible).

{% highlight xml %}
<dependency>
    <groupId>org.springframework.integration</groupId>
    <artifactId>spring-integration-ftp</artifactId>
    <version>2.1.0.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.integration</groupId>
    <artifactId>spring-integration-xml</artifactId>
    <version>2.1.0.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.integration</groupId>
    <artifactId>spring-integration-stream</artifactId>
    <version>2.1.0.RELEASE</version>
</dependency>
{% endhighlight %}

And we’re off. I made a simple Spring applicationContext.xml which is started up by the ContextLoaderListener when my WAR is deployed. All the integration beans will be put in here.

One of the basic concepts of SI is channels. Channels are pipes to which content can be written to by producers and read from by consumers. As with all ESB processes, you’re wise to write out your message flow in order to find out which channels you’ll have. In this case I have three: one containing my ftp files, one containing the content of each of the files and one containing my unmarshalled objects. Channels contain Message objects with various payloads. My channel configuration looks like this:

{% highlight xml %}
<int:channel id="ftpFiles"/>
<int:channel id="fileContent"/>
<int:channel id="xmlObjects"/>
{% endhighlight %}

Step one: reading the files from the FTP server. I have a local FTP server running on my trusty Linux Mint laptop which has complete access to my home folder. I’ve made a test folder which will contain the files I want to read. So what you need to do is to make a ftp inbound adapter which will read the files and put them somewhere locally. Adapters connect channels to external sources. This is the XML configuration:

{% highlight xml %}
<bean id="ftpClientFactory" class="org.springframework.integration.ftp.session.DefaultFtpSessionFactory">
    <property name="host" value="localhost"/>
    <property name="port" value="21"/>
    <property name="username" value="lievendoclo"/>
    <property name="password" value="*******"/> <!-- I'm not stupid, guys -->
</bean>
<int-ftp:inbound-channel-adapter id="ftpInbound"
                                 channel="ftpFiles"
                                 filename-pattern="*.xml"
                                 session-factory="ftpClientFactory"
                                 auto-create-local-directory="true"
                                 delete-remote-files="true"
                                 local-filename-generator-expression="'ftpupload.' + T(java.lang.System).currentTimeMillis() + '.txt'"
                                 remote-directory="/home/lievendoclo/test"
                                 local-directory="/tmp/import">
</int-ftp:inbound-channel-adapter>
{% endhighlight %}

Short explanation: the inbound adapter connects to the FTP by using the ftpClientFactory. It scans for xml files in my test folder and copies them to a local folder (in my tmp folder), renaming them in the process. After that, it send a Message with a File payload to the first channel containing the FTP files. The scanning is done through polling. You can add a poller to the channel adapter, but for the simplicity of this article I added a default poller which is used by all the beans which may require a poller. This poller polls every second, only passing through 5 messages per poll.

{% highlight xml %}
<int:poller id="defaultPoller" default="true" max-messages-per-poll="5" fixed-rate="1000"/>
{% endhighlight %}

Step two: read the file and extract the contents. This is accomplished through a standard SI transformer, which transforms a File message to a String message. In my case, it also deletes the local file in the process. After the transformation, the new message is sent to the second channel, which contains the file content.

{% highlight xml %}
<int-file:file-to-string-transformer id="fileToString" input-channel="ftpFiles" output-channel="fileContent" delete-files="true"/>
{% endhighlight %}

Step three: unmarshal the String into a JAXB2 object graph. In this example, I have XML files which adhere to a certain schema. This allowed me to create (or generate if you want to) JAXB2 classes.

{% highlight xml %}
<bean id="xmlMarshaller" class="org.springframework.oxm.jaxb.Jaxb2Marshaller">
    <property name="classesToBeBound">
        <list value-type="java.lang.Class">
            <value>be.insaneprogramming.examples.SimpleObject</value>
        </list>
    </property>
</bean>
<int-xml:unmarshalling-transformer unmarshaller="xmlMarshaller" input-channel="fileContent" output-channel="xmlObjects" />
{% endhighlight %}

This shouldn’t need much explanation, the unmarshalling transformers changes my String objects into complete object graphs using JAXB2. After the transformation, the objects are sent to the third and final channel.

As a last step, I send the objects to the standard outputstream to see whether the objects has succesfully gone through the ESB.

{% highlight xml %}
<int-stream:stdout-channel-adapter id="stdoutAdapterWithDefaultCharset" channel="xmlObjects"/>
{% endhighlight %}

And that’s it. To test it I started out with an empty test folder and deployed the WAR to my Tomcat. The moment I drop an XML file in the test folder, it’s picked up by SI and output appears in my tomcat log file, containing the toString() output of the object which has be unmarshalled. Nice.

I would never have guessed SI was that easy to use. Looking at the documentation, you can do quite a lot with it, from reading from databases to posting tweets. I’ll definitely play some more with it.

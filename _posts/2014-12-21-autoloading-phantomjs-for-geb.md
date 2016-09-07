---
layout: post
category : article
title: "Autoloading PhantomJS with Geb"
comments: true
tags : [java, testing]
---

When it comes to building integration tests for web applications, nothing beats Geb for me. The concise and readable syntax, together with excellent Spock support, makes testing a web application a joy. For those not acquainted with Geb, it's a Groovy library on top of Selenium that provides a more easier API to interact with the browser from your unit tests. <!--more-->

If you're running your tests on a headless server, it's a bit of a hassle though. Unless you're willing to play with a Selenium grid or do some virtual X magic, you're limited to either using HTMLUnit or PhantomJS as a driver. Using HTMLUnit is not a good choice unless you have a very simple application, since HTMLUnit does not support all the feature modern browsers do. PhantomJS on the other hand is a Webkit-compatible headless browser and supports all of the features a browser like Chrome or Safari does.

When you want to use Geb with PhantomJS, you just need to include the PhantomJS driver and make sure you have PhantomJS installed on your path. Then it's just a matter of configuring Geb to use the PhantomJS driver and you're off. That being said, there is an easier way to use PhantomJS which doesn't even need to have PhantomJS on your path. Since Geb is Groovy-based, you can include some scripting magic to automatically download a version of PhantomJS and configure Geb to use that version. This way you don't need to alter the path of your server.

This approach assumes that you have the ant library on your classpath.

In order to have Geb automatically download and use a version of PhantomJS, add the following to your GebConfig.groovy.

{% highlight groovy %}
String phantomJSVersion = '1.9.2'

String platform
String archiveExtension
String execFilePath

if (Platform.current.is(Platform.WINDOWS)) {
    execFilePath = 'phantomjs.exe'
    platform = 'windows'
    archiveExtension = 'zip'
}
else if (Platform.current.is(Platform.MAC)) {
    execFilePath = '/bin/phantomjs'
    platform = 'macosx'
    archiveExtension = 'zip'
} else if (Platform.current.is(Platform.LINUX)) {
    execFilePath = '/bin/phantomjs'
    platform = 'linux-i686'
    archiveExtension = 'tar.bz2'
} else {
    throw new RuntimeException("Unsupported operating system [${Platform.current}]")
}

String phantomjsExecPath = "phantomjs-${phantomJSVersion}-${platform}/${execFilePath}"

String phantomJsFullDownloadPath = "https://phantomjs.googlecode.com/files/phantomjs-${phantomJSVersion}-${platform}.${archiveExtension}"

File phantomJSDriverLocalFile = downloadDriver(phantomJsFullDownloadPath, phantomjsExecPath, archiveExtension)

System.setProperty('phantomjs.binary.path', phantomJSDriverLocalFile.absolutePath)

driver = {
    Capabilities caps = DesiredCapabilities.phantomjs()
    def phantomJsDriver = new PhantomJSDriver(PhantomJSDriverService.createDefaultService(caps), caps)
    phantomJsDriver.manage().window().setSize(new Dimension(1028, 768))

    return phantomJsDriver
}

private File downloadDriver(String driverDownloadFullPath, String driverFilePath, String archiveFileExtension) {
    File destinationDirectory = new File("target/drivers")
    if (!destinationDirectory.exists()) {
        destinationDirectory.mkdirs()
    }

    File driverFile = new File("${destinationDirectory.absolutePath}/${driverFilePath}")

    String localArchivePath = "target/driver.${archiveFileExtension}"

    if (!driverFile.exists()) {
        def ant = new AntBuilder()
        ant.get(src: driverDownloadFullPath, dest: localArchivePath)

        if (archiveFileExtension == "zip") {
            ant.unzip(src: localArchivePath, dest: destinationDirectory)
        } else {
            ant.untar(src: localArchivePath, dest: destinationDirectory, compression: 'bzip2')
        }

        ant.delete(file: localArchivePath)
        ant.chmod(file: driverFile, perm: '700')
    }

    return driverFile
}
{% endhighlight %}

This script will detect on which platform you're trying to run the tests and download the correct PhantomJS. 

Enjoy testing your web applications!

---
layout: post
title: "REST prototyping with Spark and Groovy"
date: 2013-12-17 22:00
comments: true
categories:
---

Spark is rapidly becoming one of my favorite web application prototyping frameworks. Setting up a quick project is extremely easy and plugging in simple templating engines is even easier. It's being used frequently for teaching other frameworks in order to provide a quick web front-end for that framework (for example, the MongoDB courses use Spark).

But I've also used Spark for something else. It's extremely easy to make a REST prototype with Spark. This way you can make an easy system that can 'mock' a future REST backend, but is still adaptable.<!--more-->

The concept is simple: create a text file mapping URLs to JSON files and use Spark to serve those files through the specified URLs. I'm using Groovy here, but it's just as simple to do this with Java.

``` java
class RestPrototype {
    public static void main(String[] args) {
        def file = RestPrototype.class.getResource("mapping.txt")
        file.eachLine {
            def values = it.split(" ")
            get(new Route(values[0]) {
                def handle(Request request, Response response) {
                    def content = RestPrototype.class.getResource(values[1])
                    def text = content.text
                    response.type("application/json")
                    text
                }
            })
        }
    }
}
```

The mapping file is a simple text file that has the url path and the file location, one per line, separated by a whitespace

``` java
hello/world data/helloworld.json
```

In this case, the response files are looked up on the classpath, so the application is expecting a `helloworld.json` file in the `data` folder on the classpath. The content can be any JSON.

``` json
{
    "value": "hello world"
}
```

If you now start the application and go to `http://localhost:4567/hello/world`, you'll get the JSON returned.

This is a really simple way to provide initial JSON files for a REST interface you're still working on. You could even use a database table and a file share so that you could add URLs at runtime (you'll need to create some sort of reloading mechanism in that case).

**Update**:

Implementing a dynamic solution was so easy I decided to put it up as well. I also added support for other HTTP methods.

You create a basic table (mapping) containing 3 columns, path, location and method (all varchar), that contains the same as your mapping file. I'm using MySQL with the MariaDB driver (because that one isn't GPL). The application class now loads the mapping and also contains a basic path for reloading the mapping (so whenever you add new records, you just need to call an URL). Mind you that the file locations now need to be valid URLs instead of classpath locations.

``` java
class RestPrototype {
    public static void main(String[] args) {
       loadResources();
        get(new Route("reload") {
            def handle(Request request, Response response) {
                loadResources()
                "OK"
            }
        })
    }

    static loadResources(){
        def sql = Sql.newInstance("jdbc:mysql://localhost/restprototype", "root", "", "org.mariadb.jdbc.Driver")
        sql.eachRow("select * from mapping") {
            final path = it.path as String
            final location = it.location as String
            final method = it.method as String
            switch(method) {
                case 'GET':
                    get(new Route(path) {
                        def handle(Request request, Response response) {
                            def content = new URL(location)
                            def text = content.text
                            response.type("application/json")
                            text
                        }
                    })
                    break
                case 'PUT':
                    put(new Route(path) {
                        def handle(Request request, Response response) {
                            def content = new URL(location)
                            def text = content.text
                            response.type("application/json")
                            text
                        }
                    })
                    break
                case 'POST':
                    post(new Route(path) {
                        def handle(Request request, Response response) {
                            def content = new URL(location)
                            def text = content.text
                            response.type("application/json")
                            text
                        }
                    })
                    break
                case 'DELETE':
                    delete(new Route(path) {
                        def handle(Request request, Response response) {
                            def content = new URL(location)
                            def text = content.text
                            response.type("application/json")
                            text
                        }
                    })
                case 'OPTIONS':
                    options(new Route(path) {
                        def handle(Request request, Response response) {
                            def content = new URL(location)
                            def text = content.text
                            response.type("application/json")
                            text
                        }
                    })
                case 'HEAD':
                    head(new Route(path) {
                        def handle(Request request, Response response) {
                            def content = new URL(location)
                            def text = content.text
                            response.type("application/json")
                            text
                        }
                    })
            }
            
        }
    }
}
```

So in order to get the same mapping as above, just execute the following SQL (assuming the helloworld.json file is on the root of your D drive).

``` sql
INSERT INTO mapping VALUES ('hello/world', 'file:///D:/helloworld.json', 'GET');
```

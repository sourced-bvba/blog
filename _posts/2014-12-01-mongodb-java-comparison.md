---
layout: post
category : article
title: "A simple use-case comparison of JVM libraries for MongoDB"
comments: true
tags : [java]
---

MongoDB is one of my favorite data stores when it comes to storing document-based JSON data. Communicating with MongoDB with JVM languages can be done in a lot of ways. I thought it would be a nice exercise to take 4 of the most popular solutions and implement a simple use case in each of those solutions. The use case: create a REST service that can get a list of cities and get the nearest city for a given city with the distance to that city.

The 4 approaches I'll compare are using the standard MongoDB Java Driver, Jongo, Morphia and finally Spring Data for MongoDB. All the code is written in Groovy for brevity and I'll be using Spring Boot to minimize boilerplate code to provide the REST layer.<!--more--> 

### The foundation

The Spring Boot app is as simple as it can be:

``` groovy
import org.springframework.boot.SpringApplication
import org.springframework.boot.autoconfigure.EnableAutoConfiguration
import org.springframework.context.annotation.ComponentScan
import org.springframework.context.annotation.Configuration

@EnableAutoConfiguration
@ComponentScan
@Configuration
class MongoComparison
{
    static void main(String[] args) {
        SpringApplication.run(MongoComparison, args);
    }
}
```

For those interested, I'll also provide the Gradle build file used for the comparison.

``` groovy
buildscript {
    repositories {
        jcenter()
        maven {
            url 'http://repo.spring.io/milestone'
        }
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.1.9.RELEASE")
    }
}

apply plugin: 'groovy'
apply plugin: 'spring-boot'

repositories {
    jcenter()
    maven { url 'http://repo.spring.io/milestone' }
    maven { url 'http://www.allanbank.com/repo/' }
}

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
    compile("org.springframework.boot:spring-boot-starter-data-mongodb")
    compile("org.jongo:jongo:1.1")
    compile("org.mongodb.morphia:morphia:0.108")
    compile("de.grundid.opendatalab:geojson-jackson:1.2")
    compile("org.codehaus.groovy:groovy-all:2.3.6")
 }

task wrapper(type: Wrapper) {
    gradleVersion = '2.1'
}
```

Since I'm using Spring Boot with the Spring Data MongoDB support library, there is some autoconfiguration possible. For example, Spring Boot provides a MongoClient and MongoTemplate bean automatically in your Spring Boot's application context. You do need to add some configuration in your application properties (I'm using the YAML style configuration).

``` yaml
spring:
    groovy:
        template:
            check-template-location: false
    data:
        mongodb:
            host: "localhost"
            database: "citydbdata"
```

With the foundation in place, we can start to compare.

### The MongoDB Java Driver

All of the solutions are built upon the foundation of the Java driver supplied by MongoDB themselves, so I thought it would be fitting to start off with it. The driver is the most low-level approach you can take to access a MongoDB database from the JVM. This also means it's a bit more verbose and the API is not as user-friendly as the other alternatives.
However, there's nothing you can't do with the Java driver. The driver is automatically provided in the gradle build by the Spring Data MongoDB support, if you want to use it separately you'll need to include the dependency.

This is the implementation for the MongoDB Java Driver:

``` groovy
import com.mongodb.*
import org.bson.types.ObjectId
import org.geojson.Point
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.http.*
import org.springframework.web.bind.annotation.*
import javax.annotation.PostConstruct
import static org.springframework.web.bind.annotation.RequestMethod.GET

@RestController
@RequestMapping("/mongoclient")
class CityControllerMongoClient {

    final DB db

    def dbObjectToCityTransformer = { DBObject it ->
        def objectMap = it.toMap()
        return new City(_id: objectMap._id, name: objectMap.name, location: new Point(objectMap.location.coordinates[0], objectMap.location.coordinates[1]))
    }

    @Autowired
    CityControllerMongoClient(MongoClient mongoClient) {
        db = mongoClient.getDB("citydbmongoclient")
    }

    @RequestMapping(value="/", method = GET)
    List<City> index() {
        return db.getCollection("city").find().collect(dbObjectToCityTransformer)
    }

    @RequestMapping(value="/near/{cityName}", method = GET)
    ResponseEntity nearCity(@PathVariable String cityName) {
        def city = dbObjectToCityTransformer(db.getCollection("city").findOne(new BasicDBObject("name", cityName)))
        if(city) {
            def point = new BasicDBObject([type: "Point", coordinates: [city.location.coordinates.longitude, city.location.coordinates.latitude]])
            def geoNearCommand =  new BasicDBObject([geoNear: "city", spherical: true, near: point])
            def closestCities = db.command(geoNearCommand).toMap()
            def closest = closestCities.results[1]
            return new ResponseEntity([name:closest.obj.name, distance:closest.dis/1000], HttpStatus.OK)
        }
        else {
            return new ResponseEntity(HttpStatus.NOT_FOUND)
        }
    }

    @PostConstruct
    void populateCities() {
        db.getCollection("city").drop()
        [new City(name: "London",
                location: new Point(-0.125487, 51.508515)),
         new City(name: "Paris",
                 location: new Point(2.352222, 48.856614)),
         new City(name: "New York",
                 location: new Point(-74.005973, 40.714353)),
         new City(name: "San Francisco",
                 location: new Point(-122.419416, 37.774929))].each {
            DBObject location = new BasicDBObject([type: "Point", coordinates: [it.location.coordinates.longitude, it.location.coordinates.latitude]])
            DBObject city = new BasicDBObject([name: it.name, location: location])
            db.getCollection("city").insert(city)
        }
        db.getCollection("city").createIndex(new BasicDBObject("location", "2dsphere"))
    }

    static class City {
        ObjectId _id
        String name
        Point location
    }
}
```

The Java drivers revolves entirely around DBObject objects and you'll need to constantly provide mappings between your domain objects and DBObject instances. The MongoDB Java Driver does not provide any form of object mapping. Luckily, the DBObject structure is very map-like and with the Groovy map support with its terse notation style make it a bit less of a pain.
For the geoNear command, which you need to use to find the nearest city and the distance to that city, you'll probably need to have a look at the MongoDB manual to find out the exact syntax. In short the syntax is

``` json
{
   geoNear: collectionName,
   near: { type: "Point" , coordinates: [ longitude, latitude ] } ,
   spherical: true
}
```

A geoNear command returns the nearest objects in the collection and also provides a field that indicates the distance, which is in meters by default. The format of the near can be either what is shown above or the legacy way of an array of 2 doubles. The former way is now recommended as it adheres to the GeoJSON specification. In all my examples I'm trying to use the GeoJSON notation to store geolocation data where possible. As you can see I'm using a Java library that provides classes for all GeoJSON types.

Aside from all the DBObject to domain object conversions, the code is quite easy to read. You do need to know the details on MongoDB querying, but when you do, the standard MongoDB Java Driver is a very powerful tool.

### Jongo

Jongo is a framework that allows you to interact with a MongoDB instance in a way that is very similar to interacting with it through the Mongo shell as it supports String based interaction and querying (so you don't need to create a DBObject for querying). It also provides in object mapping by utilizing Jackson so you don't need to convert query results or data you want to insert to DBObject instances. The GeoJSON library I'm using has built-in support for Jackson, so we don't need to do anything special for this.

This is the implementation for Jongo:

``` groovy
import com.fasterxml.jackson.databind.ObjectMapper
import com.mongodb.MongoClient
import org.bson.types.ObjectId
import org.geojson.Point
import org.jongo.Jongo
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.http.*
import org.springframework.web.bind.annotation.*

import javax.annotation.PostConstruct

import static org.springframework.web.bind.annotation.RequestMethod.GET

@RestController
@RequestMapping("/jongo")
class CityControllerJongo {

    final Jongo jongo

    @Autowired
    CityControllerJongo(MongoClient mongoClient) {
        jongo = new Jongo(mongoClient.getDB("citydbjongo"))
    }

    @RequestMapping(value="/", method = GET)
    List<City> index() {
        return jongo.getCollection("city").find().as(City).asList()
    }

    @RequestMapping(value="/near/{cityName}", method = GET)
    ResponseEntity nearCity(@PathVariable String cityName) {
        def city = jongo.getCollection("city").findOne("{name:'$cityName'}").as(City)
        if(city) {
            def command = """{
                geoNear: "city",
                near: ${new ObjectMapper().writeValueAsString(city.location)},
                spherical: true
            }"""
            def closestCities = jongo.runCommand(command).as(GeoNearResult) as GeoNearResult<City>
            def closest = closestCities.results[1]
            return new ResponseEntity([name:closest.obj.name, distance:closest.dis/1000], HttpStatus.OK)
        }
        else {
            return new ResponseEntity(HttpStatus.NOT_FOUND)
        }
    }

    @PostConstruct
    void populateCities() {
        jongo.getCollection("city").drop()
        [ new City( name:"London",
                location: new Point(-0.125487, 51.508515)),
          new City( name:"Paris",
                  location: new Point(2.352222, 48.856614)),
          new City( name:"New York",
                  location: new Point(-74.005973, 40.714353)),
          new City( name:"San Francisco",
                  location: new Point(-122.419416, 37.774929)) ].each {
            jongo.getCollection("city").save(it)
        }
        jongo.getCollection("city").ensureIndex("{location:'2dsphere'}")
    }

    static class GeoNearResult<O> {
        List<GeoNearItem<O>> results
    }

    static class GeoNearItem<O> {
        Double dis
        O obj
    }

    static class City {
        ObjectId _id
        String name
        Point location
    }
}
```

As you can see, Jongo is a lot more String based, especially for the geoNear query. However, thanks to the automatic mapping by Jackson, all the conversion code for the query and the insert can be omitted. 

Jongo is handy when you're starting with MongoDB, know the shell commands well and don't want to do manual mapping. You will however need to know the exact syntax of the MongoDB shell API. You also have no code completion on the queries or the commands, but if you're willing to live with this, it's a good choice.

### Morphia

The people (I can't say guys because of Trisha Gee) at MongoDB also have made a mapping framework for MongoDB. Morphia is annotation driven, which means you'll have to annotate your POJOs in order to get them to work with Morphia (although you could get away with the defaults). It has support for most functions in MongoDB, but unfortunately no built-in support for GeoJSON and no support whatsoever for geoNear, which really sucks. MongoDB has been concentrating on the 3.0 version of the MongoDB Java Driver and they have been neglecting Morphia a bit. Perhaps in future versions, they'll provide the support needed.

Since I'm using the geoNear function, there is no choice but to take the piece of code in the Java Driver example and reuse that for the geo functionality of the use case. This is the implementation for Morphia:

``` groovy
import com.mongodb.*
import org.bson.types.ObjectId
import org.geojson.Point
import org.mongodb.morphia.*
import org.mongodb.morphia.annotations.*
import org.mongodb.morphia.converters.TypeConverter
import org.mongodb.morphia.mapping.MappedField
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.http.*
import org.springframework.web.bind.annotation.*

import javax.annotation.PostConstruct

import static org.springframework.web.bind.annotation.RequestMethod.GET

@RestController
@RequestMapping("/mongomorphia")
class CityControllerMorphia {
    final Datastore datastore

    @Autowired
    CityControllerMorphia(MongoClient mongoClient) {
        def morphia = new Morphia()
        morphia.mapper.converters.addConverter(GeoJsonPointTypeConverter)
        datastore = morphia.createDatastore(mongoClient, "citymorphia")
    }

    @RequestMapping(value="/", method = GET)
    List<City> index() {
        return datastore.find(City).asList()
    }

    @RequestMapping(value="/near/{cityName}", method = GET)
    ResponseEntity nearCity(@PathVariable String cityName) {
        def city = datastore.find(City, "name", cityName).get()
        if(city) {
            def point = new BasicDBObject([type: "Point", coordinates: [city.location.coordinates.longitude, city.location.coordinates.latitude]])
            def geoNearCommand =  new BasicDBObject([geoNear: "City", spherical: true, near: point])
            def closestCities = datastore.DB.command(geoNearCommand).toMap()
            def closest = (closestCities.results as List<Map>).get(1)
            return new ResponseEntity([name:closest.obj.name, distance:closest.dis/1000], HttpStatus.OK)
        }
        else {
            return new ResponseEntity(HttpStatus.NOT_FOUND)
        }
    }

    @PostConstruct
    void populateCities() {
        datastore.delete(datastore.createQuery(City))
        [new City(name: "London",
                location: new Point(-0.125487, 51.508515)),
         new City(name: "Paris",
                 location: new Point(2.352222, 48.856614)),
         new City(name: "New York",
                 location: new Point(-74.005973, 40.714353)),
         new City(name: "San Francisco",
                 location: new Point(-122.419416, 37.774929))].each {
            datastore.save(it)
        }
        datastore.getCollection(City).createIndex(new BasicDBObject("location", "2dsphere"))
    }

    @Entity
    static class City {
        @Id
        ObjectId id
        String name
        Point location
    }

    static class GeoJsonPointTypeConverter extends TypeConverter {

        GeoJsonPointTypeConverter() {
            super(Point)
        }

        @Override
        Object decode(Class<?> targetClass, Object fromDBObject, MappedField optionalExtraInfo) {
            double[] coordinates = (fromDBObject as DBObject).get("coordinates")
            return new Point(coordinates[0], coordinates[1])
        }

        @Override
        Object encode(Object value, MappedField optionalExtraInfo) {
            def point = value as Point
            return new BasicDBObject([type:"Point", coordinates:[point.coordinates.longitude, point.coordinates.latitude]])
        }
    }
}
```

Because Morphia has no built-in support for GeoJSON, you can either use the legacy way of an array of doubles containing the 2 coordinates or write your own converter. I chose the latter and it's actually not that difficult to write. Just don't forget to add the converter to Morphia. As you can see I had to annotate City with some Morphia annotations but for those familiar with JPA it's quite straightforward. You still need to create the 2dsphere index manually since Morphia doesn't have 2dsphere index query support (yet, it's to be added in the next release).

### Spring Data for MongoDB

Last but not least, I'm taking a look at how Spring Data can handle this use case. Those familiar with Spring Data know that you'll be writing repository interfaces to interact with your datastore, using the method names to indicate what query you want. In this case we need two queries: get a city by name and find the nearest cities. Spring Data has support for geospatial queries. 

Spring Data also has its own set of classes to represent geospatial coordinates, which are unfortunately GeoJSON incompatible (once again Spring does it its own way). Luckily generating the index automatically works as expected and MongoDB is able to handle the coordinate representation Spring Data uses. 

This is the implementation for Morphia:

``` groovy
import org.bson.types.ObjectId
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.data.geo.*
import org.springframework.data.mongodb.core.MongoTemplate
import org.springframework.data.mongodb.core.index.*
import org.springframework.data.mongodb.core.mapping.Document
import org.springframework.data.mongodb.repository.MongoRepository
import org.springframework.http.HttpStatus
import org.springframework.http.ResponseEntity
import org.springframework.web.bind.annotation.*

import javax.annotation.PostConstruct

import static org.springframework.web.bind.annotation.RequestMethod.GET

@RestController
@RequestMapping("/mongodata")
class CityControllerMongoData {

    final CityRepository cityRepository

    @Autowired
    CityControllerMongoData(CityRepository cityRepository) {
        this.cityRepository = cityRepository
    }

    @RequestMapping(value="/", method = GET)
    List<City> index() {
        return cityRepository.findAll()
    }

    @RequestMapping(value="/near/{cityName}", method = GET)
    ResponseEntity nearCity(@PathVariable String cityName) {
        def city = cityRepository.findByName(cityName)
        if(city) {
            GeoResults<City> closestCities = cityRepository.findByLocationNear(city.location, new Distance(10000, Metrics.KILOMETERS))
            def closest = closestCities.content.get(1)
            return new ResponseEntity([name:closest.content.name, distance:closest.distance.in(Metrics.KILOMETERS).value], HttpStatus.OK)
        }
        else {
            return new ResponseEntity(HttpStatus.NOT_FOUND)
        }
    }

    @PostConstruct
    void populateCities() {
        cityRepository.deleteAll()
        [ new City( name:"London",
                location: new Point(-0.125487, 51.508515)),
          new City( name:"Paris",
                  location: new Point(2.352222, 48.856614)),
          new City( name:"New York",
                  location: new Point(-74.005973, 40.714353)),
          new City( name:"San Francisco",
                  location: new Point(-122.419416, 37.774929)) ].each {
            cityRepository.save(it)
        }
    }

    @Document(collection = "city")
    static class City {
        ObjectId id
        String name
        @GeoSpatialIndexed(type = GeoSpatialIndexType.GEO_2DSPHERE)
        Point location
    }
}

interface CityRepository extends MongoRepository<CityControllerMongoData.City, ObjectId> {
    CityControllerMongoData.City findByName(String name);
    GeoResults<CityControllerMongoData.City> findByLocationNear(Point point, Distance distance);
}
```

When it comes to readability, Spring Data clearly wins. You don't need to know how queries in MongoDB are built, you just use the naming conventions in the repositories. One small thing you need to remember when using 2dsphere indexes is that you always have to add the distance parameter to the near query methods, otherwise Spring data omits the spherical option in the query for MongoDB (which will fail in that case). If you don't need the distance, you can make the near method return a list of City objects. You don't need to provide an implementation for the interface, Spring Data will do that for you.

When using Spring Data for MongoDB you also have access to the MongoTemplate class, which provides about the same low level possibilities of Jongo and the Java driver. This approach can also handle geoNear queries with ease.

You also may have noticed that this approach is the only approach that doesn't have the database name in the code. Spring Data uses a MongoTemplate underneath which is configured with the database you have provided in the configuration. That being said, you could inject the value of that property into a variable in the other examples and use that variable for the database name.

The only thing I dislike about Spring Data is the fact that they have chosen to take a non-standard route when it comes to representing geospatial data. If you have a MongoDB collection that has data with GeoJSON formatted data, you're essentially screwed as the repositories can't handle this (at least not for the generated near queries). I tried using GeoJSON classes in my mapped City object, but couldn't get the conversion working correctly (Spring Data doesn't use Jackson for the serialization). Also, the queries generated by the geoNear method in the repository uses the legacy approach of coordinate pairs instead of GeoJSON geometries. If Spring Data would provide support for GeoJSON formatted locations and queries it would be the cherry on top of an already nice cake.

### Conclusion

Based on this use case, my vote goes to Spring Data, followed by Jongo and the standard Java driver. Jongo takes second due to its mapping capabilities but otherwise the functionality of it and the Java driver are about the same. Morphia comes last due to the lack of support for the geoNear query and no built-in support for geo objects (except for double arrays). If the next version of Morphia is released, its spot might change. Using the Java driver can be a bit verbose, but with the combination of Groovy and the Java driver this verbosity can be overcome. 

This was a fairly simple example, but to me it was a great learning experience. Based on what I've seen here, I'll probably opt for Spring Data for MongoDB and if needed add standard Java driver to the mix to do things that are too complex to put in a repository. Perhaps with a more difficult use case my choice may have been different, but only time will tell. I think with the combination I have in my head, there's nearly nothing you can't do.




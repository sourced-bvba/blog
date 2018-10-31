---
layout: post
category : article
title: "Using GeoJSON with Spring Data for MongoDB and Spring Boot"
comments: true
tags : [java]
---

In my previous articles I compared 4 frameworks commonly used in communicating with MongoDB from the JVM and found out that in that use-case, Spring Data for MongoDB was the easiest solution. However I did make the remark that it doesn't use the GeoJSON format to store geolocation coordinates and geometries.

I tried to add GeoJSON support before, but couldn't get the conversion to work propertly. But after some extensive searching I found out that the reason for it not working was my use of Spring Boot: its autoconfiguration for MongoDB does not support custom conversion out of the box.<!--more--> Luckily, the solution was simple: provide an extra configuration that extends from `AbstractMongoConfiguration` and import that in the Boot application. In that configuration you can override the `customConversions()` and add your converters.

When you compare the geo classes in Spring Data and GeoJSON, I noticed that only a subset of GeoJSON geometries can be mapped on Spring Data geo classes: Point and Polygon. Spring Boot does not support LineString, MultiLineString, MultiPolygon or MultiPoint. However, in your mapped domain classes, you won't use these normally. Creating a converter that adheres to the GeoJSON format is quite straightforward.

``` groovy
import com.mongodb.BasicDBObject
import com.mongodb.DBObject
import org.springframework.core.convert.converter.Converter
import org.springframework.data.convert.ReadingConverter
import org.springframework.data.convert.WritingConverter
import org.springframework.data.geo.Point
import org.springframework.data.geo.Polygon

final class GeoJsonConverters {
    static List<Converter<?, ?>> getConvertersToRegister() {
        return [
                GeoJsonDBObjectToPointConverter.INSTANCE,
                GeoJsonDBObjectToPolygonConverter.INSTANCE,
                GeoJsonPointToDBObjectConverter.INSTANCE,
                GeoJsonPolygonToDBObjectConverter.INSTANCE
        ]
    }

    @WritingConverter
    static enum GeoJsonPointToDBObjectConverter implements Converter<Point, DBObject> {
        INSTANCE;

        @Override
        DBObject convert(Point source) {
            return new BasicDBObject([type: 'Point', coordinates: [source.x, source.y]])
        }
    }

    @ReadingConverter
    static enum GeoJsonDBObjectToPointConverter implements Converter<DBObject, Point> {
        INSTANCE;

        @Override
        Point convert(DBObject source) {
            def coordinates = source.coordinates as double[]
            return new Point(coordinates[0], coordinates[1])
        }
    }

    @WritingConverter
    static enum GeoJsonPolygonToDBObjectConverter implements Converter<Polygon, DBObject> {
        INSTANCE;

        @Override
        DBObject convert(Polygon source) {
            def coordinates = source.points.collect {
                [it.x, it.y]
            }
            return new BasicDBObject([type: 'Polygon', coordinates: coordinates])
        }
    }

    @ReadingConverter
    static enum GeoJsonDBObjectToPolygonConverter implements Converter<DBObject, Polygon> {
        INSTANCE;

        @Override
        Polygon convert(DBObject source) {
            def coordinates = source.coordinates as double[]
            return new Point(coordinates[0], coordinates[1])
        }
    }
}
```

To add those converters to the Spring context, you'll have to override some methods in your MongoDB spring configuration class.

``` groovy
import com.mongodb.Mongo
import org.springframework.beans.factory.annotation.*
import org.springframework.boot.SpringApplication
import org.springframework.boot.autoconfigure.EnableAutoConfiguration
import org.springframework.context.annotation.*
import org.springframework.data.mongodb.config.AbstractMongoConfiguration
import org.springframework.data.mongodb.core.convert.*

@EnableAutoConfiguration
@ComponentScan
@Configuration
@Import([MongoComparisonMongoConfiguration])
class MongoComparison
{
    static void main(String[] args) {
        SpringApplication.run(MongoComparison, args);
    }
}

@Configuration
class MongoComparisonMongoConfiguration extends AbstractMongoConfiguration {

    @Autowired
    Mongo mongo;

    @Value("\${spring.data.mongodb.database}")
    String databaseName;

    @Override
    protected String getDatabaseName() {
        return databaseName
    }

    @Override
    Mongo mongo() throws Exception {
        return mongo
    }

    @Override
    CustomConversions customConversions() {
        def customConverters = []
        customConverters << GeoJsonConverters.convertersToRegister
        return new CustomConversions(customConverters.flatten())
    }
}
```

As Spring Boot already provides the configuration of the `Mongo` instance and the name of the database, we can reuse these in the MongoDB configuration class. The custom conversions take preference over the existing ones for Point and Polygon.

I'll be writing a library this weekend to add support for all GeoJSON geometries in Spring Data for MongoDB. However, I already noticed it'll be very hard to provide support for those in generated query methods in repositories, but with annotated queries being possible, I don't think this will be a big issue but we'll see.

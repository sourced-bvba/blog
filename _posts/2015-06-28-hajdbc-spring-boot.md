---
layout: post
category : article
title: "Using HA-JDBC with Spring Boot"
comments: true
tags : [java]
---

Sometimes you want a bit more resilience in your application and not depend on a single database instance. There are different ways you can do this, but recently I've come across a really simple way of enabling failover and load-balancing to any Java backend using JDBC. The library is called HA-JDBC and allows you to transparently route calls to multiple datasources. It will automatically replicate all write calls (using a master-slave mechanism) and load-balance all read calls. If a datasource gets dropped, it can also rebuild a datasource when it gets back up.

I thought it would be nice to see whether I could add Spring Boot support which transparently enables HA-JDBC support. And thanks to the simplicity of Spring Boot, it wasn't that hard.<!--more--> This is the code to enable HA-JDBC support.

{% highlight groovy %}
import net.sf.hajdbc.SimpleDatabaseClusterConfigurationFactory
import net.sf.hajdbc.SynchronizationStrategy
import net.sf.hajdbc.balancer.BalancerFactory
import net.sf.hajdbc.balancer.random.RandomBalancerFactory
import net.sf.hajdbc.balancer.roundrobin.RoundRobinBalancerFactory
import net.sf.hajdbc.balancer.simple.SimpleBalancerFactory
import net.sf.hajdbc.cache.DatabaseMetaDataCacheFactory
import net.sf.hajdbc.cache.eager.EagerDatabaseMetaDataCacheFactory
import net.sf.hajdbc.cache.eager.SharedEagerDatabaseMetaDataCacheFactory
import net.sf.hajdbc.cache.lazy.LazyDatabaseMetaDataCacheFactory
import net.sf.hajdbc.cache.lazy.SharedLazyDatabaseMetaDataCacheFactory
import net.sf.hajdbc.cache.simple.SimpleDatabaseMetaDataCacheFactory
import net.sf.hajdbc.sql.Driver
import net.sf.hajdbc.sql.DriverDatabase
import net.sf.hajdbc.sql.DriverDatabaseClusterConfiguration
import net.sf.hajdbc.state.StateManagerFactory
import net.sf.hajdbc.state.bdb.BerkeleyDBStateManagerFactory
import net.sf.hajdbc.state.simple.SimpleStateManagerFactory
import net.sf.hajdbc.state.sql.SQLStateManagerFactory
import net.sf.hajdbc.state.sqlite.SQLiteStateManagerFactory
import net.sf.hajdbc.sync.*
import net.sf.hajdbc.util.concurrent.cron.CronExpression
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass
import org.springframework.boot.context.properties.ConfigurationProperties
import org.springframework.context.annotation.Configuration

import javax.annotation.PostConstruct

@ConfigurationProperties(prefix="hajdbc")
@ConditionalOnClass(Driver)
@Configuration
class HaJdbcConfiguration {
    List<DriverDatabase> driverDatabases;
    String clusterName = "default"
    String cronExpression = "0 0/1 * 1/1 * ? *"
    String balancerFactory = "round-robin"
    String defaultSynchronizationStrategy = "full"
    String databaseMetaDataCacheFactory = "shared-eager"
    String stateManagerFactory = "simple"
    String stateManagerUrl
    String stateManagerUser
    String stateManagerPassword
    String stateManagerLocation
    boolean identityColumnDetectionEnabled = true
    boolean sequenceDetectionEnabled = true

    @PostConstruct
    def register() {
        DriverDatabaseClusterConfiguration config = new DriverDatabaseClusterConfiguration();
        config.setDatabases(driverDatabases);
        config.databaseMetaDataCacheFactory = DatabaseMetaDataCacheChoice.fromId(databaseMetaDataCacheFactory)
        config.balancerFactory = BalancerChoice.fromId(balancerFactory)
        config.stateManagerFactory = StateManagerChoice.fromId(stateManagerFactory)
        switch(stateManagerFactory) {
            case "sql":
               def mgr = config.stateManagerFactory as SQLStateManagerFactory
                if(stateManagerUrl)
                    mgr.urlPattern = stateManagerUrl
                if(stateManagerUser)
                    mgr.user = stateManagerUser
                if(stateManagerPassword)
                    mgr.password = stateManagerPassword
                break;
            case "berkeleydb":
            case "sqlite":
                def mgr = config.stateManagerFactory as BerkeleyDBStateManagerFactory
                if(stateManagerLocation)
                    mgr.locationPattern = stateManagerLocation
                break;
        }
        config.synchronizationStrategyMap = SynchronizationStrategyChoice.idMap
        config.defaultSynchronizationStrategy = defaultSynchronizationStrategy
        config.identityColumnDetectionEnabled = identityColumnDetectionEnabled
        config.sequenceDetectionEnabled = sequenceDetectionEnabled
        config.setAutoActivationExpression(new CronExpression(cronExpression))

        Driver.setConfigurationFactory(clusterName,
                                       new SimpleDatabaseClusterConfigurationFactory<java.sql.Driver, DriverDatabase>(config));
    }

    static enum DatabaseMetaDataCacheChoice {
        SIMPLE(new SimpleDatabaseMetaDataCacheFactory()),
        LAZY(new LazyDatabaseMetaDataCacheFactory()),
        EAGER(new EagerDatabaseMetaDataCacheFactory()),
        SHARED_LAZY(new SharedLazyDatabaseMetaDataCacheFactory()),
        SHARED_EAGER(new SharedEagerDatabaseMetaDataCacheFactory());

        DatabaseMetaDataCacheFactory databaseMetaDataCacheFactory;

        DatabaseMetaDataCacheChoice(DatabaseMetaDataCacheFactory databaseMetaDataCacheFactory) {
            this.databaseMetaDataCacheFactory = databaseMetaDataCacheFactory
        }

        static DatabaseMetaDataCacheFactory fromId(String id) {
            values().find {
                it.databaseMetaDataCacheFactory.id == id
            }.databaseMetaDataCacheFactory
        }
    }

    static enum BalancerChoice {
        ROUND_ROBIN(new RoundRobinBalancerFactory()),
        RANDOM(new RandomBalancerFactory()),
        SIMPLE(new SimpleBalancerFactory());

        BalancerFactory balancerFactory;

        BalancerChoice(BalancerFactory balancerFactory) {
            this.balancerFactory = balancerFactory
        }

        static BalancerFactory fromId(String id) {
            values().find {
                it.balancerFactory.id == id
            }.balancerFactory
        }
    }

    static enum SynchronizationStrategyChoice {
        FULL(new FullSynchronizationStrategy()),
        DUMP_RESTORE(new DumpRestoreSynchronizationStrategy()),
        DIFF(new DifferentialSynchronizationStrategy()),
        FASTDIFF(new FastDifferentialSynchronizationStrategy()),
        PER_TABLE_FULL(new PerTableSynchronizationStrategy(new FullSynchronizationStrategy())),
        PER_TABLE_DIFF(new PerTableSynchronizationStrategy(new DifferentialSynchronizationStrategy())),
        PASSIVE(new PassiveSynchronizationStrategy());

        SynchronizationStrategy synchronizationStrategy;

        SynchronizationStrategyChoice(SynchronizationStrategy synchronizationStrategy) {
            this.synchronizationStrategy = synchronizationStrategy
        }

        static SynchronizationStrategy fromId(String id) {
            values().find {
                it.synchronizationStrategy.id == id
            }.synchronizationStrategy
        }

        static Map<String, SynchronizationStrategy> getIdMap() {
            return values().collectEntries {
                [(it.synchronizationStrategy.id):it.synchronizationStrategy]
            }
        }
    }

    static enum StateManagerChoice {
        SIMPLE(new SimpleStateManagerFactory()),
        BERKELEYDB(new BerkeleyDBStateManagerFactory()),
        SQLITE(new SQLiteStateManagerFactory()),
        SQL(new SQLStateManagerFactory());

        private StateManagerFactory stateManagerFactory;

        StateManagerChoice(StateManagerFactory stateManagerFactory) {
            this.stateManagerFactory = stateManagerFactory
        }

        static StateManagerFactory fromId(String id) {
            values().find {
                it.stateManagerFactory.id == id
            }.stateManagerFactory
        }

    }
}
{% endhighlight %}

When your Spring Boot application picks up this class, you can configure your Spring Boot application to use HA-JDBC by configuring its databases and the standard datasource Spring Boot provides when including JDBC support. For example, this configuration creates a datasource that replicates between two MySQL databases.

{% highlight properties %}
spring.datasource.url=jdbc:ha-jdbc:default
spring.datasource.username=root
spring.datasource.driver-class-name=net.sf.hajdbc.sql.Driver

hajdbc.driverDatabases[0].id=db1
hajdbc.driverDatabases[0].location=jdbc:mysql://localhost/hatest1
hajdbc.driverDatabases[0].user=root
hajdbc.driverDatabases[1].id=db2
hajdbc.driverDatabases[1].location=jdbc:mysql://localhost/hatest2
hajdbc.driverDatabases[1].user=root
{% endhighlight %}

Be mindful that you always need to use the same username for all datasources. With this configuration, by default it will check every minute whether any disabled databases (due to connection failures) can be enabled again and will resync those databases.

This is a really simple way to provide high-availability with fail-over and load balancing.

---
layout: post
category : "java"
title: "Combining Liquibase with Spring and JPA, revisited"
tags : [spring]
---

In one of my previous articles, I described how you could combine Spring and Liquibase to make a schema self-updating JPA application. It has been a while since I wrote the article and a lot of things have changes. Both Spring and Liquibase have a new major version release out and a lot of API’s have changed.

The principle stays the same, but the implementation has changed a bit. It’s based on the built-in Spring integration now provided by Liquibase. The reason I slightly altered it is because 1) it’s based on Spring 2.0.x, 2) it uses String instead of Resource for the changelog (Resource fields result in better autocompletion in most IDE’s) and 3) it doesn’t allow me to manipulate the table names used by Liquibase (needed in some corporate environments where naming conventions are in place).

<!--more-->

So without further ado (and with imports), the new and improved SpringLiquibaseUpdater for Liquibase 2.0.3 and Spring 3.x:



``` java
import liquibase.Liquibase;
import liquibase.database.Database;
import liquibase.database.DatabaseFactory;
import liquibase.database.jvm.JdbcConnection;
import liquibase.exception.DatabaseException;
import liquibase.exception.LiquibaseException;
import liquibase.logging.LogFactory;
import liquibase.logging.Logger;
import liquibase.resource.ResourceAccessor;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.io.Resource;
import org.springframework.core.io.ResourceLoader;

import javax.annotation.PostConstruct;
import javax.sql.DataSource;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.io.InputStream;
import java.net.URL;
import java.sql.Connection;
import java.sql.SQLException;
import java.util.Enumeration;
import java.util.Map;
import java.util.Vector;

public class SpringLiquibaseUpdater {
    private class SpringResourceOpener implements ResourceAccessor {
        public InputStream getResourceAsStream(String file) throws IOException {
            try {
                Resource resource = getResource(file);
                return resource.getInputStream();
            }
            catch ( FileNotFoundException ex ) {
                return null;
            }
        }

        public Enumeration<URL> getResources(String packageName) throws IOException {
            Vector<URL> tmp = new Vector<URL>();

            tmp.add(getResource(packageName).getURL());

            return tmp.elements();
        }

        public Resource getResource(String file) {
            return getResourceLoader().getResource(file);
        }

        public ClassLoader toClassLoader() {
            return getResourceLoader().getClassLoader();
        }
    }

    @Autowired
    private ResourceLoader resourceLoader;

    private final DataSource dataSource;

    private Logger log = LogFactory.getLogger(SpringLiquibaseUpdater.class.getName());

    private final Resource changeLog;

    private String contexts;

    private Map<String, String> parameters;

    private String defaultSchema;

    private boolean dropFirst = false;

    private String changeLogTableName;
    private String changeLogLockTableName;

    public SpringLiquibaseUpdater(DataSource dataSource, Resource changeLog) {
        super();
        this.dataSource = dataSource;
        this.changeLog = changeLog;
    }

    public boolean isDropFirst() {
        return dropFirst;
    }

    public void setDropFirst(boolean dropFirst) {
        this.dropFirst = dropFirst;
    }

    public String getDatabaseProductName() throws DatabaseException {
        Connection connection = null;
        String name = "unknown";
        try {
            connection = getDataSource().getConnection();
            Database database =
                    DatabaseFactory.getInstance().findCorrectDatabaseImplementation(
                        new JdbcConnection(dataSource.getConnection()));
            name = database.getDatabaseProductName();
        } catch (SQLException e) {
            throw new DatabaseException(e);
        } finally {
            if (connection != null) {
                try {
                    if (!connection.getAutoCommit())
                    {
                        connection.rollback();
                    }
                    connection.close();
                } catch (Exception e) {
                    log.warning("problem closing connection", e);
                }
            }
        }
        return name;
    }

    /**
     * The DataSource that liquibase will use to perform the migration.
     *
     * @return
     */
    private DataSource getDataSource() {
        return dataSource;
    }

    /**
     * Returns a Resource that is able to resolve to a file or classpath resource.
     *
     * @return
     */
    private Resource getChangeLog() {
        return changeLog;
    }

    public String getContexts() {
        return contexts;
    }

    public void setContexts(String contexts) {
        this.contexts = contexts;
    }

    public String getDefaultSchema() {
        return defaultSchema;
    }

    public void setDefaultSchema(String defaultSchema) {
        this.defaultSchema = defaultSchema;
    }

    /**
     * Executed automatically when the bean is initialized.
     */
    @PostConstruct
    public void performAfterInitialization() throws LiquibaseException {
        String shouldRunProperty = System.getProperty(Liquibase.SHOULD_RUN_SYSTEM_PROPERTY);
        if (shouldRunProperty != null && !Boolean.valueOf(shouldRunProperty)) {
            LogFactory.getLogger().info("Liquibase did not run because '" + Liquibase.SHOULD_RUN_SYSTEM_PROPERTY +
                "' system property was set to false");
            return;
        }

        Connection c = null;
        Liquibase liquibase = null;
        try {
            c = getDataSource().getConnection();
            liquibase = createLiquibase(c);
            liquibase.update(getContexts());
        } catch (SQLException e) {
            throw new DatabaseException(e);
        } catch (IOException e) {
            throw new RuntimeException(e);
        } finally {
            if (c != null) {
                try {
                    c.rollback();
                    c.close();
                } catch (SQLException e) {
                    //nothing to do
                }
            }
        }

    }

    protected Liquibase createLiquibase(Connection c) throws LiquibaseException, IOException {
        Liquibase liquibase = new Liquibase(getChangeLog().getURL().toString(), createResourceOpener(), createDatabase(c));
        if (parameters != null) {
            for(Map.Entry<String, String> entry: parameters.entrySet()) {
                liquibase.setChangeLogParameter(entry.getKey(), entry.getValue());
            }
        }

        if (isDropFirst()) {
            liquibase.dropAll();
        }

        return liquibase;
    }

    /**
     * Subclasses may override this method add change some database settings such as
     * default schema before returning the database object.
     * @param c
     * @return a Database implementation retrieved from the {@link DatabaseFactory}.
     * @throws DatabaseException
     */
    protected Database createDatabase(Connection c) throws DatabaseException {
        Database database = DatabaseFactory.getInstance().findCorrectDatabaseImplementation(new JdbcConnection(c));
        if(this.changeLogTableName != null)
            database.setDatabaseChangeLogTableName(this.changeLogTableName);
        if(this.changeLogLockTableName != null)
            database.setDatabaseChangeLogLockTableName(this.changeLogLockTableName);
        if (this.defaultSchema != null)
            database.setDefaultSchemaName(this.defaultSchema);
        return database;
    }

    public void setChangeLogParameters(Map<String, String> parameters) {
        this.parameters = parameters;
    }

    /**
     * Create a new resourceOpener.
     */
    protected SpringResourceOpener createResourceOpener() {
        return new SpringResourceOpener();
    }

    private ResourceLoader getResourceLoader() {
        return resourceLoader;
    }

    public String getChangeLogLockTableName() {
        return changeLogLockTableName;
    }

    public void setChangeLogLockTableName(String changeLogLockTableName) {
        this.chaù}ngeLogLockTableName = changeLogLockTableName;
    }

    public String getChangeLogTableName() {
        return changeLogTableName;
    }

    public void setChangeLogTableName(String changeLogTableName) {
        this.changeLogTableName = changeLogTableName;
    }

    @Override
    public String toString() {
        return getClass().getName()+"("+this.getResourceLoader().toString()+")";
    }
}
```

Have fun!

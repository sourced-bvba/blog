---
layout: post
category : "java"
title: "Having fun with JSON and DbUnit, continued"
tags : [testing, database]
---

The beauty of a blog like this is the fact that people read the articles and take the ideas in them to the next level. And that’s exactly [what Dan Haywood did](http://danhaywood.com/2011/12/20/db-unit-testing-with-dbunit-json-hsqldb-and-junit-rules/).

He reused my JSONDataSet idea and combined it with JUnit 4′s new Rule system to add configurable dataset loading to individual test methods. It’s a basic, yet very functional implementation. But I thought, what the heck, let’s take his idea a step further.

For most of my projects, I use Spring, JPA and Liquibase for database operations. The use of Liquibase makes the @Ddl annotation in Dan’s implementation practically useless, so in my implementation, I could drop this part of the code (I’m not really a fan of unit tests handling my schema creation). And since I’m using Spring and JPA, I can assume that there is a valid database connection in my Spring config under the form of a DataSource, so I can omit the entire constructor from the DbUnitRule and use injection instead. In his comments he mentions the idea of having the possibility of using the different file formats DbUnit uses so I added that as well (my JSON dataset implementation, flat XML and CSV). I then added some finishing touches (adding the ability to use Spring-style resources, enabling multiple datasets, reordering dataset so that foreign keys are resolved correctly and using TestRule instead of the deprecated MethodRule). The result is a very terse but powerful implementation.<!--more-->


{% highlight java %}
/**
 * A JUnit rule which enables per-test data set loading using DbUnit. This rule is meant to be
 * used in a Spring context, reusing a DataSource in the context.
 *
 * This rule supports Spring-style resource urls (classpath:/..., file:/...) and is capable of handling
 * multiple JSON, CSV and Flat XML DbUnit data set files (auto-detected by extension).
 * The data sets are automatically reordered so that foreign keys are resolved correctly when loading
 * the data set.
 *
 * @author Lieven DOCLO
 * @author Dan HAYWOOD
 */
@Component
public class DbUnitRule implements TestRule {
    /**
     * Configures the data set that should be loaded by the test method. All Spring-style resource
     * values are supported, data sets are currently supported in JSON, CSV and Flat XML (auto-detected by extension).
     */
    @Retention(RetentionPolicy.RUNTIME)
    @Target({ElementType.METHOD})
    public static @interface Data {
        String[] resources();
    }

    @Autowired
    private DataSource dataSource;
    @Autowired
    private ResourceLoader resourceLoader;

    @Override
    public Statement apply(final Statement base, final Description description) {
        return new Statement() {
            @Override
            public void evaluate() throws Throwable {
                try {
                    Data data = description.getAnnotation(Data.class);
                    if (data != null) {

                        String[] dataSetFiles = data.resources();
                        List<IDataSet> dataSets = new ArrayList<IDataSet>(dataSetFiles.length);
                        for (String dataSetFile : dataSetFiles) {
                            IDataSet ds;
                            if (dataSetFile.endsWith(".json"))
                                ds = new JSONDataSet(resourceLoader.getResource(dataSetFile).getInputStream());
                            else if (dataSetFile.endsWith(".xml"))
                                ds = new FlatXmlDataSet(resourceLoader.getResource(dataSetFile).getInputStream());
                            else if (dataSetFile.endsWith(".csv"))
                                ds = new CsvDataSet(resourceLoader.getResource(dataSetFile).getFile());
                            else
                                throw new IllegalStateException("DbUnitRule only supports JSON, CSV or Flat XML data sets for the moment");
                            dataSets.add(ds);
                        }
                        CompositeDataSet dataSet = new CompositeDataSet(dataSets.toArray(new IDataSet[dataSets.size()]));
                        DatabaseDataSourceConnection databaseDataSourceConnection = new DatabaseDataSourceConnection(dataSource);
                        IDataSet fkDataSet = new FilteredDataSet(new DatabaseSequenceFilter(databaseDataSourceConnection), dataSet);
                        DatabaseOperation.CLEAN_INSERT.execute(databaseDataSourceConnection, fkDataSet);
                    }
                    base.evaluate();
                } finally {
                }
            }
        };
    }
}
{% endhighlight %}

Using this is extremely easy: first declare the DbUnitRule as a Spring bean or use classpath scanning (hence the @Component). Then simply inject the rule in your Spring enabled tests like this:

{% highlight java %}
@ContextConfiguration("classpath:/be/insaneprogramming/examples/spring-config.xml")
public class DbUnitRuleTests extends AbstractTransactionalJUnit4SpringContextTests {
    public @Rule @Autowired DbUnitRule dbUnitRule ;
    // other autowiring like services, dao's
    @Test
    @Data(resources = "classpath:/be/insaneprogramming/examples/testDataForTryingOutMyRule.json")
    public void tryingOutMyRule() {
        // do stuff with database data
        ...
    }

}
{% endhighlight %}

This now allows to use specialized data sets for particular test cases, which also causes data sets to become smaller and therefore more manageable. This was one of the drawbacks I encountered using the method as described in my previous article on JUnit and DBUnit.

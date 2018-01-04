---
title: "Local Resources"
bg:  fnavy
color: white
fa-icon: cube
---

A local resource is an asset that is managed locally and similar to a deployment environment resource which be drawn on to effectively test assumptions. The easiest way to get started with local resources is to create a local resource provider implementation. In this example we will create an in-memory HSQLDB local resource. Using Testify's resource provider archetype create a new project:

{% highlight bash linenos=table %}
mvn archetype:generate \
 -DarchetypeGroupId=org.testifyproject.archetypes \
 -DarchetypeArtifactId=junit-resourceprovider-archetype
{% endhighlight %}

### Example HSQL LocalResourceProvider
To create a hsql resource provider that can be used in your test cases simply implement the LocalResourceProvider SPI contract:

{% highlight java linenos=table %}
public class InMemoryHSQLResource implements
        LocalResourceProvider<JDBCDataSource, DataSource, Connection> {

    private JDBCDataSource server;
    private Connection client;

    @Override
    public JDBCDataSource configure(TestContext testContext,
            LocalResource localResource,
            PropertiesReader configReader) {
        JDBCDataSource dataSource = new JDBCDataSource();
        String url = format("jdbc:hsqldb:mem:%s?default_schema=public", testContext.getName());
        dataSource.setUrl(url);
        dataSource.setUser("sa");
        dataSource.setPassword("");

        return dataSource;
    }

    @Override
    public LocalResourceInstance<DataSource, Connection> start(TestContext testContext,
            LocalResource localResource,
            JDBCDataSource dataSource)
            throws Exception {
        server = dataSource;
        client = dataSource.getConnection();

        return LocalResourceInstanceBuilder.builder()
                .resource(server, DataSource.class)
                .client(client, Connection.class)
                .build("hsql", localResource);
    }

    @Override
    public void stop(TestContext testContext,
            LocalResource localResource,
            LocalResourceInstance<DataSource, Connection> instance)
            throws Exception {
        server.getConnection()
                .createStatement()
                .executeQuery("SHUTDOWN");
        client.close();
    }

}
{% endhighlight %}

LocalResourceProvider? Hmmm...

In the above implementation of `LocalResourceProvider` contract we are creating a reusable HSQL in-memory database resource provider. Notice that our implementation:
1. Specifies 3 type parameters of `LocalResourceProvider` contract, JDBCDataSource (resource configuration), DataSource (the resource), and Connection (resource client)
1. Does not declare a constructor
1. Implements three methods defined by the contract (configure, start, and stop)
1. Has state (server and client instances)

Our example is pretty simple because our configuration object and resource object are the same but this may not be the case for a more complex implementations. Regardless of the complexity of an implementation the ultimate goal is to provide a managed, reusable, and disposable resource for our integration and system tests and to enable us to replace production code that relies on the resource (DataSource) and client (Connection) with the ones provided by our `LocalResourceProvider` implementation.

As you might guess the `configure` method is responsible for creating a pre-configured configuration object that can be refined further via `@ConfigHander` prior to starting the resource and execution of our test case. The `start` method is responsible for starting the resource and returning a `LocalResourceInstance` that encapsulates a resource component and a client component which is configured to communicate with the resource. In this example our resource instance happens to comprises of an in-memory HSQL JDBC `DataSource` and `Connection`. Finally, the stop method is responsible for gracefully stopping the resource and closing the client.

Now that we have a resource provider implementation we can start using it to provide a database resource for our integration and system tests by annotating our test class with `@LocalResource(InMemoryHSQLResource.class)`:

{% highlight java linenos=table %}
@Module(GreetingConfig.class)
@LocalResource(InMemoryHSQLResource.class)
@RunWith(SpringIntegrationTest.class)
public class GetGreetingIT {

    @Sut
    GetGreeting cut;

    @Real
    GreetingRepository greetingRepository;

    @Real
    ResourceInstance<DataSource, Connection> resourceInstance;

    @Real
    DataSource dataSource;

    @Real
    Connection connection;

    @ConfigHandler
    void configure(JDBCDataSource dataSource) {
        //XXX: You can refine the configuration object of the
        //InMemoryHSQLResource implementation
    }

    @Test
    public void givenExistentKeyGetGreetingShouldReturnGreetingEntity() {
        //Arrange
        UUID id = UUID.fromString("0d216415-1b8e-4ab9-8531-fcbd25d5966f");

        //Act
        Optional<GreetingEntity> result = cut.getGreeting(id);

        //Assert
        assertThat(result).isPresent();
    }

}
{% endhighlight %}

Notice that we are able to inject the resource instance, data source and connection created in our `InMemoryHSQLResource` resource provider implementation which can be useful if you wish to directly interact with the resource and the client. Please note that the above example is kitchen-sink example and may not reflect a typical use-case.


For complete LocalResourceProvider implementations and examples take a look at:
[Example JUnit Resource Provider][example-junit-resourceprovider]
[Local Resource Implementations][local-resources]

[example-junit-resourceprovider]: https://github.com/testify-project/examples/tree/master/junit4/example-junit-resourceprovider
[local-resources]: https://github.com/testify-project/resources
[docker-configuration]: https://docs.docker.com/engine/articles/configuring
[docker-mac-install]: https://docs.docker.com/engine/installation/mac/
[docker-windows-install]: https://docs.docker.com/engine/installation/windows/
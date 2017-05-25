---
title: "Test Resources"
bg:  fnavy
color: white
fa-icon: cube
---

A test resource is an asset that can be drawn on to effectively test assumptions. Testify supports three types of test resources, local, virtual, and remote.

---

## Local Resources

---

The easiest way to get started with local resources is to create a local resource provider implementation. In this example we will create an in-memory HSQLDB local resource. Using Testify's required resource archetype create a new project:

{% highlight bash linenos=table %}
mvn archetype:generate \
 -DarchetypeGroupId=org.testifyproject.archetypes \
 -DarchetypeArtifactId=junit-resourceprovider-archetype
{% endhighlight %}

### Example HSQL LocalResourceProvider
To create a hsql resource provider that can be used in your test cases simply implement the LocalResourceProvider SPI contract:

{% highlight java linenos=table %}
public class InMemoryHSQLResource
        implements LocalResourceProvider<JDBCDataSource, DataSource, Connection> {

    private JDBCDataSource server;
    private Connection client;

    @Override
    public JDBCDataSource configure(TestContext testContext) {
        JDBCDataSource dataSource = new JDBCDataSource();
        String url = format("jdbc:hsqldb:mem:%s?default_schema=public", testContext.getName());
        dataSource.setUrl(url);
        dataSource.setUser("sa");
        dataSource.setPassword("");

        return dataSource;
    }

    @Override
    public LocalResourceInstance start(TestContext testContext,
            LocalResource localResource,
            JDBCDataSource dataSource)
            throws Exception {
        server = dataSource;
        client = dataSource.getConnection();

        return LocalResourceInstanceBuilder.builder()
                .resource(server, "inmemoryHSQLDataSource", DataSource.class)
                .client(client, "inmemoryHSQLConnection", Connection.class)
                .build();
    }

    @Override
    public void stop(TestContext testContext, LocalResource localResource)
            throws Exception {
        server.getConnection()
                .createStatement()
                .executeQuery("SHUTDOWN");
        client.close();
    }
{% endhighlight %}

LocalResourceProvider? Hmmm...

In the above implementation of `LocalResourceProvider` contract we are creating a reusable HSQL in-memory database resource provider. Notice that our implementation:
1. Specifies 3 type parameters of `LocalResourceProvider` contract, JDBCDataSource (resource configuration), DataSource (the resource), and Connection (resource client)
1. Does not declare a constructor
1. Implements three methods defined by the contract (configure, start, and stop)
1. Has state (server and client instances)

Our example is pretty simple because our configuration object and resource object are the same but this may not be the case for a more complex implementations. Regardless of the complexity of an implementation the ultimate goal is to provide a managed, reusable, and disposable resource for our integration and system tests and to enable us to replace production code that relies on the resource (DataSource) and client (Connection) with the ones provided by our `LocalResourceProvider` implementation.

As you might guess the `configure` method is responsible for creating a pre-configured configuration object that can be refined further via `@ConfigHander` annotated method prior to starting the resource and execution of our test case. The `start` method is responsible for starting the resource and returning a `LocalResourceInstance` that encapsulates a resource component and a client component which is configured to communicate with the resource. In this example our resource instance happens to comprises of an in-memory HSQL JDBC DataSource and connection. Finally, the stop method is responsible for gracefully stopping the resource and closing the client.

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

Notice that we are able to inject the resource instance, data source and connection created in our `InMemoryHSQLResource` resource provider implementation which can be useful if you wish to interact with the resource and the client. Please note that the above example is kitchen-sink example and may not reflect a typical use-case.


For complete LocalResourceProvider implementations and examples take a look at:
[Example JUnit Resource Provider][example-junit-resourceprovider]
[Local Resource Implementations][local-resources]

---

## Virtual Resources

---

### Docker Virtual Resource

Testify uses Docker Containers to provide virtual resources. Before you can use virtual resources in your integration and system tests you will need to install and configure Docker.

**Install Docker**

Before you can test your application with Docker container based virtual resources you must install Docker on your system. To install Docker please follow the [Docker instalation instructions for your OS platform](https://docs.docker.com/engine/installation/).

**Configuring Docker**

Testify uses Docker Remote API to pull images and manage Docker containers. By default Docker Remote API is not enabled. To enable it insure that `DOCKER_OPTS` and `DOCKER_TLS_VERIFY` environmental variables are set to the following:

{% highlight bash linenos=table %}
export DOCKER_OPTS="-H tcp:/127.0.0.1:2375 -H unix:///var/run/docker.sock"
export DOCKER_HOST="http://127.0.0.1:2375"
export DOCKER_TLS_VERIFY=0
{% endhighlight %}

These environmental variable must be set prior to executing your tests. You can set these variables permanently:

**Mint 17 / Ubuntu 14.04**

TODO

**Ubuntut 15.04 / Mint 18**

TODO

**Redhat / Fedora / CentOS / Oracle Linux**

TODO

**Windows**

TODO

**Mac OS X**

TODO

<!--
**Mac / Windows (Docker-Machine)**

On Macs and Windows systems Docker installation and configuration process is a
bit different than on Linux systems as they require a VM (VirtualBox). Follow
the Docker [Mac Installation][docker-mac-install] and
[Windows Installation][docker-windows-install] documentation to install and
configure Docker on your system. Next you will need to configure your VM to
disable Docker Remote API SSL configuration and enable VM port forwarding.

- Capture your VM's SSH port:
{% highlight bash linenos=table %}
SSHPORT=$(docker-machine inspect default | \
grep SSHPort | \
awk '{ print $2 }' | \
sed s/\"//g | \
sed s/,//g)
{% endhighlight %}
- SSH into the Docker VM:
{% highlight bash linenos=table %}
#if you don't have sshpass installed you can enter the password 'tcuser' manually.
sshpass -p tcuser ssh -p $SSHPORT docker@localhost
{% endhighlight %}
- Update docker2boot profile configuration and insure DOCKER_TLS is set to no:
{% highlight bash linenos=table %}
tce-load -w -i nano
sudo nano /var/lib/boot2docker/profile
{% endhighlight %}
{% highlight bash linenos=table %}
CACERT=/var/lib/boot2docker/ca.pem
DOCKER_HOST='-H tcp://0.0.0.0:2376'
DOCKER_STORAGE=aufs
DOCKER_TLS=no
SERVERKEY=/var/lib/boot2docker/server-key.pem
SERVERCERT=/var/lib/boot2docker/server.pem
{% endhighlight %}
- Restart Docker and Exit
{% highlight bash linenos=table %}
sudo /etc/init.d/docker restart
exit
{% endhighlight %}
- Setup VirtualBox Port Forwarding:
{% highlight bash linenos=table %}
#assumes your VM is named "default"
VBoxManage controlvm default natpf1 report_api,tcp,127.0.0.1,2375,,2376
{% endhighlight %}
-->

***Docker Remote API and Testify***

By default Testify communicates with Docker through Docker Remote API on
non-secure `http://127.0.0.1:2375` URL endpoint. If you wish to use an endpoint
with a different IP address and port or are using a Docker-Machine, or want to
use secure-communication then you will need to explictly configure the Docker
Client used by Testify using `@ConfigHandler` (not recommended):
{% highlight java linenos=table %}
@Config
public void configure(DockerClientConfig.DockerClientConfigBuilder builder) {
    builder.withUri("http://192.168.99.100:2376");
    //add additional configuration such as registry configuration here
}
{% endhighlight %}


[example-junit-resourceprovider]: https://github.com/testify-project/examples/tree/master/junit4/example-junit-resourceprovider
[local-resources]: https://github.com/testify-project/resources
[docker-configuration]: https://docs.docker.com/engine/articles/configuring
[docker-mac-install]: https://docs.docker.com/engine/installation/mac/
[docker-windows-install]: https://docs.docker.com/engine/installation/windows/
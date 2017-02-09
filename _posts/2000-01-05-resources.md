---
title: "Test Resources"
bg:  fnavy
color: white
fa-icon: cubes
---

---

## Custom Resource Providers

---

### Create Custom Resource Provider Project from Archetype
The easiest way to get started with creating custom required resource providers is to create a new project using Testify's required resource archetype:

{% highlight bash linenos=table %}
mvn archetype:generate \
 -DarchetypeGroupId=org.testify.archetypes \
 -DarchetypeArtifactId=junit-resourceprovider-archetype
{% endhighlight %}

Alternatively, you can manually add the following dependencies to your project:

{% highlight xml linenos=table %}
<dependencies>
    <dependency>
        <groupId>org.testify</groupId>
        <artifactId>api</artifactId>
        <version>${project.version}</version>
    </dependency>
    <dependency>
        <groupId>org.testify</groupId>
        <artifactId>core</artifactId>
        <version>${project.version}</version>
    </dependency>
    <dependency>
        <groupId>org.hsqldb</groupId>
        <artifactId>hsqldb</artifactId>
        <version>${hsqldb.version}</version>
    </dependency>

    <!-- Test Deps -->
    <dependency>
        <groupId>org.testify.junit</groupId>
        <artifactId>unit-test</artifactId>
        <version>${project.version}</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.testify.mock</groupId>
        <artifactId>mockito</artifactId>
        <version>${project.version}</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.testify.tools</groupId>
        <artifactId>test-logger</artifactId>
        <version>${project.version}</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
{% endhighlight %}

### Example Custom ResourceProvider
To create a custom resource provider that can be used in your test cases simply implement the ResourceProvider SPI contract:

{% highlight java linenos=table %}
public class InMemoryHSQLResource implements ResourceProvider<JDBCDataSource, DataSource, Connection> {

    private JDBCDataSource server;
    private Connection client;

    @Override
    public JDBCDataSource configure(TestContext testContext) {
        JDBCDataSource dataSource = new JDBCDataSource();
        dataSource.setUrl(format("jdbc:hsqldb:mem:%s?default_schema=public", testContext.getName()));
        dataSource.setUser("sa");
        dataSource.setPassword("");

        return dataSource;
    }

    @Override
    public ResourceInstance<DataSource, Connection> start(TestContext testContext, JDBCDataSource dataSource) {
        try {
            server = dataSource;
            client = dataSource.getConnection();

            return new ResourceInstanceBuilder<DataSource, Connection>()
                    .server(server, DataSource.class)
                    .client(client, Connection.class)
                    .build();
        }
        catch (SQLException e) {
            throw new IllegalStateException(e);
        }
    }

    @Override
    public void stop() {
        try {
            server.getConnection()
                    .createStatement()
                    .executeQuery("SHUTDOWN");
            client.close();
        }
        catch (SQLException e) {
            throw new IllegalStateException(e);
        }
    }

}
{% endhighlight %}

ResourceProvider? Hmmm...

In the above implementation of `ResourceProvider` contract we are implementing a reusable HSQL in-memory database resource provider. Notice that our implementation:
1. Specifies 3 type parameters of `ResourceProvider` contract, JDBCDataSource (configuration), DataSource (server), and Connection (client)
1. Does not declare a constructor
1. Implements three methods defined by the contract (configure, start, and stop)
1. Has state (server and client instances)

Our example is pretty simple because our configuration object and server object are the same but this may not be the case for a more complex implementations. Regardless of the complexity of an implementation the ultimate goal is to provide a managed and reusable resource to integration and system tests and to replace production code that relies on the server (DataSource) and client (Connection) with the ones provided by our `ResourceProvider` implementation.

As you might guess the `configure` method is responsible for creating a pre-configured configuration object that can be refined further in a test class (via `@ConfigHander` annotated method) prior to starting the resource. The `start` method is responsible for starting the resource and returning a `ResourceInstance` that encapsulates a server component and a client component that is configured to communicate with the server. In this example our resource instance happens to comprises of an in-memory HSQL JDBC DataSource (server) and JDBC connection (client). Finally, the stop method is responsible for gracefully stopping the server and closing the client.

Now that we have a resource provider implementation we can start using it to provide a database resource to our application during integration and system tests by annotating our test class with `@RequiresResource(InMemoryHSQLResource.class)`:

{% highlight java linenos=table %}
@Module(GreetingConfig.class)
@RequiresResource(InMemoryHSQLResource.class)
@RunWith(SpringIntegrationTest.class)
public class GetGreetingIT {

    @Cut
    GetGreeting cut;

    @Real
    GreetingRepository greetingRepository;

    @Inject
    ResourceInstance<DataSource, Connection> resourceInstance;

    @Inject
    DataSource dataSource;

    @Inject
    Connection connection;

    @ConfigHandler
    void configure(JDBCDataSource dataSource) {
        //XXX: You can refine the configuration object of our
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

Notice that we are able to inject the resource instance, data source and connection created in our `InMemoryHSQLResource` resource provider implementation which can be useful if you wish to interact with the server and the client. Please note that the above example is kitchen-sink example and may not reflect a typical use-case.


For complete ResourceProvider implementation example please take a look at the below example and be sure to read the JavaDocs:
[Example JUnit Resource Provider][example-junit-resourceprovider].


---

## Docker Container Resources

---

### Installing and Configuring Docker

#### [Install Docker](#install-docker)

Before you can test your application with Docker container based needs you must
install Docker on your system. To install Docker please follow the
[Docker instalation instructions](https://docs.docker.com/engine/installation/).

#### Configuring Docker

Testify uses Docker Remote API to pull images and manage Docker containers. By
default Docker Remote API is not enabled. To enable it insure that `DOCKER_OPTS`
and `DOCKER_TLS_VERIFY` environmental variable are set to the following:

{% highlight bash linenos=table %}
export DOCKER_OPTS=" -H tcp:/127.0.0.1:2375 -H unix:///var/run/docker.sock"
export DOCKER_TLS_VERIFY=0
{% endhighlight %}

These two environmental variable must be set prior to executing your tests. You
can set these variables permanently through
[Docker Configuration][docker-configuration] file:

**Mint / Ubuntu / Debian Linux**

{% highlight bash linenos=table %}
/etc/default/docker
{% endhighlight %}

**Fedora / Red Hat / CentOS / Oracle**

{% highlight bash linenos=table %}
/etc/sysconfig/docker
{% endhighlight %}

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

#### Notes on Docker Remote API and Testify

By default Testify communicates with Docker through Docker Remote API on
non-secure `http://127.0.0.1:2375` URL endpoint. If you wish to use an endpoint
with a different IP address and port or are using a Docker-Machine, or want to
use secure-communication then you will need to explictly configure the Docker
Client used by Testify in each of your test classe (not recommended):
{% highlight java linenos=table %}
@Config
public void configure(DockerClientConfig.DockerClientConfigBuilder builder) {
    builder.withUri("http://192.168.99.100:2376");
}
{% endhighlight %}


[example-junit-resourceprovider]: https://github.com/testify-project/examples/tree/master/junit/example-junit-resourceprovider
[docker-configuration]: https://docs.docker.com/engine/articles/configuring
[docker-mac-install]: https://docs.docker.com/engine/installation/mac/
[docker-windows-install]: https://docs.docker.com/engine/installation/windows/
---
title: "Getting Started"
bg: fnavy
color: white
fa-icon: toggle-on
---

## Configuration Checklist
- Latest release version is [{{ site.testify_version }}][maven-central]
- Take a look at the [release notes][releases]
- Install JDK version {{ site.jdk_version }} and insure `JAVA_HOME` environmental variable is set
- Install Maven version {{ site.maven_version }} or above
- Install Git version {{ site.git_version }}
- (Optional) [Install Docker version](#install-docker) {{ site.docker.version }}
- Insure formal parameter names of constructors and methods are added to
compiler generated class files:
{% highlight xml linenos=table %}
<plugin>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
        <source>{{ site.java_version }}</source>
        <target>{{ site.java_version }}</target>
        <compilerArguments>
            <!-- Enable runtime discovery of parameter names -->
            <parameters />
        </compilerArguments>
    </configuration>
</plugin>
{% endhighlight %}

[releases]: https://github.com/testify-project/testify/releases
[maven-central]: http://repo1.maven.org/maven2/org/testify

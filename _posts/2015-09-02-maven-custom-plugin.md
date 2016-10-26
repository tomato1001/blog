---
layout: post
title: maven自定义插件
tags: ['maven plugin']
date: 2015-09-02 10:09:38
---

maven

{% highlight java %}
@Mojo(name = "build", defaultPhase = LifecyclePhase.COMPILE, requiresDependencyResolution = ResolutionScope.COMPILE)
public class BuilderMojo extends AbstractMojo {

    @Parameter(defaultValue = "${project.basedir}")
    private File projectDirectory;

    @Parameter(defaultValue = "${project.build.outputDirectory}")
    private File outputDirectory;

    @Parameter(required = true, alias = "profiles")
    private List<Profile> profilesConfig;

    @Parameter(property = "supervisor.expiredDate", defaultValue = "2999-12-31 23:59:59")
    private String expiredDate;

    /**
     * The activated profiles
     */
    @Parameter(property = "supervisor.profiles")
    private String activeProfileStr;

    /**
     * Defaults profiles if {@link #activeProfileStr} is empty or null
     */
    @Parameter(defaultValue = "production")
    private String defaultProfiles;

    @Parameter(defaultValue = "${project.build.outputDirectory}")
    private File persistProfileDir;

    @Parameter
    private List<Replace> replaces;

    private Replacer replacer;

    @Parameter(defaultValue = "${plugin}", readonly = true)
    private PluginDescriptor pluginDescriptor;

    @Parameter(defaultValue = "${project}", readonly = true)
    private MavenProject currentProject;

    @Parameter(defaultValue = "${session}", readonly = true)
    private MavenSession session;

    private List<String> activeProfileList;

    private List<String> excludeProfileList;

    public void execute() throws MojoExecutionException, MojoFailureException {
		// implementation here
    }
}
{% endhighlight %}

{% highlight xml %}

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>custom</artifactId>
        <groupId>com.xxx</groupId>
        <version>0.0.1</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>build-plugin</artifactId>
    <packaging>maven-plugin</packaging>

    <dependencies>
        <dependency>
            <groupId>org.apache.maven</groupId>
            <artifactId>maven-core</artifactId>
            <version>3.3.3</version>
        </dependency>

        <dependency>
            <groupId>org.apache.maven</groupId>
            <artifactId>maven-plugin-api</artifactId>
            <version>3.3.3</version>
        </dependency>

        <!-- dependencies to annotations -->
        <dependency>
            <groupId>org.apache.maven.plugin-tools</groupId>
            <artifactId>maven-plugin-annotations</artifactId>
            <version>3.4</version>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.4</version>
        </dependency>

        <dependency>
            <groupId>com.arcsoft</groupId>
            <artifactId>supervisor-utils</artifactId>
            <version>0.0.1</version>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-plugin-plugin</artifactId>
                <version>3.4</version>
                <executions>
                    <execution>
                        <id>default-descriptor</id>
                        <phase>process-classes</phase>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>

{% endhighlight %}
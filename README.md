[![Build Status](https://travis-ci.org/ulisesbocchio/spring-boot-jar-resources.svg?branch=master)](https://travis-ci.org/ulisesbocchio/spring-boot-jar-resources)
[![Gitter](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/ulisesbocchio/spring-boot-jar-resources?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)
[![Maven Central](https://maven-badges.herokuapp.com/maven-central/com.github.ulisesbocchio/spring-boot-jar-resources/badge.svg?style=plastic)](https://maven-badges.herokuapp.com/maven-central/com.github.ulisesbocchio/spring-boot-jar-resources)

# spring-boot-jar-resources

[![Join the chat at https://gitter.im/ulisesbocchio/spring-boot-jar-resources](https://badges.gitter.im/ulisesbocchio/spring-boot-jar-resources.svg)](https://gitter.im/ulisesbocchio/spring-boot-jar-resources?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
When using Spring Boot out of the box, resources from classpath are jarred, and while they can be accessed through input streams, they cannot be accessed as Files. Some libraries require Files as input instead of input streams or Spring Resources. This library deals whith that limitation by allowing you to do resource.getFile() on any jarred resource. It does so by extracting the file from the jar to a temporary location.

## How to use this library?

Simply add the following dependency to your project:

```xml
<dependency>
	<groupId>com.github.ulisesbocchio</groupId>
	<artifactId>spring-boot-jar-resources</artifactId>
	<version>1.1</version>
</dependency>
```

And the following configuration to your Spring Boot app:

```java
new SpringApplicationBuilder()
            .sources(Application.class)
            .resourceLoader(new JarResourceLoader())
            .run(args);
```

Alternatively, provide a path to the JarResourceLoader where jarred resources will be extracted when accessed through a File handle.

```java
new SpringApplicationBuilder()
            .sources(Application.class)
            .resourceLoader(new JarResourceLoader("/path/to/extract"))
            .run(args);
```

If you want to expose the path to be configurable, you can use this trick to get it either from the command running the spring boot app, a system property, or an environment variable (with "." for "_" and lower to UPPER case conversion)

```java
public static void main(String[] args) throws Exception {
    new SpringApplicationBuilder()
        .sources(Application.class)
        .resourceLoader(new JarResourceLoader(getExtractDir(args)))
        .run(args);
}

static String getExtractDir(String[] args) {
    return new StandardEnvironment() {
        @Override
        protected void customizePropertySources(MutablePropertySources propertySources) {
            super.customizePropertySources(propertySources);
            propertySources.addLast(new SimpleCommandLinePropertySource("cmd", args));
        }
    }.getProperty("resources.extract.dir");
}
```

With this you can run you `app.jar` this ways:

* `java -Dresources.extract.dir=/some/path -jar app.jar`
* `java -jar app.jar resources.extract.dir=/some/path`
* `export RESOURCES_EXTRACT_DIR=/some/path && java -jar app.jar`

It won't work if you try to put `resources.extract.dir` in `application.properties` since this is too early in the game to access those properties from Spring and after all we're creating our own customized environment with a command line arguments property source to get the property we want.  If you want to put this property in `application.properties` you can add after the `SimpleCommandLinePropertySource` another property source for `application.properties` like this:

`propertySources.addLast(new ResourcePropertySource("app.props", new ClassPathResource("application.properties")));`

It'd be a different property source if you use YAML.

## How this library works?

Internally, this library simply wraps existing resources loaded by DefaultResourceLoader with a custom JarResource implementation that deals with the details of extracting the resource from the Jar. The implementation only extracts resources from jars if they need to be extracted, i.e. if actually being inside a jar. If for some reason, such as when running within an IDE or using an absolute path to load resources, the resources are not inside a jar, then the actual file is used instead.

## 

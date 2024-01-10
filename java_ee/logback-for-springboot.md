---
description: logback
---

# Logback for SpringBoot

## 1. Initial Setup

```java
@RestController
public class LoggingController {

    Logger logger = LoggerFactory.getLogger(LoggingController.class);

    @RequestMapping("/")
    public String index() {
        logger.trace("A TRACE Message");
        logger.debug("A DEBUG Message");
        logger.info("An INFO Message");
        logger.warn("A WARN Message");
        logger.error("An ERROR Message");

        return "Howdy! Check out the Logs to see the output...";
    }
}

```

Once we’ve loaded the web application, **we’ll be able to trigger those logging lines by simply visiting **_**http://localhost:8080/**_**.**

## 2. Zero Configuration Logging

关于 jar 包依赖

**We shouldn’t worry about importing **_**spring-jcl**_** at all if we’re using a Spring Boot Starter** (which we almost always are). That’s because every starter, like our _spring-boot-starter-web_, depends on _spring-boot-starter-logging,_ which already pulls in _spring-jcl_ for us.

### 2.1 Default Logback Logging

**When using starters, Logback is used for logging by default.**

Spring Boot preconfigures it with patterns and ANSI colors to make the standard output more readable.

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

As we can see, **the default logging level of the Logger is preset to INFO, meaning that **_**TRACE**_** and **_**DEBUG**_** messages are not visible.**

In order to activate them without changing the configuration, **we can pass the **_**–debug**_** or **_**–trace**_** arguments on the command line**:

```bash
java -jar target/demo-1.0.jar --debug
```

### **2.2 Log Levels**

Spring Boot also gives us access to a more fine-grained log level setting.

First, we can set our logging level within our <mark style="color:orange;">**VM Options**</mark>:

```bash
-Dlogging.level.org.springframework=DEBUG
-Dlogging.level.com.baeldung=DEBUG
```

Alternatively, if we’re using Maven, we can **define our log settings via the** <mark style="color:yellow;">command line</mark>:

```bash
mvn spring-boot:run
  -Dspring-boot.run.arguments=--logging.level.org.springframework=DEBUG,--logging.level.com.baeldung=DEBUG
```

If we want to change the verbosity permanently, we can do so in the _<mark style="background-color:green;">application.properties</mark>_ file as described [here](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-logging.html#boot-features-custom-log-levels):

```plaintext
logging.level.root=WARN
logging.level.com.baeldung=DEBUG
```

Finally, we can **change the logging level permanently by using our logging framework **<mark style="color:red;">**configuration file.**</mark>

We mentioned that Spring Boot Starter uses Logback by default. Let’s see how to define a fragment of a Logback configuration file in which we set the level for two separate packages:

```markup
<logger name="org.springframework" level="INFO" />
<logger name="com.baeldung" level="INFO" />
```

这几个选项，最终日志级别会是最低的



## 3. <mark style="color:red;">Logback Configuration</mark>

Let’s see **how to include a Logback configuration** with a different color and logging pattern, with separate specifications for _console_ and _file_ output, and with a decent _rolling policy_ to avoid generating huge log files.

**When a file in the classpath has one of the following names, Spring Boot will automatically load it** over the default configuration:

* _logback-spring.xml_
* _logback.xml_
* _logback-spring.groovy_
* _logback.groovy_






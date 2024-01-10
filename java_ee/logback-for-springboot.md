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
java -jar target/demo-1.0.jar --trace#
```

### **2.2 Log Levels**

Spring Boot also gives us access to a more fine-grained log level setting.

First, we can set our logging level within our <mark style="color:orange;">**VM Options**</mark>:

```bash
-Dlogging.level.org.springframework=TRACE
-Dlogging.level.com.baeldung=TACE
```

Alternatively, if we’re using Maven, we can **define our log settings via the command line**:






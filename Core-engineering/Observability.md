
---

Application logs and webserver logs are monitored at centralized place along with system usage.

Why centralized logging and monitoring:

1. Rapid incident response and root cause analysis: Usually if server fails to keep application keep and running it will be passed to passive server. And then analysis happens. 
2. Performance optimization and bottleneck identification: By observing metrics we can also plan for capacity way ahead.

### Implementations:

1. ELK (Elasticsearch , Logstash , Kibana): Logstash - collects and process. Elastic search provides fast and scalable log. Kibana offers interface for data visualization.
2. Splunk: Collects and alert mechanism. 
3. Datadog: Cloud based monitoring platform real-time insights into your application.

### Basics of logging:

### Logging Mechanism (Framework)

The "logging mechanism" refers to the overall framework or library that you use to handle logging in your application. This framework provides a standardized API for developers to write log messages without worrying about how or where those messages will be stored.

Key components of a logging framework include:

- **Loggers:** These are the objects that your application code interacts with directly. You obtain a `Logger` instance for a specific class or component and use its methods (e.g., `info()`, `debug()`, `error()`) to record events.
    
- **Logging Levels:** Every log message is assigned a severity level (e.g., `TRACE`, `DEBUG`, `INFO`, `WARN`, `ERROR`, `FATAL`). The logging framework uses these levels to filter messages, ensuring that only relevant logs are processed based on the configured level. For example, if you set the logging level to `INFO`, messages with a level of `DEBUG` or `TRACE` will be ignored.
    
- **Filters:** These provide a more granular way to control which log events are processed. You can define custom filters to accept or deny log events based on criteria beyond just the log level, such as the log message content or the originating class.
    
- **Layouts/Encoders:** These components are responsible for formatting the log message. They take the raw log event and convert it into a structured format, such as a simple string, XML, or JSON, before it's sent to its destination.
    

Some of the most popular logging frameworks in Java are:

- **Logback:** A modern, fast, and feature-rich framework. It's often used with **SLF4J**.
    
- **Log4j2:** A powerful and highly performant framework that is the successor to Log4j 1.x. It's known for its asynchronous logging capabilities.
    
- **SLF4J (Simple Logging Facade for Java):** This is a facade, not a logging implementation itself. It provides a simple API that acts as a bridge, allowing you to switch between different logging frameworks (like Logback or Log4j2) without changing your application code. This is considered a best practice.
    
- **`java.util.logging` (JUL):** The built-in logging mechanism that comes with the JDK. It's a solid choice for simple applications but may lack some of the advanced features of external frameworks.
    

### Appenders

An "appender" is the part of the logging mechanism that is responsible for sending the formatted log messages to their final destination. It answers the question, "Where should this log go?"

A logging framework can have multiple appenders configured, allowing a single log event to be sent to different places simultaneously. For example, you could have one appender sending `INFO` level logs to a file, and another appender sending only `ERROR` and `FATAL` logs to an external service like an email server.

Common types of appenders include:

- **Console Appender:** Writes log messages to the standard output (`System.out`) or standard error (`System.err`). This is often used during development and for simple applications.
    
- **File Appender:** Writes log messages to a file on the local filesystem.
    
- **Rolling File Appender:** An enhanced version of the file appender. It automatically creates new log files (or "rolls" them) based on criteria such as file size or a time-based schedule (e.g., daily). It also manages the archiving of old log files.
    
- **Socket Appender:** Sends log messages over a network connection (e.g., TCP or UDP) to a remote server. This is the mechanism used to send logs to a tool like Logstash.
    
- **Database Appender (JDBC Appender):** Logs messages to a relational database.
    
- **Async Appender:** A special kind of appender that runs on a separate, low-priority thread. It processes log events asynchronously, which helps to minimize the performance impact of logging on the main application thread.
    
- **SMTP Appender:** Sends log messages as an email, typically for high-severity events like `ERROR` or `FATAL`.

```groovy
<?xml version="1.0" encoding="UTF-8"?>  
<configuration>  
    <include resource="org/springframework/boot/logging/logback/base.xml" />  
    <appender name="LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">  
        <destination>localhost:5000</destination>  
        <encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">  
            <providers>                <timestamp>                    <timeZone>UTC</timeZone>  
                </timestamp>                <pattern>                    <pattern>                        {  
                        "severity": "%level",  
                        "service": "spring-centralized-logging",  
                        "trace": "%X{X-B3-TraceId:-}",  
                        "span": "%X{X-B3-SpanId:-}",  
                        "exportable": "%X{X-Span-Export:-}",  
                        "pid": "${PID:-}",  
                        "thread": "%thread",  
                        "class": "%logger{40}",  
                        "message": "%message"  
                        }  
                    </pattern>  
                </pattern>                <arguments/>                <stackTrace/>            </providers>        </encoder>    </appender>    <root level="INFO">  
        <appender-ref ref="LOGSTASH"/>  
    </root></configuration>
```

### Prometheus and ELK:

 
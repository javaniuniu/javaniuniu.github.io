---
title: logback配置
permalink: /docs-util/logback
tags: CodeMark 工具类
key: docs-util-logback
---
#### logback-spring.xml 配置      
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- scan 配置文件如果发生改变，将会被重新加载  scanPeriod 检测间隔时间-->
<configuration scan="true" scanPeriod="60 seconds" debug="false">
    <contextName>logback</contextName>
    <include resource="org/springframework/boot/logging/logback/base.xml"/>
    <!-- 日志存储级别 -->
    <springProperty scope="context" name="rootlevel" source="logging.level.root" />
    <springProperty scope="context" name="busilevel" source="logging.level.com.xncoding" />
    <!-- 日志存储路径 -->
    <springProperty scope="context" name="logPath" source="logging.path" />
    <!-- 普通日志 -->
    <appender name="INFO_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${logPath}/info.log</file>
        <!-- 循环政策：基于时间创建日志文件 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 日志命名:单个文件大于128MB 按照时间+自增i 生成log文件 -->
            <fileNamePattern>${logPath}/info-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>128MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!-- 最大保存时间：30天-->
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <append>true</append>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS}  %p ${PID:-} --- [%15thread] %logger:%-3L : %msg%n</pattern>
            <charset>utf-8</charset>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>info</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
    <!-- 错误日志 -->
    <appender name="ERROR_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${logPath}/error.log</file>
        <!-- 循环政策：基于时间创建日志文件 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 日志命名:单个文件大于2MB 按照时间+自增i 生成log文件 -->
            <fileNamePattern>${logPath}/error-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>2MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!-- 最大保存时间：180天-->
            <maxHistory>180</maxHistory>
        </rollingPolicy>
        <append>true</append>
        <!-- 日志格式 -->
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS}  %p ${PID:-} --- [%15thread] %logger:%-3L : %msg%n</pattern>
            <charset>utf-8</charset>
        </encoder>
        <!-- 日志级别过滤器 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!-- 过滤的级别 -->
            <level>ERROR</level>
            <!-- 匹配时的操作：接收（记录） -->
            <onMatch>ACCEPT</onMatch>
            <!-- 不匹配时的操作：拒绝（不记录） -->
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
    <!-- 控制台 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <!-- 日志格式 -->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS}  %p ${PID:-} --- [%15thread] %logger:%-3L : %msg%n</pattern>
            <charset>utf-8</charset>
        </encoder>
        <!--此日志appender是为开发使用，只配置最底级别，控制台输出的日志级别是大于或等于此级别的日志信息-->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>DEBUG</level>
        </filter>
    </appender>
    <!-- 屏蔽kafka的警告 -->
    <logger name="org.apache.kafka" level="ERROR"/>
    <!-- additivity 避免执行2次 -->
    <logger name="com.xncoding" level="${busilevel}" additivity="false">
        <appender-ref ref="STDOUT"/>
        <appender-ref ref="INFO_FILE"/>
        <appender-ref ref="ERROR_FILE"/>
    </logger>
    <root level="${rootlevel}">
        <appender-ref ref="INFO_FILE"/>
        <appender-ref ref="ERROR_FILE"/>
    </root>
</configuration>
```

#### application.yml 配置   
```xml
##########################################################
##################  所有profile共有的配置  #################
##########################################################

###################  项目启动端口  ###################
server.port: 8092

###################  spring配置  ###################
spring:
  profiles:
    active: dev

---

#####################################################################
########################  开发环境profile  ##########################
#####################################################################
spring:
  profiles: dev

logging:
  level:
    root: INFO
    com.xncoding: DEBUG
  path: D:/logs/springboot-aop

```

参考链接
- **logback** `继承自 log4j`，它建立在有十年工业经验的日志系统之上。它 `比其它所有的日志系统更快并且更小`，包含了许多独特并且有用的特性。[详细参考文档](http://www.logback.cn) [logback 常用配置（详解）](https://blog.csdn.net/qq_36850813/article/details/83092051)

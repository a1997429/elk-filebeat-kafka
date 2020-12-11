# elk-filebeat-kafka
elk-filebeat-kafka 日志处理

logback.xml

<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
    <springProperty scope="context" name="springAppName" source="spring.application.name"/>
    <springProperty scope="context" name="logPatternLevel" source="logging.pattern.level"/>
    <springProperty scope="context" name="consoleLevel" source="logging.level.console"/>
    <property name="LOG_ROOT" value="${LOG_ROOT:-logs}/${springAppName}"/>

    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>${consoleLevel:-all}</level>
        </filter>
        <encoder>
            <pattern>%clr(%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}</pattern>
        </encoder>
    </appender>
    <conversionRule conversionWord="ip" converterClass="*****.LogbackIPConfig" />

    <appender name="logstash"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_ROOT}/elk.log</file>
        <rollingPolicy
                class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_FILE}.json.%d{yyyy-MM-dd}.gz</fileNamePattern>
            <maxHistory>7</maxHistory>
        </rollingPolicy>
        <encoder
                class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
            <providers>
                <timestamp>
                    <timeZone>UTC</timeZone>
                </timestamp>
                <pattern>
                    <pattern>
                        {
                        "hostIp":"%ip",
                        "level":"%level",
                        "service": "${springAppName:-}",
                        "trace": "%X{X-B3-TraceId:-}",
                        "span": "%X{X-B3-SpanId:-}",
                        "stack_trace": "%exception{10}",
                        "req_id": "%X{reqId}",
                        "elapsed_time": "#asLong{%X{elapsedTime}}",
                        "pid": "${PID:-}",
                        "thread": "%thread",
                        "class": "%logger{40}",
                        "method_name":"%logger{40}.%method",
                        "method":"%method",
                        "message": "%message"
                        }
                    </pattern>
                </pattern>
            </providers>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="console"/>
        <appender-ref ref="logstash"/>
    </root>
</configuration>

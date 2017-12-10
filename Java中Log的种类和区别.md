# Java中Log的种类和区别

## JDK内置Logger

1. 支持七种Logger级别，从高到低依次为：

    SEVERE->WARNING->INFO->FINE->FINER- >FINEST   
2. 支持的Formatter
    支持SimpleFormatter和XMLFormatter，前者文本形式记录日志信息，后者XML格式
3. 支持的Handler

    主要为MemoryHandler和StreamHandler，ConsoleHandler，FileHandler和SocketHandler都继承自StreamHandler
    
## Log4J2 

1. 支持五种Logger级别，分别为：
    FATAL->ERROR->WARN->INFO->DEBUG->TRACE  
2. Handler与前者类似
3. Log4J2中的每一个log上下文对应一个configuration，configuration详细描述了Log系统的各个LoggerConfig、Appender、EventLog过滤器等。
4. 异步日志系统比Log4j1和Logback性能更好
5. 重新配置的时候不会丢失之前的日志文件

## Logback 

1. 是一个通用，快速的日志框架
2. 分为`logback-core`,`logback-classic`,`logback-access`三个模块，其中，`logback-core`为基础模块，`logback-classic`为log4j的改良版，完整实现了SLF4J API，可以更方便地更换成其他日志系统，`logback-access`与servlet集成提供通过http访问日志


## Commons Logging

是一种通用日志工具包

## SLF4J

1. 是一种简单的日志门面，而不是日志解决方案。通过Facade Pattern提供一些Java logging API
2. 适用于类库或是嵌入式组件的开发，因为不可能影响最终用户选择哪种系统日志
3. 用来代替Commons Logging


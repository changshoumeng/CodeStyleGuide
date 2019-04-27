## 阿里java编程规范-日志部分

### 日志规约
    【强制】应用中不可直接使用日志系统（Log4j、Logback）中的API，而应依赖使用日志框架SLF4J中的API，使用门面模式的日志框架，
           有利于维护和各个类的日志处理方式统一。
           
          import org.slf4j.Logger;
          import org.slf4j.LoggerFactory;
              private static final Logger logger = LoggerFactory.getLogger(Abc.class);        
          slf4j是日志门面框架，其仅提供日志记录的API，而不实现日志记录的功能，slf4j需要通过适配库适配到log4j或logback等
          日志系统来实现日志的记录。使用slf4j api能够提升代码和应用的可移植性，在使用不同日志系统的应用之间能够做到无缝的适配。
          同时，使用slf4j api的应用，在切换日志系统时（比如从logback切换到log4j2，不需要代码改造）
    
    【强制】日志文件推荐至少保存15天，因为有些异常具备以“周”为频次发生的特点。
    
    【强制】应用中的扩展日志（如打点、临时监控、访问日志等）命名方式：appName_logType_logName.log。
           logType:日志类型，推荐分类有stats/desc/monitor/visit等；
           logName:日志描述。这种命名的好处：通过文件名就可知道日志文件属于什么应用，什么类型，什么目的，也有利于归类查找。
           正例：mppserver应用中单独监控时区转换异常，如： mppserver_monitor_timeZoneConvert.log
           说明：推荐对日志进行分类，如将错误日志和业务日志分开存放，便于开发人员查看，也便于通过日志对系统进行及时监控。
    
    【强制】对trace/debug/info级别的日志输出，必须使用条件输出形式或者使用占位符的方式。
           说明：logger.debug("Processing trade with id: " + id + " symbol: " + symbol); 
           如果日志级别是warn，上述日志不会打印，但是会执行字符串拼接操作，
           如果symbol是对象，会执行toString()方法，浪费了系统资源，执行了上述操作，最终日志却没有打印。
           
           正例：（条件）
               if (logger.isDebugEnabled()) { 
                   logger.debug("Processing trade with id: " + id + " symbol: " + symbol); 
                 }      
           正例：（占位符）
               logger.debug("Processing trade with id: {} symbol : {} ", id, symbol);     
          占位符方式，log4j2/logback支持，log4j1.x是不直接支持的，只能通过slf4j库适配
    
    【强制】避免重复打印日志，浪费磁盘空间，务必在log4j.xml中设置additivity=false。
            正例：
               additivity默认为true，即通过该logger输出的日志会同时输出到root logger，如果还为该logger指定了
               独立的appender，就会导致这部分日志重复输出
    
    【强制】异常信息应该包括两类信息：案发现场信息和异常堆栈信息。如果不处理，那么通过关键字throws往上抛出。
            正例：
                logger.error(各类参数或者对象toString + "_" + e.getMessage(), e);             
          记录异常日志的常见错误：
            logger.error(e);
            logger.error(e.getMessage());
            logger.error("上下文"+e.getMessage())；        
          上面这几种都是错的！请确保使用的是两个入参的API，如error(String s, Throwable t)
    
    【推荐】谨慎地记录日志。生产环境禁止输出debug日志；有选择地输出info日志；如果使用warn来记录刚上线时的业务行为信息，
            一定要注意日志输出量的问题，避免把服务器磁盘撑爆，并记得及时删除这些观察日志。
            说明：大量地输出无效日志，不利于系统性能提升，也不利于快速定位错误点。记录日志时请思考：这些日志真的有人看吗？
            看到这条日志你能做什么？能不能给问题排查带来好处？不要认为日志记录不怎么消耗性能，我见过不少事无巨细式的日志
            把系统性能严重拖慢的案例
    
    【参考】可以使用warn日志级别来记录用户输入参数错误的情况，避免用户投诉时，无所适从。注意日志输出的级别，error级别只记
    录系统逻辑出错、异常等重要的错误
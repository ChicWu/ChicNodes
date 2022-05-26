---
title: Logger 基本用法
date: 2018-09-04 20:04:54
top: 8
tags:
---
　　简单的使用Logger进行日志处理
<!-- more -->
# 一、创建Logger对象
`static Logger getLogger(String name) `
　　为指定子系统查找或创建一个 logger。
`static Logger getLogger(String name, String resourceBundleName) `
　　为指定子系统查找或创建一个 logger。
注意：name是Logger的名称，当名称相同时候，同一个名称的Logger只创建一个。
# 二、Logger的级别
　　比log4j的级别详细，全部定义在java.util.logging.Level里面。各级别按降序排列如下：
>* SEVERE（最高值）
* WARNING
* INFO(默认)
* CONFIG
* FINE
* FINER
* FINEST（最低值）

　　此外，还有一个级别 OFF，可用来关闭日志记录，使用级别 ALL 启用所有消息的日志记录。
logger默认的级别是INFO，比INFO更低的日志将不显示。
# 三、Logger的Handler
　　创建方法:
　　`FileHandler fileHandler = new FileHandler(FILE_PATH, true);`
　　`FILE_PATH:日志文件的路径，true：是否才用追加的方式写入日志文件`
　　Handler 对象从 Logger 中获取日志信息，并将这些信息导出。例如，它可将这些信息写入控制台或文件中，也可以将这些信息发送到网络日志服务中，或将其转发到操作系统日志中。
　　可通过执行 setLevel(Level.OFF) 来禁用 Handler，并可通过执行适当级别的 setLevel 来重新启用。
　　Handler 类通常使用 LogManager 属性来设置 Handler 的 Filter、Formatter 和 Level 的默认值
# 四、Logger的Formatter
　　Formatter 为格式化 LogRecords 提供支持。 
　　一般来说，每个日志记录 Handler 都有关联的 Formatter。Formatter 接受 LogRecord，并将它转换为一个字符串。 
　　有些 formatter（如 XMLFormatter）需要围绕一组格式化记录来包装头部和尾部字符串。可以使用 getHeader 和 getTail 方法来获得这些字符串。
　　LogRecord 对象用于在日志框架和单个日志 Handler 之间传递日志请求。
　　`LogRecord(Level level, String msg)`
　　Level用给定级别和消息值构造 LogRecord。
# 五、简单案例
　　日志处理类
```
    public class MyLog {
        private static Logger logger = Logger.getLogger(MyLog.class.getName());

        private static String FILE_PATH = "E:\\MyLog\\winfo.log";
        static {
            try {
                FileHandler fileHandler = new FileHandler(FILE_PATH, true);
                fileHandler.setEncoding("utf-8");
                MyLogFormatter sf = new MyLogFormatter();
                fileHandler.setFormatter(sf);
                logger.addHandler(fileHandler);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }

        public static void error(Object obj) {
            //严重
            LogRecord lr = new LogRecord(Level.SEVERE, obj.toString());
            logger.log(lr);
        }

        public static void warn(Object obj) {
            //警告
            LogRecord lr = new LogRecord(Level.WARNING, obj.toString());
            logger.log(lr);
        }
        public static void info(Object obj) {
            //信息
            LogRecord lr = new LogRecord(Level.INFO, obj.toString());
            logger.log(lr);
        }

    }
```
　　日志信息处理类
```
    class OTLLogFormatter extends Formatter {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy/MM/dd HH:mm:ss");

        @Override
        public String format(LogRecord record) {
            return "[" + sdf.format(new Date()) + "] [" + record.getLevel() + "] "
                    + record.getMessage().replace("\r", " ").replace("\n", " ") + "\r\n";
        }
    }
```
　　测试案例
```
	public class LogTest {
        @Test
        public void test0(){
            OTLLog.error("严重信息测试");
        	OTLLog.warn("警告信息测试");
        	OTLLog.info("信息输出测试");
        }
	}
```
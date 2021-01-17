# log

```yml
logging:
  pattern:
    console: "%d - %msg%n" #     定义打印的日志格式
#    dateformat: #设置日志日期格式
#    file: #定义输出到日志文件的日志格式
#  config: #日志配置文件的位置。例如，classpath:logback.xml。
  file: E:/sell/sell.log #设置保存日志的日志文件
#    max-history:
#    max-size: #设置日志文件最大大小 #设置日志等级
#  path: / #日志文件的位置，例如/var/log
  register-shutdown-hook: false #当初始化日志系统时，为其注册一个关闭钩子。
  level:
    root: info
```
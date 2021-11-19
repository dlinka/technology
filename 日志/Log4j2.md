```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- monitorInterval:检测配置文件是否修改并重新配置,单位是秒 -->
<Configuration monitorInterval="5">
  
	<Properties>
    	<!-- 
					%date:日期
					%thread:线程名
					%-5level:日志级别从左显示5个字符宽度
					%logger{36}:Logger名字最长36个字符
					%msg:日志消息
					%n:换行符
 			-->
  		<property name="LOG_PATTERN" value="%date{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n" />
    	<property name="LOG_PATH" value="日志路径" />
  </Properties>
  
  <Appenders>
    <!-- 自定义Appender -->
    <Log4j2Appender name="CustomAppender" appName="test_address_service_appender">
    
    <!-- 输出到控制台 -->
  	<Console name="Console-Appender" target="SYSTEM_OUT">
      <PatternLayout pattern="${LOG_PATTERN}" />
      <!-- 
					onMatch="ACCEPT":匹配该级别及以上
					onMatch="DENY":不匹配该级别及以上
					onMatch="NEUTRAL":该级别及以上的,由下一个filter处理,如果当前是最后一个,则表示匹配该级别及以上
					onMismatch="ACCEPT":匹配该级别以下
					onMismatch="DENY":不匹配该级别以下的
					onMismatch="NEUTRAL":该级别及以下的,由下一个filter处理,如果当前是最后一个,则不匹配该级别以下的
			-->
      <!-- 下面的配置只打印INFO和DEBUG日志 -->
      <Filters>
      	<ThresholdFilter level="WARN" onMatch="DENY" onMismatch="NEUTRAL"/>
				<ThresholdFilter level="DEBUG" onMatch="ACCEPT" onMismatch="DENY"/>
      </Filters>
    </Console>
    
    <!-- 输出到文件 -->
    <!-- append:等于false表示每次运行程序这个文件自动清空 -->
    <File name="File-Appender" fileName="${LOG_PATH}/test.log" append="false">
      <PatternLayout pattern="${LOG_PATTERN}" />
      <!-- 不设置ThresholdFilter表示打印所有级别 -->
    </File>
    
    <!-- 滚动文件 -->
    <RollingFile name="RollingFile-Appender" fileName="${LOG_PATH}/info.log" filePattern="${FILE_PATH}/${FILE_NAME}-INFO-%d{yyyy-MM-dd HH}_%i.log.gz">
      <PatternLayout pattern="${LOG_PATTERN}" />
      <ThresholdFilter level="INFO" onMatch="ACCEPT" onMismatch="DENY" />
      <Policies>
        <!-- 每小时滚动一次 -->
        <TimeBasedTriggeringPolicy interval="1" modulate="true"/>
        <!-- 文件到达10MB滚动 -->
        <SizeBasedTriggeringPolicy size="10MB" />
      </Policies>
      <!-- 每小时最多保留15个,默认是7个 -->
      <DefaultRolloverStrategy max="15" />
    </RollingFile>
    
    <!-- 滚动文件(底层使用RandomAccessFile) -->
    <RollingRandomAccessFile name="RollingRandomAccessFile-Appender" fileName="${LOG_PATH}/error.log" filePattern="${FILE_PATH}/${FILE_NAME}-ERROR-%d{yyyy-MM-dd HH}_%i.log.gz">
			<PatternLayout pattern="${LOG_PATTERN}" />
			<Policies>
        <!-- 每小时滚动一次 -->
      	<TimeBasedTriggeringPolicy interval="1" modulate="true"/>
				<!-- 文件到达10MB滚动 -->
        <SizeBasedTriggeringPolicy size="10MB" />
			</Policies>
			<DefaultRolloverStrategy>
				<Delete basePath="${LOG_PATH}">
        	<IfFileName glob="${FILE_NAME}-ERROR-*.log.gz"/>
          <IfLastModified age="14d"/>
        </Delete>
      </DefaultRolloverStrategy>
		</RollingRandomAccessFile>

  </Appenders>

  <Loggers>
    <!-- 同步Logger配置 -->
    <Logger name="org.mybatis" level="INFO" additivity="false">
    	<AppenderRef ref="Console-Appender" />
    </Logger>
    <Root level="INFO">
    	<AppenderRef ref="Console-Appender" />
      <AppenderRef ref="File-Appender" />
      <AppenderRef ref="RollingFile-Appender" />
    </Root>
    
    <!-- 异步Logger配置 -->
    <!-- 是否显示文件行数,若为异步Logger,开启此项会造成性能影响 -->
    <AsyncLogger name="org.mybatis" level="INFO" additivity="false" includeLocation="true">
      <AppenderRef ref="Console-Appender" level="ERROR" />
    </AsyncLogger>
    <AsyncRoot level="INFO" includeLocation="true">
      <AppenderRef ref="Console-Appender" />
			<AppenderRef ref="File-Appender" />
			<AppenderRef ref="RollingFile-Appender" />
    </AsyncRoot>
  </Loggers>
</Configuration>
```

---

### RollingFile

日志写入app.log文件中,每经过1小时或者当文件大小到达250M时,按照app-2017-08-01 12.log的格式对app.log进行重命名并归档,并生成新的app.log文件用于写入log

```xml
<Configuration>
  <Appenders>
    <!-- RollingFile -->
		<!-- fileName指定日志文件的位置和文件名称 -->
		<!-- filePattern指定触发rollover时,文件的重命名规则.filePattern中可以指定date pattern(如yyyy-MM-dd HH)或者%i整数计数器 -->
    <RollingFile name="RollingFile" fileName="app.log" filePattern="app-%d{yyyy-MM-dd HH}.log">
      <PatternLayout>
        <Pattern>%d %p %c{1.} [%t] %m%n</Pattern>
      </PatternLayout>
      <Policies>
        <!-- TimeBasedTriggeringPolicy指定了基于时间的触发策略 -->
        <TimeBasedTriggeringPolicy interval="1" />
        <!-- SizeBasedTriggeringPolicy指定了基于文件大小的触发策略 -->
        <SizeBasedTriggeringPolicy size="250MB" />
      </Policies>
    </RollingFile>
  </Appenders>
  <Loggers>
    <Root level="info">
      <AppenderRef ref="RollingFile" />
    </Root>
  </Loggers>
</Configuration>
```

##### SizeBasedTriggeringPolicy

当日志文件达到了指定的size时,触发rollover操作,size参数可以用KB、MB、GB等做后缀来指定具体的字节数

```xml
<SizeBasedTriggeringPolicy size="250MB"/>
```

##### TimeBasedTriggeringPolicy

| 参数     | 类型    | 描述                                                         |
| -------- | ------- | ------------------------------------------------------------ |
| interval | Integer | 此参数需要与filePattern结合使用,规定了触发频率,默认值为1<br/>假设interval为4,若filePattern的date pattern的最小时间粒度为小时(如yyyy-MM-dd HH),则每4小时触发一次rollover<br/>若filePattern的date pattern的最小时间粒度为分钟(如yyyy-MM-dd HH-mm),则每4分钟触发一次rollover |

##### DefaultRolloverStrategy

```xml
默认的rollover策略,即使没有显式指明,也相当于为RollingFile配置下添加了如下语句<DefaultRolloverStrategy max="7"/>,DefaultRolloverStrategy默认的max为7

max参数需要于filePattern中的计数器%i配合才起作用
1.如果filePattern中仅含有date pattern,max参数将不起作用,例如filePattern="logs/app-%d{yyyy-MM-dd}.log"

2.如果filePattern中仅含有整数计数器(即%i),每次rollover文件重命名时的计数器将每次加1(初始值为1),若达到max的值,将删除旧的文件,例如filePattern="logs/app-%i.log"

3.如果filePattern中既含有date pattern,又含有%i,每次rollover计数器将每次加1,若达到max的值,将删除旧的文件,直到data不再符合,被替换为当前的日期和时间,计数器再从1开始,例如filePattern="logs/app-%d{yyyy-MM-dd HH-mm}-%i.log"
```

```xml
<Appenders>
  <RollingFile name="demo" fileName="app.log" filePattern="app-%d{yyyy-MM-dd}.log">
    <PatternLayout>
      <Pattern>%d %p %c{1.} [%t] %m%n</Pattern>
    </PatternLayout>
    <Policies>
      <SizeBasedTriggeringPolicy size="10KB" />
    </Policies>
    <DefaultRolloverStrategy max="3" />
  </RollingFile>
</Appenders>
```

| 第X次rollover | **当前用于写入log的文件** | **归档的文件**     |                                                              |
| ------------- | ------------------------- | ------------------ | ------------------------------------------------------------ |
| 0             | app.log                   |                    | 所有的log都写进app.log中                                     |
| 1             | app.log                   | app-2017-08-17.log | 当app.log的size达到10KB,触发第1次rollover,app.log被重命名为app-2017-08-17.log<br/>新的app.log被创建出来，用于写入log |
| 2             | app.log                   | app-2017-08-17.log | 当app.log的size达到10KB,触发第2次rollover,原来的app-2017-08-17.log将删除<br/>app.log被重命名为app-2017-08-17.log<br/>新的app.log文件被创建出来,用于写入log。 |

```xml
<Appenders>
  <RollingFile name="demo" fileName="app.log" filePattern="app-%i.log">
    <PatternLayout>
      <Pattern>%d %p %c{1.} [%t] %m%n</Pattern>
    </PatternLayout>
    <Policies>
      <SizeBasedTriggeringPolicy size="10KB" />
    </Policies>
    <DefaultRolloverStrategy max="3" />
  </RollingFile>
</Appenders>
```

| **第X次rollover** | **当前用于写入log的文件** | **归档的文件**                        |                                                              |
| :---------------: | :-----------------------: | :------------------------------------ | :----------------------------------------------------------- |
|         0         |          app.log          | -                                     | 所有的log都写进app.log中                                     |
|         1         |          app.log          | app-1.log                             | 当app.log的size达到10KB,触发第1次rollover,app.log被重命名为app-1.log<br/>新的app.log被创建出来,用于写入log |
|         2         |          app.log          | app-1.log<br/>app-2.log               | 当app.log的size达到10KB,触发第2次rollover,app.log被重命名为app-2.log<br/>新的app.log被创建出来，用于写入log |
|         3         |          app.log          | app-1.log<br/>app-2.log<br/>app-3.log | 当app.log的size达到10KB,触发第3次rollover,app.log被重命名为app-3.log<br/>新的app.log被创建出来,用于写入log |
|         4         |          app.log          | app-1.log<br/>app-2.log<br/>app-3.log | 当app.log的size达到10KB,触发第4次rollover,app-1.log被删除(即最初的、最旧的app.log)<br/>app-2.log被重命名为app-1.log,app-3.log被重命名为app-2.log,app.log被重命名为app-3.log<br/>新的app.log被创建出来,用于写入log |

```xml
<Appenders>
  <RollingFile name="demo" fileName="app.log" filePattern="app-%d{yyyy-MM-dd HH-mm}-%i.log">
    <PatternLayout>
      <Pattern>%d %p %c{1.} [%t] %m%n</Pattern>
    </PatternLayout>
    <Policies>
      <TimeBasedTriggeringPolicy />
      <SizeBasedTriggeringPolicy size="10KB" />
    </Policies>
    <DefaultRolloverStrategy max="3" />
  </RollingFile>
</Appenders>
```

| **第X次rollover** | **当前用于写入log的文件** | **归档的文件**                                               |                                                              |
| ----------------- | ------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 0                 | app.log                   | -                                                            | 所有的log都写进app.log中                                     |
| 1                 | app.log                   | app-2017-08-17 20-52-1.log                                   | 当app.log的size达到10KB,触发第1次rollover,app.log被重命名为app-2017-08-17 20-52-1.log<br/>新的app.log被创建出来,用于写入log |
| 2                 | app.log                   | app-2017-08-17 20-52-1.log<br/>app-2017-08-17 20-52-2.log<br/> | 当app.log的size达到10KB,触发第2次rollover,app.log被重命名为app-2017-08-17 20-52-2.log<br/>新的app.log被创建出来,用于写入log |
| 3                 | app.log                   | app-2017-08-17 20-52-1.log<br/>app-2017-08-17 20-52-2.log<br/>app-2017-08-17 20-52-3.log | 当app.log的size达到10KB,触发第3次rollover,app.log被重命名为app-2017-08-17 20-52-3.log.log<br/>新的app.log被创建出来,用于写入log |
| 4                 | app.log                   | app-2017-08-17 20-52-1.log<br/>app-2017-08-17 20-52-2.log<br/>app-2017-08-17 20-52-3.log | 当app.log的size达到10KB,触发第4次rollover,因计数器的值到达max值,app-2017-08-17 20-52-1.log被删除(即最初的、最旧的app.log)<br/>app-2017-08-17 20-52-2.log被重命名为app-2017-08-17 20-52-1.log<br/>app-2017-08-17 20-52-3.log被重命名为app-2017-08-17 20-52-2.log<br/>app.log被重命名为app-2017-08-17 20-52-3.log<br/>新的app.log被创建出来,用于写入log |
| 5                 | app.log                   | app-2017-08-17 20-52-1.log<br/>app-2017-08-17 20-52-2.log<br/>app-2017-08-17 20-52-3.log | 当前时间变为app-2017-08-17 20-53,触发第5次rollover,app-2017-08-17 20-52-1.log被删除<br/>app-2017-08-17 20-52-2.log被重命名为app-2017-08-17 20-52-1.log<br/>app-2017-08-17 20-52-3.log被重命名为app-2017-08-17 20-52-2.log<br/>app.log被重命名为app-2017-08-17 20-52-3.log<br/>新的app.log被创建出来,用于写入log |
| 6                 | app.log                   | app-2017-08-17 20-52-1.log<br/>app-2017-08-17 20-52-2.log<br/>app-2017-08-17 20-52-3.log<br/>app-2017-08-17 20-53-1.log | 当app.log的size达到10KB,触发第6次rollover,app.log被重命名为app-2017-08-17 20-53-1.log<br/>新的app.log被创建出来,用于写入log |

##### Delete

当触发rollover时,删除Dir文件夹或其子文件下面的文件名符合app-*.log.gz且距离最后的修改日期超过60天的文件

```xml
<DefaultRolloverStrategy>
  <!-- basePath指定路径 -->
  <!-- maxDepth指定目录扫描深度,为2表示扫描Dir文件夹及其子文件夹 -->
  <Delete basePath="Dir" maxDepth="2">
    <IfFileName glob="*/app-*.log.gz" />
    <IfLastModified age="60d" />
  </Delete>
</DefaultRolloverStrategy>
```

---
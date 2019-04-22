# Seven Tips For Logging in C#

In the world of software development, unit testing has quickly risen to the prominence of a required process during the development phase. And documentation is quickly ascending as a must need for post production. Lost in the white hot glare of these two shining stars is another process, one that has existed for some time but does not get the same attention of testing and documentation, logging. 

Most developers will argue that they do not neglect logging. It is a vital tool in their debugging arsenal. One of the benchmarks of progressing from a novice developer to the next level is the ability to output a stack trace when an exception occurs and understand it. But how many developers think of logs as a post production asset? 

Logging is the powerful bridge between being a vital tool for unit testing and debugging in production and as an additional means of communication with the end users of your application, who may not have the technical savvy to read a stack trace. 

This article will show you how to take your C# logging to the next level whether it is logging to the console, a file or across multiple server and to review the concept of logging as a vital part of your software products post production support.  

## Getting Started

Logging with C# requires a logging library. Fortunately Microsoft bundles a decent native library called TraceSource within the C# package. In order to utilize TraceSource, it will need to be defined in your configuration file. The configuration file is located in the folder with the application executable and has the name of the application with the .config file name extension added.  

```XML
<configuration>  
  <system.diagnostics>  
    <sources>  
      <source name="TraceTest" switchName="SourceSwitch"   
        switchType="System.Diagnostics.SourceSwitch" >  
        <listeners>  
          <add name="console" />  
          <remove name ="Default" />  
        </listeners>  
      </source>  
    </sources>  
    <switches>  
      <!-- You can set the level at which tracing is to occur -->  
      <add name="SourceSwitch" value="Warning" />  
        <!-- You can turn tracing off -->  
        <!--add name="SourceSwitch" value="Off" -->  
    </switches>  
    <sharedListeners>  
      <add name="console"   
        type="System.Diagnostics.ConsoleTraceListener"   
        initializeData="false"/>  
    </sharedListeners>  
    <trace autoflush="true" indentsize="4">  
      <listeners>  
        <add name="console" />  
      </listeners>  
    </trace>  
  </system.diagnostics>  
</configuration> 
```

Now define TraceSource in your code.

```C#
// Initialize the trace source.
  static TraceSource ts = new TraceSource("TraceTest");
  SwitchAttribute("SourceSwitch", typeof(SourceSwitch))]

  Console.WriteLine("TraceSource name = " + ts.Name);
  Console.WriteLine("TraceSource switch level = " + ts.Switch.Level);
  Console.WriteLine("TraceSource switch = " + ts.Switch.DisplayName);
```

Logging information to the screen is helpful, but what if you want to run a series of debug test and compare the results or maintain an event log? TraceSource outputs its messages to a `listener`, an object that receives messages from the `System.Diagnostics.Debug` and `System.Diagnostics.Trace` classes and outputs them to a predetermined location, such as the console.log, event log, or a file.  

To add a `listener` to your configuration file that outputs to a file use this code: 

```XML

<configuration>
  <system.diagnostics>
    <trace autoflush="false" indentsize="4">
      <listeners>
        <add name="myListener" type="System.Diagnostics.TextWriterTraceListener" initializeData="TextWriterOutput.log" />
        <remove name="Default" />
      </listeners>
    </trace>
  </system.diagnostics>
</configuration>

```

This adds a `listener` object called *myListener* which is of a `TextWriterTraceListener` type and outputs its messages to a log file titled *TextWriterOutput*.

Use the Trace class to output text to the file. 

```C#
Trace.TraceInformation("Test message.");
// You must close or flush the trace to empty the output buffer.
Trace.Flush();
```

Listeners can also be added in code. You do so by adding the new listener to the Trace.Listeners collection and pass trace information to the listeners. 

```C#
Trace.Listeners.Add(new TextWriterTraceListener("TextWriterOutput.log", "myListener"));
Trace.TraceInformation("Test message.");
// You must close or flush the trace to empty the output buffer.
Trace.Flush();
```

## Logging Libraries

TraceSource is effective for simple applications but as an application becomes more complex you will want to consider a more robust logging solution. There are several libraries that handle the heavy lifting of working with the advanced features of `System.Diagnostic.Tracesource` such as [log4net](https://logging.apache.org/log4net/), [Nlog](https://nlog-project.org/), or [serilog](https://serilog.net/). 

Which library you chose will depend on your unique needs though for the most part they are all similar in functionality. For our examples we will use log4net.

### Configuring Log4Net

To utilize a library for logging you will need to add it to your configuration file.

```C#
<configuration>
  <configSections>
    <section name="log4net" type="log4net.Config.Log4NetConfigurationSectionHandler, log4net" />
  </configSections>
  <log4net>
    <appender name="LogFileAppender" type="log4net.Appender.RollingFileAppender">
    <param name="File" value="myLoggerFile.log"/>
    <lockingModel type="log4net.Appender.FileAppender+MinimalLock" />
    <appendToFile value="true" />
    <rollingStyle value="Size" />
    <maxSizeRollBackups value="2" />
    <maximumFileSize value="1MB" />
    <staticLogFileName value="true" />
    <layout type="log4net.Layout.PatternLayout">
            <param name="ConversionPattern" value="%d [%t] %-5p %c %m%n"/>
    </layout>
</appender>

<root>
    <level value="ALL" />
    <appender-ref ref="LogFileAppender" />
</root>
  </log4net>
</configuration>
```

The `<appender />` element specifies the name and logger type. In this example we are using a `RollingFileAppender` but there are many [types you can select](https://logging.apache.org/log4net/log4net-1.2.13/release/sdk/log4net.Appender.html). 

`<param name="File" value="myLoggerFile.log"/>` sets the file name and path. In this case we will be logging to a file called *myLoggerFile.log*.

The `<lockingModel />` element is required to enable you to write to the file. Without setting this the file will be locked until you shutdown the application or website.

`<appendToFile value="true"/>` as it implies directs Log4Net to append new entries to the end of the file. 

`<rollingStyle value="Size" />` is needed for the `RollingFileAppender` type we selected. It defines a the criteria for when it should  roll (create a new file). In this case we will set a maxmimum file size. 

`<maxSizeRollBackups value="2" />` indicates the number of log files that will be kept. This is instructing Log4Net to keep two. 

`<maximumFileSize value="1MB" />` determines the maximum file size. Here we are instructing that the file sizes not exceed 1MB. 

`<staticLogFileName value="true" />` indicates that the file name will not change. 

The `<layout/>` element tells Log4Net how each log line should look like. As with the types of appenders there are multiple options to displaying your log content. You can review these options [here](https://logging.apache.org/log4net/log4net-1.2.13/release/sdk/log4net.Layout.html). 

The `<root>` element is the root logger. If it is set then all logger instances in your application will turn up in the logger specified in the <appender-ref />element. 







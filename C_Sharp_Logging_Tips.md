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
using System;
using System.Diagnostics;
class Program
    {
        private static TraceSource _source = new TraceSource("TraceTest");
        static void Main(string[] args)
        {
            var i = 10; var j = 5;
            _source.TraceEvent(TraceEventType.Information, 0, "Calculating a multiplication problem: {0}x{1}", i,j);
           ....
            }
}
```

You will get the following output: 

```C#
TraceTest Information: 0 : Calculating a multiplication problem: 10x5
```


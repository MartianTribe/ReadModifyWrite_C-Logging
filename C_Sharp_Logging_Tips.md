# Seven Tips For Logging in C#

In the world of software development, unit testing has quickly risen to the prominence of a required process in development. And documentation is quickly ascending as a must need for post production. Lost in the white hot glare of these two shining stars is another process, one that has existed for some time but does not get the same attention as testing and documentation, logging. 

Logging is the powerful bridge between being a vital tool for unit testing and debugging in production and a data rich repository of the real world use of your application. Logs can also provide a layer of support and communication with system administrators and even users.  

This article will provide you with seven tips to take C# logging to the next level whether it is logging to the console, a file or database for debugging and post-production support. We'll examine: 

1. Configuring for TraceSource, the Microsoft bundled logging library
2. Logging to a file
3. Creating a logging plan
4. Configuring a logging framework
5. Logging Contextual Data
6. Structured Logging
7. Diagnostic Logging

## Configuring for TraceSource, the bundled logging library

Logging with C# requires a logging library. Fortunately Microsoft bundles a decent native library called TraceSource within the C# package. In order to utilize TraceSource, it will need to be defined in your configuration file. The configuration file is located in the folder with the application executable and has the name of the application with the .config file name extension added. You can define TraceSource by adding the code below. 

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
```

And you can now send messages to the console.

```C#
  Console.WriteLine("TraceSource name = " + ts.Name);
  Console.WriteLine("TraceSource switch level = " + ts.Switch.Level);
  Console.WriteLine("TraceSource switch = " + ts.Switch.DisplayName);
```

## Logging to a file

Logging information to the screen is helpful, but what if you want to run a series of debug test and compare the results or maintain an event log? TraceSource outputs its messages to a `listener`, an object that receives messages from the `System.Diagnostics.Debug` and `System.Diagnostics.Trace` classes and outputs them to a predetermined location, such as the console.log, event log, or a file.  

To add a `listener` to your configuration file that outputs to a file add this code to you r configuration file.  

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

This adds a `listener` object called *myListener* which is of a `TextWriterTraceListener` type and outputs the messages to a log file titled *TextWriterOutput*.

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

## Creating a logging plan

In order to consider your logs as part of your development debugging strategy as well as live application support there should be some set goals to your logging and a plan to achieve those goals.

Logs on your local machine are easy to open files with manageable and helpful data. A released app is a different story, logs and accessing them can sometimes be a challange.  
- There is a lot more data being accumulated.  
- You may not have access to that data.  
- The logs could be on multiple servers or spread across multiple service boundaries. 
- Generally, there is little context of the user.
- The logs themselves are hard to query.
- The logs may only be retained for a short period of time. 

Having a strategy of what and when to log for released apps is usually the only resource you will have to discover why a feature of your app is not functioning or is functioning incorrectly. There are tools that can let you know of a memory leak or performance issues but they can't provide specific information on why a user can't login or view a certain area of the application. 

Development teams should incorporate a culture of logging in the same manner that TDD has become incorporated within the development cycle. Code reviews should look at not just the testing coverage but if the developer has followed the logging plan. A few points to help develop this culture would be:  

- Logging key transactional events. For example, if we had a method that added integers we would want to log the calculation. 

- Logging that something happened is not enough, you want to ensure your logs contain contextual data. 

- Aggregate your data. Having to dig across multiple servers and then the log files on each is an unnecessary pain. Devise a strategy to consolidate all your log data into one location, available to your entire team and easily distilled. 

- Monitor the logs. They should not be static files that are only reviewed when a bug report crosses your desk but they should be inspected on a regular basis looking for issues and errors. 

- Select a logging framework that can handle the needs of your logging plan. 

## Configuring a logging framework

There are a lot of tech shops where the log messages are simply the error description. 

```#C
  catch(Exception e) 
  { 
      this.log(e)
  }
```

This is okay, we can see the Exception thrown, but it would be so much better with some context. The problem with putting context into your log messages, the user id, a timestamp, is it requires a lot of code. 

This is where logging libraries and frameworks come in. These libraries, such as  such as [Log4Net](https://logging.apache.org/log4net/), [Nlog](https://nlog-project.org/), or [serilog](https://serilog.net/), handle the heavy lifting of working with the advanced features of `System.Diagnostic.Tracesource` so you don't have to. 

Which library you chose will depend on your unique needs though for the most part they are all similar in functionality. For our examples we will use *Log4Net*.

### Configuring Log4Net

To utilize a library for logging it will need to be added to your configuration file.

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

### What are all those elements for?

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

### Outputting a message

Once your configuration is complete you call Log4Net in the class you want to use logging in by adding this code: 

```C#
private static readonly ILog log = LogManager.GetLogger(MethodBase.GetCurrentMethod().DeclaringType);
```

Next, at the very top of your Main() method, add the following line.  

```C#
XmlConfigurator.Configure();
```

And finally, add this line where you want to log information: 

```C#
log.Info("This is my first log message");
``` 

The full class code would look like this: 

```C#

class LogTest 
{
  private static readonly ILog log = LogManager.GetLogger(MethodBase.GetCurrentMethod().DeclaringType); 

  static void Main(string[] args) 
  {
    XmlConfigurator.Configure();

    var displayMessage("Hello World!");

    log.Info(displayMessage)

  }

}
```

and the results put this log message into more context than just using TraceSource. 

```
2019-04-23 00:39:10, 092 [1] INFO  ConsoleProject.LogTest Hello World!
```

This log message provides us with a timnestamp, the thread, the logging level, the class the message originated from and the message. Not bad, but it would be nice to get even more context with our message. 

## Logging Contextual Data

Using the standard contextual items available within the app such as the timestamp provides some much needed context to our log message. But what if you need specific data of a class or method? Let's add some strengh to our log message by adding some specific detail to it. Start by adding a class. 

```C#
  public class Dog

    public int ID { get; set; }
    public int License { get; set; }
    public string Name { get; set; }
    public string Breed { get; set; }
```

In our Dog class License is an int that is required. Let's create an instance of Dog and save it to a database. 

```C#
public static void CreateDog(int? licenseNum, string dogName, string dogBreed)
{

    using (var context = new ApplicationDbContext())
   {
      var myDog = new Dog();
      myDog.License = (int) licenseNum;
      myDog.Name = dogName;
      myDog.Breed = dogBreed

      context.Dogs.Add(myDog);
      context.SaveChanges();

   }

}
```

The problem with this code is I am passing in a license number that could be null, but my class requires a value, and I am not checking for null. The first time I ran it, my app failed on the passing of a null licenseNun. Let's wrap this in a try statement and log to see what failed. 

```C#
public static void CreateDog(int? licenseNum, string dogName, string dogBreed)
{

  try
  {


    using (var context = new ApplicationDbContext())
    {
      var myDog = new Dog();
      myDog.License = (int) licenseNum;
      myDog.Name = dogName;
      myDog.Breed = dogBreed

      log.Debug("Creating a dog.");

      context.Dogs.Add(myDog);
      context.SaveChanges();

    }

  catch (Exception e) 
  {
    log.Error(e)
  }

}
```
Let's create some dogs. 

```C#

  CreateDog(1, "Fido", "Labrador Retriever")
  CreateDog(null, "Rex", "German Shepherd")
```

Now check out the logs.

```C#
DEBUG2019-04-23 13:11:08.9834 [11] Creating a dog [CID:(null)]
DEBUG2019-04-23 13:11:10.8458 [11] Creating a dog [CID:(null)]
ERROR2019-04-23 13:11:10.8673 [11] System.InvalidOperationException: Nullable object must have a value.
at System.ThrowHelper.ThrowInvalidOperationException(ExceptionResource resource)
at System.Nullable`1.get_Value()
at DogPark.Web.Controllers.GimmeErrorsController.CreateDogs(Nullable`1 licenseNum, String dogName, String dogBreed) in c:DogParkSandbox.NetDogPark.WebDogPark.WebControllersGimmeErrorsController.cs:line 57 [CID:(null)]
```
This is useful, we get the boilerplate data like timestamp and we can certainly see the error thrown. In our simplified example above we could easily find the error by placing a breakpoint and rewrite our "Creating a dog" log message to something like: 

```C#
  log.Debug("Creating a dog named {0}.", myDog.dogName);
```

Our log messages would now look like this. 

```C#
DEBUG2019-04-23 13:11:08.9834 [11] Creating a dog named Fido [CID:(null)]
DEBUG2019-04-23 13:11:10.8458 [11] Creating a dog named Rex [CID:(null)]
ERROR2019-04-23 13:11:10.8673 [11] System.InvalidOperationException: Nullable object must have a value.
at System.ThrowHelper.ThrowInvalidOperationException(ExceptionResource resource)
at System.Nullable`1.get_Value()
at DogPark.Web.Controllers.GimmeErrorsController.CreateDogs(Nullable`1 licenseNum, String dogName, String dogBreed) in c:DogParkSandbox.NetDogPark.WebDogPark.WebControllersGimmeErrorsController.cs:line 57 [CID:(null)]
```

So now we know when we created a dog named Rex there was an error with the license number. But what if we needed to see the whole Dog object? 

See those `[CID:(null)]` entries. Those are part of the Log4Net logging methods, specifically a property called `debugData` that will display key-value pairs of data. Let's rewrite our opening log statement and serialize the data. 

```C#
  log.Debug(string.Format("Creating a dog: {0}",JsonConvert.SerializeObject(myDog)));
```

Which will display in our log file as: 

```C#
  DEBUG2019-23-04 10:39:53.3295 [11] Creating a dog: {"ID":0,"License":1,"Fido":"Labrador Retriever"}
```

Not bad, we're starting to get some context that provides a better picture of what is happening in our app. The problem with this approach is, if we had a very complext object, we would end up having to add a lot of additional items to our log string.  

## Strucutred Logging

Logging should be simple and quick, not another process within itself and the end result easily readable. Log4Net offers a JSON pacakge, log4net.Ext.Json which enables you log any object as JSON. Install the package and add it as a serialized layout to any appender. 

```XML
<appender...>
    <layout type='log4net.Layout.SerializedLayout, log4net.Ext.Json'>
    </layout> 
</appender>
```

We can now reduce our log call to this. 

```C#
var aDog = CreateDog(1, "Fido", "Labrador Retriever")
log.Debug("Created a dog", new { Dog=aDog,CreatedBy=User.Identity.Name});
```

Which will display a very readable log message. 

```JSON

"json":
{
  "_Dog": { 
    "id":"10",
    "license":"1",
    "dogName":"Fido",
    "dogBreed":"Labrador Retriever"
  },
  "createdBy": "mkurtz@alldogs.com"
}
```

## Diagnostic Logging

And finally, our last point, diagnostic logging. In the `aDog` object I created above you might of noticed I logged the user who created it. This can be incredibly useful when your app is in production and thousands of users might be creating a `Dog` object. Adding diagnostic details such as this is what separates top level logging from just outputting the Exception to the console. 

There's a whole host of details that could be useful, from the user location to HTTPRequest parameters. But why place the burden on developers to remember to add those details to each log message when frameworks like Log4Net and others make it easy to automate. 

Log4Net provides a class, `LogicalThreadContext` which enables the automation of diagnostic data. You just have to input what data want to log. The set up is simple, just add context variables you want to log: 

```XML
type="DogPark.log4net.DogParkAppender, DogPark.log4net">
   <logicalThreadContextKeys>User,Request</logicalThreadContextKeys>
```
We're telling Log4Net to add `User` and `Request` as keys for the `LogicalThreadContext`. And in the `Global.asax` you can add this code for every HTTPRequest. You can even modify what request parameters will be logged. 


```C#
void MvcApplication_AuthenticateRequest(object sender, EventArgs e)
{
   try
   {
      log4net.LogicalThreadContext.Properties["User"] = User;
   }
   catch (Exception ex)
   {
      log.Error(ex);
   }
}   void MvcApplication_BeginRequest(object sender, EventArgs e)
{
   log4net.LogicalThreadContext.Properties["Request"] = new
   {
      HostAddress = Request.UserHostAddress,
      RawUrl = Request.RawUrl,
      QueryString = Request.QueryString,
      FormValues = Request.Form
   };
}
```

Now to log structured, diagnostic content you simply add this line of code: 

```C#
var aDog = CreateDog(1, "Fido", "Labrador Retriever")
log.Debug("Created a dog", aDog);
```

and get this context rich, easy to read log message: 

```JSON

"json":
{
  "_Dog": { 
    "id":"10",
    "license":"1",
    "dogName":"Fido",
    "dogBreed":"Labrador Retriever",
    "_context": {

      "_request": { 
         "rawurl": "https://www.doghouse.com",
         "hostAddress": "::1",
         "FormValues":[],
         "QueryString":[]
      },

      "_user": { 
        ...
      }

    }
  },
}
```

## Conclusion

Logging is a powerful tool for both development and production debugging. To ensure that your application provides the level of logging that'll assist your development team in tracking down and fixing bugs in a timely manner follow these steps: 

1. Have a plan: Know what needs to be logged and when and where to store the log files so that the entire team can access them when needed. 
2. Use a logging framework, someone already invented that wheel, spend your time setting up what needs to be logged rather than how to do it. 
3. Structured, diagnostic, context rich logging provides the best means, from a logging perspectife,  of solving your bug issues.  






























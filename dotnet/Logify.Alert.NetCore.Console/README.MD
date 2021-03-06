# Logify Alert for .NET Core console applications
A console client to report exceptions to [Logify Alert](https://logify.devexpress.com).

## Install <a href="https://www.nuget.org/packages/Logify.Alert.Console/"><img alt="Nuget Version" src="https://img.shields.io/nuget/v/Logify.Alert.Console.svg" data-canonical-src="https://img.shields.io/nuget/v/Logify.Alert.Console.svg" style="max-width:100%;" /></a>
```sh
$ Install-Package Logify.Alert.Console
```

## Quick Start

### Manual error reporting
```csharp
using DevExpress.Logify.Console;
try {
    LogifyAlert.Instance.ApiKey = "SPECIFY_YOUR_API_KEY_HERE";
    RunYourCode();
}
catch (Exception e) {
    LogifyAlert.Instance.Send(e);
}
```

## Configuration
You can set up the Logify Alert client using configuration file as follows.

#### JSON configuration file:
```json
{
  "LogifyAlert": {
    "apiKey": "SPECIFY_YOUR_API_KEY_HERE",
    "customData": {
      "MACHINE_NAME": "My Server"
    }
  }
}
```

```csharp
var builder = new ConfigurationBuilder()
    .SetBasePath(Directory.GetCurrentDirectory())
    .AddJsonFile("LogifyAlert.json", optional: false, reloadOnChange: false);

IConfigurationRoot configuration = builder.Build();
LogifyAlert client = LogifyAlert.Instance;
client.Configure(configuration.GetSection("LogifyAlert"));
```

#### XML configuration file:
```xml
<configuration>
  <LogifyAlert>
    <ApiKey>SPECIFY_YOUR_API_KEY_HERE</ApiKey>
    <AppName>Your application name</AppName>
    <AppVersion>1.0.2</AppVersion>
    <CustomData>
      <MACHINENAME>My Server</MACHINENAME>
    </CustomData>
  </LogifyAlert>
</configuration>
```

```csharp
var builder = new ConfigurationBuilder()
    .SetBasePath(Directory.GetCurrentDirectory())
    .AddXmlFile("LogifyAlert.xml", optional: false, reloadOnChange: false);

IConfigurationRoot configuration = builder.Build();
LogifyAlert client = LogifyAlert.Instance;
client.Configure(configuration.GetSection("LogifyAlert"));
```

#### INI configuration file:
```ini
[LogifyAlert]
ApiKey = SPECIFY_YOUR_API_KEY_HERE
AppName = Your application name
ApiVersion = 1.0.2
```

```csharp
var builder = new ConfigurationBuilder()
    .SetBasePath(Directory.GetCurrentDirectory())
    .AddIniFile("LogifyAlert.ini", optional: false, reloadOnChange: false);

IConfigurationRoot configuration = builder.Build();
LogifyAlert client = LogifyAlert.Instance;
client.Configure(configuration.GetSection("LogifyAlert"));
```

## API
### Properties
#### ApiKey
String. Specifies an [API Key](https://logify.devexpress.com/Documentation/CreateApp) used to register the applications within the Logify service.
```csharp
client.ApiKey = "My Api Key";
```

#### AppName
String. Specifies the application name.
```csharp
client.AppName = "My Application";
```
#### AppVersion
String. Specifies the application version.
```csharp
client.AppVersion = "1.0.2";
```

#### CustomData
IDictionary<String, String>. Gets the collection of custom data sent with generated reports.
Use the **CustomData** property to attach additional information to the generated report. For instance, you can use this property to track additional metrics that are important in terms of your application: CPU usage, environment parameters, and so on.

```csharp
client.CustomData["CustomerName"] = "Mary";
```

#### Instance
Singleton. Returns the single instance of the LogifyAlert class.
```csharp
LogifyAlert client = LogifyAlert.Instance;
```

#### OfflineReportsCount
Integer. Specifiies the number of last reports to be kept once an Internet connection is lost. Reports are only saved when the *OfflineReportsEnabled* property is set to **true**.
```csharp
client.OfflineReportsEnabled = true;
client.OfflineReportsCount = 20; // Keeps the last 20 reports
```

#### OfflineReportsDirectory
String. Specifies a directory path that will be used to store reports once an Internet connection is lost. Reports are only saved when the *OfflineReportsEnabled* property is set to **true**.
```csharp
client.OfflineReportsEnabled = true;
client.OfflineReportsDirectory = "<directory-for-offline-reports>";
```

#### OfflineReportsEnabled
Boolean. Default value is **false**. Specifies whether or not Logify should store the last *OfflineReportsCount* reports once an Internet connection is lost. To send the kept reports once an Internet connection is available, call the *SendOfflineReports* method.
```csharp
client.OfflineReportsEnabled = true;
client.OfflineReportsCount = 20; // Keeps the last 20 reports
client.OfflineReportsDirectory = "<directory-for-offline-reports>";
```

#### UserId
String. Specifies a unique user identifier that corresponds to the sent report.
```csharp
client.UserId = "user@myapp.com";
```

### Methods for automatic reporting
Logify Alert allows you to automatically listen to uncaught exceptions and deliver crash reports. For this purpose, use the methods below.

#### StartExceptionsHandling()
Commands Logify Alert to start listening to uncaught exceptions and sends reports for all processed exceptions.
```csharp
client.StartExceptionsHandling();
```

#### StopExceptionsHandling()
Commands Logify Alert to stop listening to uncaught exceptions.
```csharp
client.StopExceptionsHandling();
```

### Methods for manual reporting
Alternatively, Logify Alert allows you to catch required exceptions manually, generate reports based on caught exceptions and send these reports only. For this purpose, use the methods below.

#### Send(Exception e)
Generates a crash report based on the caught exception and sends this report to the Logify Alert service.
```csharp
try {
    RunCode();
}
catch (Exception e) {
    client.Send(e);
}
```

#### Send(Exception e, IDictionary<String, String> additionalCustomData)
Sends the caught exception with specified custom data to the Logify Alert service.
```csharp
try {
    RunCode();
}
catch (Exception e) {
    var data = new Dictionary<String, String>();
    data["FailedOperation"] = "RunCode";
    client.Send(e, data);
}
```

#### SendOfflineReports
Sends all reports saved in the *OfflineReportsDirectory* folder to the Logify Alert service.

### Events

#### BeforeReportException
Occurs before Logify Alert sends a new crash report to the service.

The **BeforeReportException** event occurs between the [CanReportException](#canreportexception) event and sending a new report to the Logify Alert service. If report send is canceled in the [CanReportException](#canreportexception) event's handler, the report is not sent and the **BeforeReportException** event isn't raised.

Handle the **BeforeReportException event** to add custom data to the sent report. To do this, assign the required data to the [CustomData](#customdata) property.
```csharp
LogifyAlert.Instance.BeforeReportException += OnBeforeReportException;

void OnBeforeReportException(object sender, EventArgs e) {
    LogifyAlert.Instance.CustomData["LoggedInUser"] = "Mary";
}
```


#### CanReportException
Occurs between generating a new crash report and raising the [BeforeReportException](#beforereportexception) event.

The **CanReportException** event occurs right after a new report is generated and prepared to be sent to the Logify Alert service. Handle the **CanReportException** event to cancel sending the report. To do this, assign **true** to the appropriate CanReportExceptionEventArgs's **Cancel** property. Thus, the generated report is not posted to the service and the [BeforeReportException](#beforereportexception) isn't raised.
```csharp
LogifyAlert.Instance.CanReportException += OnCanReportException;

void OnCanReportException(object sender, CanReportExceptionEventArgs args) {
    if (args.Exception is MyCustomException)
        args.Cancel = true;
}
```

### Attributes
#### LogifyIgnoreAttribute
Indicates that exceptions thrown from a specific method should not be handled and sent to Logify Alert.

```csharp
[LogifyIgnore(true)]
void ThisMethodShouldNotReportAnyExceptions() {
    RunSomeCode();
}
```

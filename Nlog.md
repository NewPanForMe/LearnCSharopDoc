# NLog

## 基础配置

```xml
<?xml version="1.0" encoding="utf-8" ?>
<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      autoReload="true"
      internalLogLevel="Info">
	<targets>
		<target name= "allDatabase" xsi:type="Database"
		        connectionString="server=118.178.241.151,15068;database=BMS_V1;user id=sa;password=ys110565;TrustServerCertificate=true">
			<commandText>
				INSERT INTO dbo.NLog ([Application], [Logged], [Level], [Message], [Logger], [CallSite],[Exception]) VALUES (@application, @logged, @level, @message,@logger, @callSite, @exception);
			</commandText>
			<parameter name="@application" layout="AspNetCoreNlog" />
			<parameter name="@logged" layout="${date}"/>
			<parameter name="@level" layout=" ${level}" />
			<parameter name="@message" layout="${message}"/>
			<parameter name="@logger" layout="${logger}" />
			<parameter name= "@callSite" layout="${callsite:filename=true}" />
			<parameter name="@exception" layout="${exception:tostring}"/>
		</target>
		<target xsi:type="File"
		        name="info"
		        fileName="$log\${shortdate}\info\info.txt"
		        layout ="${longdate}|${logger}|${uppercase:${level}}|${message} ${exception}"/>
		<target xsi:type="File"
		        name="error"
		        fileName="$log\${shortdate}\error\error.txt"
		        layout ="${longdate}|${logger}|${uppercase:${level}}|${message} ${exception}"/>
		<target xsi:type="File"
		        name="warn"
		        fileName="$log\${shortdate}\warn\warn.txt"
		        layout ="${longdate}|${logger}|${uppercase:${level}}|${message} ${exception}"/>

		<target xsi:type="File"
		        name="fatal"
		        fileName="$log\${shortdate}\fatal\fatal.txt"
		        layout ="${longdate}|${logger}|${uppercase:${level}}|${message} ${exception}"/>
		<target xsi:type="File" name="allfile" fileName="$log\${shortdate}\all\all.txt"
		        layout="${longdate}|${event-properties:item=EventId:whenEmpty=0}|${level:uppercase=true}|${logger}|${message} ${exception:format=tostring}" />
		<target xsi:type="File" name="systemServices" fileName="log\${shortdate}\Log-SystemServices.log"  archiveAboveSize="1000000"
		        maxArchiveFile="2"
		        layout="${longdate}|${event-properties:item=EventId:whenEmpty=0}|${level:uppercase=true}|${logger}|${message} ${exception:format=tostring}" />
		<target xsi:type="Console" name="lifetimeConsole" layout="${MicrosoftConsoleLayout}" />
	</targets>
	<rules>
		<logger name="*" minlevel="fatal" writeTo="fatal" />
		<logger name="*" minlevel="error" writeTo="error" />
		<logger name="*" minlevel="warn" writeTo="allDatabase" />
		<logger name="*" minlevel="Info" writeTo="info" />
	</rules>
</nlog>
```

## nlog Rules 优先级：Trace>Debug>Info>Warn>Error>Fatal

## nlog 存数据库

```xml
<target name= "allDatabase" xsi:type="Database"
            connectionString="server=118.178.241.151,15068;database=BMS_V1;user id=sa;password=ys110565;TrustServerCertificate=true">
        <commandText>
            INSERT INTO dbo.NLog ([Application], [Logged], [Level], [Message], [Logger], [CallSite],[Exception]) VALUES (@application, @logged, @level, @message,@logger, @callSite, @exception);
        </commandText>
        <parameter name="@application" layout="AspNetCoreNlog" />
        <parameter name="@logged" layout="${date}"/>
        <parameter name="@level" layout=" ${level}" />
        <parameter name="@message" layout="${message}"/>
        <parameter name="@logger" layout="${logger}" />
        <parameter name= "@callSite" layout="${callsite:filename=true}" />
        <parameter name="@exception" layout="${exception:tostring}"/>
</target>
```

```xml
<logger name="*" minlevel="warn" writeTo="allDatabase" />
```

>需要包：NLog.Database
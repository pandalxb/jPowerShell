![](https://img.shields.io/maven-central/v/io.github.pandalxb/jPowerShell.svg)
![](https://img.shields.io/github/license/pandalxb/jPowerShell.svg)
![](https://travis-ci.org/pandalxb/jPowerShell.svg)

# jPowerShell

Respect to the author profesorfalken, pull from https://github.com/profesorfalken/jPowerShell and improved it

Simple Java API that allows to interact with PowerShell console

```java
    PowerShell.openSession()
              .executeCommandAndChain("Get-Process", (res -> System.out.println("List Processes:" + res.getCommandOutput())))
              .executeCommandAndChain("Get-WmiObject Win32_BIOS", (res -> System.out.println("BIOS information:" + res.getCommandOutput())))
              .close();
```

## Installation

To install jPowerShell for Maven you have just to add to your pom.xml:

      <dependency>
        <groupId>io.github.pandalxb</groupId>
        <artifactId>jPowerShell</artifactId>
        <version>1.0.0</version>
      </dependency>

## Basic Usage

### Single command execution

If you only need to execute a single command, this is the quickest way to do it.

```java
   //Execute a command in PowerShell session
   PowerShellResponse response = PowerShell.executeSingleCommand("Get-Process");

   //Print results
   System.out.println("List Processes:" + response.getCommandOutput());
```

### Executing one or multiple commands using the same PowerShell session

If you have to execute multiple commands, it is recommended to reuse the same session in order to be more efficient (each session has to open a PowerShell console process in the background).

```java
   //Creates PowerShell session (we can execute several commands in the same session)
   try (PowerShell powerShell = PowerShell.openSession()) {
       //Execute a command in PowerShell session
       PowerShellResponse response = powerShell.executeCommand("Get-Process");

       //Print results
       System.out.println("List Processes:" + response.getCommandOutput());

       //Execute another command in the same PowerShell session
       response = powerShell.executeCommand("Get-WmiObject Win32_BIOS");

       //Print results
       System.out.println("BIOS information:" + response.getCommandOutput());
   } catch(PowerShellNotAvailableException ex) {
       //Handle error when PowerShell is not available in the system
       //Maybe try in another way?
   }
```

You can also choose to execute the same commands with a more fluent style using the _executeCommandAndChain_ method:

```java
    PowerShell.openSession()
                    .executeCommandAndChain("Get-Process", (res -> System.out.println("List Processes:" + res.getCommandOutput())))
                    .executeCommandAndChain("Get-WmiObject Win32_BIOS", (res -> System.out.println("BIOS information:" + res.getCommandOutput())))
                    .close();
```

### Configure jPowerShell Session ####

You can easily configure the jPowerShell session:

* *By project* creating a _jpowershell.properties_ file in the classpath of your project and settings the variables you want to override.
* *By call*, using a map that can be chained to powershell call.

For example:

```java
    //Set the timeout when waiting for command to terminate to 30 seconds instead of 10 (default value)
    Map<String, String> myConfig = new HashMap<>();
    myConfig.put("maxWait", "30000");
    response = powerShell.configuration(myConfig).executeCommand("Get-WmiObject Win32_BIOS");
```

The variables that can be configured in jPowerShell are:

*waitPause*: the pause in ms between each loop pooling for a response. Default value is 10

*maxWait*: the maximum wait in ms for the command to execute. Default value is 10000

*tempFolder*: if you set this variable jPowerShell will use this folder in order to store temporary the scripts to execute.
By default the environment variable _java.io.tmpdir_ will be used.

## Advanced usage

### Setting the PowerShell executable path

If the PowerShell executable has a different name/path on your system, you can change it when opening a new session:

```java
    //Creates PowerShell session
    try (PowerShell powerShell = PowerShell.openSession("myCustomPowerShellExecutable.exe")) {
       [...]
```

### Executing PowerShell Script

In order to execute a PowerShell Script it is recommended to use the executeScript() method instead of executeCommand():

```java
   try (PowerShell powerShell = PowerShell.openSession()) {       
       //Increase timeout to give enough time to the script to finish
       Map<String, String> config = new HashMap<String, String>();
       config.put("maxWait", "80000");
       
       //Execute script
       response = powerShell.configuration(config).executeScript("./myPath/MyScript.ps1");
       
       //Print results if the script
       System.out.println("Script output:" + response.getCommandOutput());
   } catch(PowerShellNotAvailableException ex) {
       //Handle error when PowerShell is not available in the system
       //Maybe try in another way?
   }
```

### Executing PowerShell Scripts packaged inside jar

In order to execute a PowerShell Script that is bundled inside a jar you must use a BufferedReader to load the resource:

```java
    PowerShell powerShell = PowerShell.openSession();
    String script = "resourcePath/MyScript.ps1"
    String scriptParams = "-Parameter value"

    //Read the resource
    BufferedReader srcReader = new BufferedReader(
                    new InputStreamReader(getClass().getResourceAsStream(script)));

    if (scriptParams != null && !scriptParams.equals("")) {
        response = powerShell.executeScript(srcReader, scriptParams);
    } else {
        response =  powerShell.executeScript(srcReader);
    }
```

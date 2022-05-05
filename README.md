# Transforminator tool

This tool is to test Freemarker and Xslt templates that can be used in message tranformations in the opentunnel environment. The tool is also suitable for other environments but has some specific functionalities to support the Opentunnel runtime variables.

The tool requires a minimal java version 1.8.

- [Transforminator tool](#transforminator-tool)
  - [Usage](#usage)
    - [Options](#options)
  - [Debugging](#debugging)
  - [tunnelvariables or templatevariables](#tunnelvariables-or-templatevariables)
    - [constant variables](#constant-variables)
    - [file content](#file-content)
    - [tunnelfunctions](#tunnelfunctions)
    - [function](#function)
    - [XPath and jsonpath](#xpath-and-jsonpath)
    - [xmlns  xml namespace](#xmlns--xml-namespace)
    - [header](#header)
    - [attachments](#attachments)
    - [multipart formdata](#multipart-formdata)
    - [exitpoint](#exitpoint)
    - [expressions (Not OpenTunnel compatible)](#expressions-not-opentunnel-compatible)
  - [vars.tunnelvars file](#varstunnelvars-file)
    - [The Freemarker template.ftl](#the-freemarker-templateftl)
    - [Example Groovy script](#example-groovy-script)
    - [Example tasks for visual studio code](#example-tasks-for-visual-studio-code)


## Usage

` java -jar ~/tools/Transforminator.jar -a vars.txt -t template.ftl -x input.xml -o output.xml -g groovy/lib/`

### Options

```bash
version 0.3

usage: java -jar Transforminator.jar
 -a,--vars <arg>             vars input file path
 -t,--template <arg>         freemarker template file
 -x,--xml-input <arg>        xml input filepath
 -j,--json-input <arg>       json input filepath
 -i,--text-input <arg>       text input filepath
 -o,--output <arg>           output file
 -f,--function-dir <arg>     Function dir
 -g,--groovy-lib-dir <arg>   groovy lib path
 -r,--resolve <arg>          outputfile with resolved variables
 -s,--xsd-schema <arg>       xsd schema filepath
 -v,--validate               validate resulting xml file
```

When a templatefile with the `.xsl` extension is supplied `-t` the xslt generator wil be invoked. When you use a template with `.ftl` extension the freemarker generator is used. Both generators can make use of the supplied tunnelvars.

The tunnelvars are supplied using the `-a` option to the command. There is de posibitity to save the resolved tunnelvars to a separate file with the `-r` option. In the file all resolved variable values are stored so you can see what exact values were used when the template was evalueated. (works for freemarker and xslt templates)

You can use either use the xml-input `-x`, json-input `-j` or text-input `-i`, never together.
The xml dom is made available through `payloadElement`  this is the objectrepresentation for the xml data structure. When you supply a json file the ``xpath://`` tunnelvariables will work just like with an xml input file.

The output file `-o` can be of any type for freemarker, for xslt it will be mostly `.xml`

When generating xml output you can validate the generated xml using a specified schema with the `-s` option combined with the `-v ` option. You may also use the `-v` option with no schema, the xml is just validated to be valid xml (no schema is used then).

There are some special options to the Transforminator tool. You may define a directory where you keep your groovy functions `-f`. In this way you do not have to copy them from project to project.
From your groovy functions there is the possibility to use additional `.jar` libraries. You may supply this library-path with the `-g` option. When Opentunnel or some other environment uses some specific api's from the freemarker templates, you can use them from the `.jar` library.

## Debugging
When an error occurs in the template all known variables are dumped in the output file.
This way the user has an insight what the variables contained at the time of the error.

Also ther is a possibility to make the tool dump all variables at a specific point by adding
`${DEBUG()}` at some point in your template. Then all variables are dumped in the outputfile at the 
point of the DEBUG statement.

You may add a variable to the DEBUG statement so only that variable is dumped to your output file.
`${DEBUG("varname")}`
when the first argument starts with a : the folowing character is used to format the debug string;
- `:l` debug format is single line only

`${DEBUG("varname")}` prints like

```
_________________ DEBUG info __________________
{
  "DebugVariables" : {
    "varname" : "NUL0"
  }
}
_______________ END DEBUG info ________________
```

`${DEBUG(":l","varname")}` prints like
```
DEBUG -> {"DebugVariables":{"i":"NUL0"}}
```
For simple variable structures the :l is more preferable.

## tunnelvariables or templatevariables

You can add your tunnelvars to the `vars.tunnelvars` file
You may also use concatenation of tunnelvars to create a new variable.

``name=const://testtunnelvar://extension`` is invalid

### constant variables
``name=value`` or ``name=const://value``

### file content
Use the content of a file as the value:

``name=file://filepath``

### tunnelfunctions

You may use functions in your template by first adding the definition to the tunnelvars file.
The value is then the file path to the script.

``function://function_name=groovy://script.groovy``

The groovy script has all the tunnelvars available as variables 
and also accepts parameters from your freemarker template. 
These parameters can be just a string or another tunnelvar.

So in your template use the tunnelfunction like this;

``${tunnelFunction("helloworld",myname)}``

⚠️When the tunnelfunction is used from the freemarker template the groovy script is then executed. The groovy engine in Transforminator has only access to some specific OpenTunnel classes
So in the case you do use specific OT classes most of them may not work here.

When you might need java jar libraries you may place them in a directory and use the `-g` option to define the path to that directory. The Transforminator scans and loads all jar files in that directory.

Look below for an example groovy script.

### function

When you have defined your function with ``function://name=groovy://script.groovy`` You can call this function
from the next tunnelvar;

``varname=function://`function_name/varx``

The varname is then initialized with the return value of your groovy script.

### XPath and jsonpath

When working with xml or json input files you may also use the xpath:// or jsonpath:// directive. ``varname=xpath://ns:element`` It should work just like in opentunnel.
The jsonpath works just like the xpath, the json payload is first transformed to a xml-object model and then the jsonpath (xpath) is resolved.

### xmlns  xml namespace
When using xpath variables, you may need to add the `xmlns://` as well.

`xmlns://soap=http://schemas.xmlsoap.org/soap/envelope/`

you can then use the namespace in your xpath variables.

When using xpath from your template you need to add the xmlnamespace in the template ftl header as well.

```
<#ftl ns_prefixes={"e":"http://example.com/ebook"}>
```
### header

Freemarker transformations used by Opentunnel often make use of the headers that came with
the received message. in the tunnelvars file you can add these headers in the folowing way.

``header://soapaction=myAction``

These headers can then be used in groovy scripts by the folowing Freemarker Bean:
The MessageHeader class is not a full implementation like in OpenTunnel. It is more like a mock class to mimic the OpenTunnel version.
``` groovy
import nl.jnc.gateway.message.dto.MessageHeader;

def result = [:]

MessageHeader mh = messageheader;

mh.transportParametersMap.each { key, val ->
    if (!key.startsWith('opentunnel')) {
        result[key] = val
        println(key)
    }
}

return result
```

### attachments

You may use attachments from the tunnelvars file. You can just add a specific value to them or use the content of a specific file.

``attachment://name=file://filepath``

``attachment://name=value``

The attachment is then available through the `form.attachment.1`

### multipart formdata

Multipart formdata that is not a file attachment is parsed as a ``url_`` tunnelvar. You may use a file or a value to parse.

``url_name=file://filepath``

``url_name=value``

### exitpoint

``exitpoint://EXP_OUT=GET:application/json:http://localhost:8441/mock/service``

This entry needs the METHOD:content-type:URL

From your template you can use the Opentunnel `deliveryPointAttachment` function to send a message to the exitpoint using its code/name

```ftl
<#assign val = deliveryPointAttachment(payload,"EXP_OUT")>
```

### expressions (Not OpenTunnel compatible)
For convenience there is also the posibility to use expressions. This is not supported bij Opentunnel so do not use it in your tunnel runtimevariables.
The expressions are based on a java like syntax.
 
```ftl
hello=test
expr=expression://hello.length
```

expr is then resolver to 4

```aidl
expr=expression://hello.length + 10
```

epr = 14

 
## vars.tunnelvars file

The file contains a set of key/value pairs that normaly are configured in the opentunnel as tunnelvars. Or that are parsed to the template in the runtime.
 

```properties
#namespaces
xmlns://test=http://www.omgevingswet.nl/koppelvlak/stam-v2/IMAM

exitPoint=loopback.generic

businessTimestamp=20200908123049714
messageId=zWbs5X7ITY68gYPYH4iMmg@172.20.0.3
partyId=00000001805329535000
processId=8QHheAWCTBKxKGGYcZnCpA
service=NVT

timestamp=20200908123049714
timestampDay=8
timestampDay2=08
timestampHour=12
timestampMinute=30
timestampMonth=9
timestampMonth2=09
timestampSecond=49
timestampYear=2020

jsonfile=file://test.json

myname=Michiel
tunnelfunction://helloworld=groovy://test.groovy

tunnelfunction://boring=boring things happening
tunnelfunction://jsoninput=file://test.json

attachment://hello_json={"dom":"doen"}
attachment://hello_xml=<hello>xml</hello>
url_newfile=file://input.xml

xptest=xpath:////vx:type

```

### The Freemarker template.ftl

```freemarker
${tunnelFunction("helloworld",myname)}
${tunnelFunction("boring")}
${tunnelFunction("jsoninput")}
------------------------------------------------------------
get ALL available keys in the datamodel

<#assign formAttachments = {}>
<#assign multipart = {}>
<#list .data_model?keys as key>
  ${key}
  <#if key?matches("form.attachment.\\d+")>
    <#assign formAttachments += {key:.data_model[key]} >
  </#if>
  <#if key?starts_with("url_")>
  	<#assign multipart += {key:.data_model[key]}>
  </#if>
</#list>
------------------------------------------------------------
get ALL multipart data

<#list multipart?keys as key>
	${key}
	${multipart[key]}
=====
</#list>
------------------------------------------------------------
get ALL attachements by its names

<#list formAttachments?keys as key>
    <#assign att = attachments(messageheader,formAttachments[key])>
    <#list att?keys as ha>
=====
        <#list att[ha]?keys as hak>
            <#if att[ha][hak]?is_string || att[ha][hak]?is_boolean || att[ha][hak]?is_number || att[ha][hak]?is_date>
${hak} : ${att[ha][hak]}
            </#if>
        </#list>
=====        
    </#list>
</#list>
------------------------------------------------------------
get ALL attachments by their keys in attachments

<#assign allAttachments = attachments(messageheader)>
<#list allAttachments?keys as key>
keyname:     ${key}
partname:    ${allAttachments[key].partName}
contentType: ${allAttachments[key].contentType}
data:        ${allAttachments[key].data}
</#list>
```

### Example Groovy script

```groovy
class Greeter {
    String sayHello(name) {
        def greet = "Hello, Transforminator " + name
        greet
    }
}

def gr = new Greeter()

gr.sayHello(args[1])
```

### Example tasks for visual studio code

To use the Transforminator tool from vscode and test your templates you need to add a folder named .vscode
In the .vscode folder create a file named tasks.json and add the following content.
Now when you have all files in place press ctrl-shift-B and Transforminator will run with the options supplied in the tasks.json
```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "current file",
      "type": "shell",
      "command": "java -jar ~/tools/Transforminator.jar -t ${fileBasename} -x verstrekVordering_voorbeeld.xml -a vars.tunnelvars -o ${fileBasenameNoExtension}.xml",
      "problemMatcher": [
        {
          "owner": "Transforminator",
          "fileLocation": [
            "relative",
            "${workspaceFolder}"
          ],
          "pattern": {
            "regexp": ".*?template\\s\"(.*)\".*line\\s(\\d+).*column\\s(\\d+).*$",
            "file": 1,
            "line": 2,
            "column": 3
          },
          "severity": "error"
        }
      ],
      "group": {
        "kind": "build",
        "isDefault": true
      },
      "presentation": {
        "clear": true
      }
    }
  ]
}

```
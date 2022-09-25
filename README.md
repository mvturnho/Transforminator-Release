# Transforminator tool

This tool is to test Freemarker and Xslt templates that can be used in message tranformations in the opentunnel environment. The tool is also suitable for other environments but has some specific functionalities to support the Opentunnel runtime variables.
For convenience you best use this tool with the [Transforminator Visualstudio code extension](https://marketplace.visualstudio.com/items?itemName=Qris.transforminator). 

![Settings](images/transforminator-vscode-extension.png)

The tool requires a minimal java version 1.8.

- [Transforminator tool](#transforminator-tool)
  - [Usage](#usage)
    - [Options](#options)
    - [Visual studio code](#visual-studio-code)
  - [Debugging](#debugging)
  - [tunnelvariables or templatevariables](#tunnelvariables-or-templatevariables)
    - [constant variables](#constant-variables)
    - [file content](#file-content)
    - [tunnelfunctions](#tunnelfunctions)
    - [function](#function)
    - [mappings](#mappings)
    - [XPath and jsonpath](#xpath-and-jsonpath)
    - [xmlns  xml namespace](#xmlns--xml-namespace)
    - [header](#header)
    - [Freemarkerversion](#freemarkerversion)
    - [xslt factoryname](#xslt-factoryname)
    - [attachments](#attachments)
    - [multipart formdata](#multipart-formdata)
    - [exitpoint](#exitpoint)
    - [expressions (Not OpenTunnel compatible)](#expressions-not-opentunnel-compatible)
  - [Example setup](#example-setup)
    - [vars.tunnelvars file](#varstunnelvars-file)
    - [The Freemarker template.ftl](#the-freemarker-templateftl)
    - [Example Groovy script](#example-groovy-script)
    - [Input xml](#input-xml)

## Usage

` java -jar ~/tools/Transforminator.jar -a vars.tunnelvars -t template.ftl -x input.xml -o output.xml`

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

### Visual studio code

This tool is best used in combination with [Visual Studio Code](https://code.visualstudio.com/download). Install the [extension Transforminator](https://marketplace.visualstudio.com/items?itemName=Qris.transforminator) to add additional editor features.

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

### mappings

To make use of the opentunnel mapping functionality you may add your mapping tables by using the `mapping://` definition.
The most convenient way is to use a csv file for input to your mapping.

``mapping://maptable=file://mappinginput.csv``

After this the mappingtable with name `maptable` is available to create your tunnelvars;

```
mapping://maptable=file://mapping_input_file.csv
someCode=const://code
varname=function://mapping/maptable/someCode
```
The variable `varName` now contains the value that was in the csv file for key code

In your Freemarker template you can also access the mappings in the following way.

``mappingName.map["keyname"]``


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

### Freemarkerversion

The freemarker engine supports several versions. The freemarkerversion can be definded by the OpenTunnel version

`opentunnelVersion=3.0` Gives freemarkerversion 2.3.30 All other OT versions lead to freemarkerversion 2.3.28

supported OT versions are:
- ``3.0`` uses Freemarker version 2.3.30
- ``2.4`` uses Freemarker version 2.3.28
- ``2.2`` uses Freemarker version 2.3.28

You may also define the freemarkerversion to be used explicitly.

`freemarkerVersion=2.3.28`

supported freemarker versions are:
- ``2.3.30``
- ``2.3.28``
- ``2.3.27``
- ``2.3.26``
- ``2.3.25``
- ``2.3.24``
- ``2.3.23``
- ``2.3.22``

### xslt factoryname

For the xslt processor you can define what factoryname should be used. You can do this by defining the processor in your tunnelvars file.

- jdk
- xalan
- saxon

``processor=xalan`` or use:

``xslt.transformer.factory=saxon`` the prefered OT way for xslt templates.

Why use saxon?

Saxon an XSLT 2.0 and XSLT 3.0 processor, it is very actively developed and maintained.
Its developer, Dr. Michael Kay is the Editor of the W3C XSLT WG (Working Group) and thus he is probable the one that best understands the XSLT Specification and this shows in Saxon. Any language feature is strictly and precisely implemented -- usually well ahead of other vendors.


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

## Example setup

Here are some examples for use with freemarker.

`java -jar ~/tools/Transforminator.jar -a vars.tunnelvars -t template.ftl -x input.xml -o output.txt`

### vars.tunnelvars file

The file contains a set of key/value pairs that normaly are configured in the opentunnel as tunnelvars. Or that are parsed to the template in the runtime.
 

```properties
#use ${DEBUG()} in your freemarker template to dump vars in your output
#xpath always prefix with xpath:// never only //

#definitions like xmlns:// and header://
xmlns://person=http://test.org/persons/
header://action=soapaction
function://uuid=groovy://UUID.groovy
exitpoint://CODE=GET:text/xml:http://localhost:8441/mock/service
attachment://testinput=file://test.json

#Tunnelvars
uuid=function://uuid
business=transport://soapaction
name=xpath:////person:name
street=xpath:////person:street

business=tunnelvar://nameconst:// - tunnelvar://street
```

### The Freemarker template.ftl

```freemarker
<#ftl encoding='UTF-8' ns_prefixes={"p":"http://test.org/persons/"}>

<#assign zipcode=payloadElement["//p:zipcode"]/>
<#assign next_uuid=tunnelFunction("uuid") />

FULL DEBUG OUTPUT
${DEBUG()}

SINGLE LINE DEBUG OUTPUT
${DEBUG(":l","zipcode","name")}

{
    "user" : {
        "name": "${name}",
        "adress": "${street}",
        "zipcode": ${zipcode}
    }
}
```

### Example Groovy script

```groovy
java.util.UUID.randomUUID().toString()
```

### Input xml

```xml
<person xmlns:p="http://test.org/persons/">
    <p:name>Tibbe</p:name>
    <p:address>
        <p:street>Tibbe-street</p:street>
        <p:zipcode>1825SE</p:zipcode>
    </p:address>
</person>
```
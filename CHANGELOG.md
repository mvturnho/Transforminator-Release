# V1.5

## mappings from freemarker template

You may now access the mappings from the freemarker template directly. (OT3.0 an above)

``${mappingName.map["keyName"]``

## define xslt factory from tunnelvars

Define the xslt factoryname from the tunnelvars file. The xslt processor then uses this factory in the transformation process.

``processor=xalan``

Valid factorynames are:
- jdk
- xalan
- saxon

# V1,4

## bugfixes

sevaral bug fixes and added xslt file extension.

# V1.3

## added mappings to tunnelvars
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

## Bugfixes

- `varname=transport://value?default`   VarParser did not use the defaultvalue when transport was not found

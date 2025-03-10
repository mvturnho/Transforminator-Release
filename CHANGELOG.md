# V2.2.2
## bugfix
IO Error prevented syntax error on freemarker template

# V2.2.1
## memory monitor
- bugfixes
- memory usage for transformers
- error reporting for groovy

# V2.2.0
## groovy support for OT functionality
- exitpoints
- mappings
- dependencies

## Debug optimisations for freemarker


# V1.7

## several bugfixes

- mappings and freemarker transformations
- xslt factory selection

## OT like xslt transformer factory

In your ``variables.tunnelvars`` you may now use the OT ``xslt.transformer.factory`` to define the xslt factory to be used in the transformation.

Possible values are:
- ``jdk``
- ``saxon``
- ``xalan``

## define the OT version to select the correct freemarker version for that OT version

``opentunnelVersion=3.0``

- ``3.0`` uses Freemarker version 2.3.30
- ``2.4`` uses Freemarker version 2.3.28
- ``2.2`` uses Freemarker version 2.3.28

## define the freemarker version to be used.

``freemarkerVersion=2.3.28``

- ``2.3.30``
- ``2.3.28``
- ``2.3.27``
- ``2.3.26``
- ``2.3.25``
- ``2.3.24``
- ``2.3.23``
- ``2.3.22``


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


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



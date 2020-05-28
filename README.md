# ODAM data-package based on JSON-Schema

A data package is a simple container format used to describe and package a collection of data. Defining an explicit schema for structural metadata allows machines to better interpret the data for reuse. Thus, when disseminating data, a file named ‘datapackage.json’ by convention can be added to the collection of your data files. This ‘datapackage.json’ file contains all structural metadata along with unambiguous definitions of all internal elements (e.g. column definitions, units of measurement), through links to accessible (standard) definitions.

ODAM data package schema is very close to the 'Frictionless Data' framework.

 * **odam-data-package.json** : JSON Schema for ODAM data package
 * **odam-data-resource.json** : JSON Schema for ODAM data resource
 * **schema.json** : the simplest schema file for external validation
 * **datapackage-frim1.json** : example of datapackage file.
 
The ‘datapackage.json’ file can be generated directly from the ODAM API by specifying '/datapackage' at the end of the request. By default, the reference to the data files is relative. To have a URL as reference for the data files, it is necessary to add at the end of the request '?links=1'


## Frictionless Data Specifications
* http://specs.frictionlessdata.io/
* https://github.com/frictionlessdata/specs/tree/master/schemas

## json_validate: Validate a json file
https://rdrr.io/cran/jsonvalidate/man/json_validate.html


## Example of a session under R using a data package

```
library(httr)
library(jsonvalidate)
library(jsonlite)

# Get the ODAM data package schema
response <- GET('https://inrae.github.io/odam-docs/json-schema/odam-data-package.json', config(sslversion=6,ssl_verifypeer=1))
schema <- rawToChar(response$content)

# Get structural metadata information in datapackage format (json) 
# for the 'frim1' dataset directly from an ODAM repository (located on https://pmb-bordeaux.fr)
# As the option links is set to 1, we will have the absolute reference for data files (see below)
response <- GET('https://pmb-bordeaux.fr/getdata/json/frim1/datapackage?links=1', config(sslversion=6,ssl_verifypeer=1))
json <- rawToChar(response$content)

# Validate the JSON against the ODAM data package schema
jsonvalidate::validate_json(json, schema)
! [1] TRUE

# Parse the JSON object to a data.frame
metadata <- fromJSON(json)

# List some metadata about the dataset
metadata$resources[ c("name", "title", "identifier", "obtainedFrom", "joinkey") ]
!                 name                                   title identifier obtainedFrom   joinkey
! 1             plants                          Plant features    PlantID         <NA>      <NA>
! 2            samples                         Sample features   SampleID       plants   PlantID
! 3           aliquots                       Aliquots features  AliquotID      samples  SampleID
! 4    cellwall_metabo      Cell wall Compound quantifications  AliquotID     aliquots AliquotID
! 5  cellwall_metaboFW Cell Wall Compound quantifications (FW)  AliquotID     aliquots AliquotID
! 6           activome                       Activome Features  AliquotID     aliquots AliquotID
! 7              pools                Pools of remaining pools     PoolID      samples  SampleID
! 8         qMS_metabo             MS Compounds quantification     PoolID        pools    PoolID
! 9        qNMR_metabo            NMR Compounds quantification     PoolID        pools    PoolID
! 10    plato_hexosesP                       Hexoses Phosphate  AliquotID     aliquots AliquotID
! 11         lipids_AG                               Lipids AG  AliquotID     aliquots AliquotID
! 12         AminoAcid                             Amino Acids  AliquotID     aliquots AliquotID

# List the absolute reference for data files 
metadata$resources[ "path" ]
!                                                           path
! 1             https://pmb-bordeaux.fr/getdata/tsv/frim1/plants
! 2            https://pmb-bordeaux.fr/getdata/tsv/frim1/samples
! 3           https://pmb-bordeaux.fr/getdata/tsv/frim1/aliquots
! 4    https://pmb-bordeaux.fr/getdata/tsv/frim1/cellwall_metabo
! 5  https://pmb-bordeaux.fr/getdata/tsv/frim1/cellwall_metaboFW
! 6           https://pmb-bordeaux.fr/getdata/tsv/frim1/activome
! 7              https://pmb-bordeaux.fr/getdata/tsv/frim1/pools
! 8         https://pmb-bordeaux.fr/getdata/tsv/frim1/qMS_metabo
! 9        https://pmb-bordeaux.fr/getdata/tsv/frim1/qNMR_metabo
! 10    https://pmb-bordeaux.fr/getdata/tsv/frim1/plato_hexosesP
! 11         https://pmb-bordeaux.fr/getdata/tsv/frim1/lipids_AG
! 12         https://pmb-bordeaux.fr/getdata/tsv/frim1/AminoAcid


# Read the 'samples' data file 
M <- read.table(url(metadata$resources[ "path" ]$path[2]), header=TRUE, sep="\t")
dim(M)
! [1] 1288   13
 
# Display an extract 
M[1:10,]
!    SampleID PlantID Truss DevStage FruitAge HarvestDate HarvestHour FruitPosition FruitDiameter FruitHeight FruitFW  FruitDW DW
! 1         1     A26    T5    FF.01    08DPA       40379         0.5             2            NA          NA    0.72 0.090216 NA
! 2         1      C2    T5    FF.01    08DPA       40379         0.5             3            NA          NA    0.56 0.070168 NA
! 3         1     D15    T5    FF.01    08DPA       40379         0.5             4            NA          NA    0.78 0.097734 NA
! 4         1     E19    T5    FF.01    08DPA       40379         0.5             4            NA          NA    0.66 0.082698 NA
! 5         1     E34    T5    FF.01    08DPA       40379         0.5             3            NA          NA     0.7 0.087710 NA
! 6         1     E38    T5    FF.01    08DPA       40379         0.5             3            NA          NA     0.7 0.087710 NA
! 7         1     H29    T5    FF.01    08DPA       40379         0.5             5            NA          NA    1.24 0.155372 NA
! 8         1     H34    T5    FF.01    08DPA       40379         0.5             4            NA          NA    0.86 0.107758 NA
! 9         1     H52    T5    FF.01    08DPA       40379         0.5             5            NA          NA    0.77 0.096481 NA
! 10        1     H61    T5    FF.01    08DPA       40379         0.5             5            NA          NA    0.56 0.070168 NA
```

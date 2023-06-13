# STIRData data consumption pipelines
This repository hosts a set of data transformation and consumption [LinkedPipes ETL] pipelines.
The pipelines serve as a demonstration of how the harmonized data published according to the [STIRData specification] and registered in a data catalog can be used to solve specific data-oriented use cases as an alternative to accessing the data through the user-facing [STIRData portal] application.

## Number of business entities and population by NUTS level 3
[**LP-ETL Pipeline**](<pipelines/Number of business entities (STIRData) and population (Wikidata) by NUTS level 3.jsonld>)
This pipeline produces a dataset with the total population and number of business entities registered in administrative regions (NUTS level 3).

**Inputs:**
1. A data catalog with at least one dataset registered with a distribution in the form of a SPARQL endpoint service conforming to the [STIRData specification].
2. [Wikidata SPARQL endpoint].
3. RDF representation of LAU regions in a [downloadable file](https://data.europa.eu/data/datasets/https-lkod-mff-cuni-cz-zdroj-datove-sady-stirdata-lau-stirdata).

**Outputs:**
1. An RDF representation of the number of business entities and population in NUTS level 3 regions in the form of a [DCV statistical data cube].
2. A CSV representation of the same data cube where each observation is represented as a separate row.

**Expected usage:**

This pipeline serves analysts who compare the population and number of business entities in different NUTS level 3 administrative regions. The output is a set of statistical observations with the two measures (population, number of business entities) in three dimensions:
- NUTS level 3 administrative region
- the date when the measures were computed.
- the source data set used to compute the number of business entities.
  - For data sources about business entities with the same NUTS level 3 on the input, there is a separate observation for each data source.

### Technical implementation steps:
1. *Extract compliant endpoints from the catalog:* The pipeline extracts the SPARQL endpoints compliant with the STIRData business data model specification based on the metadata records in a data catalog, e.g., European Data Portal or STIRData catalog.
2. *Prepare query tasks to get the number of business entities in administrative regions:* Prepares a SPARQL construct query task for each SPARQL endpoint extracted in the previous step. Each task contains a prepared SPARQL construct query that extracts the list of business entities (instances of the class <http://www.w3.org/ns/legal#LegalEntity>), groups them by the administrative unit they are registered in (property path <http://data.europa.eu/m8g/registeredAddress>/<http://data.europa.eu/m8g/adminUnit>) and counts the number of business entities registered in each administrative unit.
  - We expect administrative regions specified at the level of LAU regions.
  - If there is no administrative unit specified a legal entity is registered in, the pipeline tries other possibilities according to the specification (postal address, street address, address of the (registered) site)
3. *Execute query tasks to get the number of business entities in administrative regions:* Executes the prepared query tasks to get the number of business entities in administrative regions for each input endpoint.
4. *Download LAU data:* Downloads a file with the RDF representation of LAU regions from the specified URL.
5. *Transform LAU data:* Transforms the RDF file with LAU regions to an RDF graph. (This is a technical step.)
6. *Merge LAU data:* Creates a single RDF graph with the RDF representation of LAU regions from the file. (This is a technical step.)
7. *Extract population in NUTS from Wikidata:* Extracts the last known population in NUTS level 3 administrative regions from Wikidata.
8. *Union LAU, population, and the number of business entities:* Makes the union of the three pipeline branches, i.e., the number of business entities in LAU regions, data about LAU regions, and population in NUTS level 3 regions. (This is a technical step.)
9. *Prepare output data structure definition:* Prepares the definition of the structure of the statistical data cube with the output data. It defines three dimensions - reference area (NUTS level 3 regions), reference period (the current date), source of data about business entities (the endpoint from which the number of business entities was extracted), and two measures - population and the number of business entities.
10. *Transform output data structure definition to graph:* Transforms the definition of the structure of the statistical data cube to an RDF graph. (This is a technical step.)
11. *Prepare output dataset definition:* Prepares the definition of the output statistical data cube data set.
12. *Transform output dataset definition:* Transforms the definition of the output statistical data cube definition to an RDF graph. (This is a technical step.)
13. *Construct statistical observations:* Constructs the final statistical observations. Each observation has three dimensions and two measures as defined by the data structure definition prepared in Step 9.
  - The number of business entities in NUTS regions is computed as a summary of the number of business entities in the LAU regions in this NUTS region. We take the affiliation of the LAU region to the NUTS region from the RDF representation of the LAU regions extracted in Steps 4-6.
  - The population in NUTS regions is taken from Wikidata in Step 7.
14. *Create final RDF:* Creates the final RDF graph with the output that contains the statistical observations, the statistical data cube as a dataset with the observations, and the definition of its structure. (This is a technical step.)
15. *Store output RDF in a file:* Stores the final RDF in an output TTL file.
16. *Create an output CSV file:* Executes a SPARQL SELECT query that produces the CSV output with the statistical observations. Each observation forms a row in the output CSV file.

## Number of business entities and GDP by NUTS level 3
[**LP-ETL Pipeline**](<pipelines/Number of business entities (STIRdata) and GDP (Eurostat) by NUTS level 3.jsonld>)

This pipeline produces a dataset with the total population and the gross domestic product (GDP) in millions of EUR in administrative regions (NUTS level 3).

**Inputs:**
1. A data catalog with at least one dataset registered with a distribution in the form of a SPARQL endpoint service conforming to the [STIRData specification].
2. [Eurostat GDP NUTS3 ZIP archive](https://ec.europa.eu/eurostat/estat-navtree-portlet-prod/BulkDownloadListing?sort=1&downfile=data%2Fnama_10r_3gdp.sdmx.zip)
3. RDF representation of LAU regions in a [downloadable file](https://data.europa.eu/data/datasets/https-lkod-mff-cuni-cz-zdroj-datove-sady-stirdata-lau-stirdata).

**Outputs:**
1. An RDF representation of the number of business entities and GDP in NUTS level 3 regions in the form of a [DCV statistical data cube].
2. A CSV representation of the same data cube where each observation is represented as a separate row.

**Expected usage:**
This pipeline serves analysts who compare the population and GDP in different NUTS level 3 administrative regions. The output is a set of statistical observations with the two measures (population, GDP in millions of EUR) in three dimensions:
- NUTS level 3 administrative region
- the date when the measures were computed.
- the source data set used to compute the number of business entities.
  - For data sources about business entities with the same NUTS level 3 on the input, there is a separate observation for each data source.

### Technical implementation steps:
1. _Extract compliant endpoints from the catalog:_ The pipeline extracts the SPARQL endpoints compliant with the STIRData business data model specification based on the metadata records in a data catalog, e.g., European Data Portal or STIRData catalog.
2. _Prepare query tasks to get the number of business entities in administrative regions:_ Prepares a SPARQL construct query task for each SPARQL endpoint extracted in the previous step. Each task contains a prepared SPARQL construct query that extracts the list of business entities (instances of the class <http://www.w3.org/ns/legal#LegalEntity>), groups them by the administrative unit they are registered in (property path <http://data.europa.eu/m8g/registeredAddress>/<http://data.europa.eu/m8g/adminUnit>) and counts the number of business entities registered in each administrative unit.
  - We expect administrative regions specified at the level of LAU regions.
  - If there is no administrative unit specified a legal entity is registered in, the pipeline tries other possibilities according to the specification (postal address, street address, address of the (registered) site)
3. _Execute query tasks to get the number of business entities in administrative regions:_ Executes the prepared query tasks to get the number of business entities in administrative regions for each input endpoint.
4. _Download LAU data:_ Downloads a file with the RDF representation of LAU regions from the specified URL.
5. _Transform LAU data:_ Transforms the RDF file with LAU regions to an RDF graph. (This is a technical step.)
6. _Merge LAU data:_ Creates a single RDF graph with the RDF representation of LAU regions from the file. (This is a technical step.)
7. _Download Eurostat GDP NUTS3 ZIP archive statistics:_ Downloads the ZIP archive with the GPD statistics in NUTS regions.
8. _Decompress Eurostat GDP NUTS3 ZIP archive statistics:_ Decompresses the downloaded ZIP archive (This is a technical step.)
9. _Transform Eurostat GDP NUTS3 XML SDMX file to RDF file:_ Transforms the input statistical file in XML SDMX Eurostat format to RDF using an XSLT script.
10. _Merge Eurostat GDP NUTS3 RDF data:_ Merges the output RDF data to a single graph. (This is a technical step.)
11. _Union LAU, GDP, and the number of business entities:_ Makes the union of the three pipeline branches, i.e., the number of business entities in LAU regions, data about LAU regions, and GDP in NUTS level 3 regions. (This is a technical step.)
12. _Prepare output data structure definition:_ Prepares the definition of the structure of the statistical data cube with the output data. It defines three dimensions - reference area (NUTS level 3 regions), reference period (the current date), source of data about business entities (the endpoint from which the number of business entities was extracted), and two measures - GDP and the number of business entities.
13. _Transform output data structure definition to graph:_ Transforms the definition of the structure of the statistical data cube to an RDF graph. (This is a technical step.)
14. _Prepare output dataset definition:_ Prepares the definition of the output statistical data cube data set.
15. _Transform output dataset definition:_ Transforms the definition of the output statistical data cube definition to an RDF graph. (This is a technical step.)
16. _Construct statistical observations:_ Constructs the final statistical observations. Each observation has three dimensions and two measures as defined by the data structure definition prepared in Step 12.
  - The number of business entities in NUTS regions is computed as a summary of the number of business entities in the LAU regions in this NUTS region. We take the affiliation of the LAU region to the NUTS region from the RDF representation of the LAU regions extracted in Steps 4-6.
  - The population in NUTS regions is taken from Wikidata in Steps 7-10.
17. _Create final RDF:_ Creates the final RDF graph with the output that contains the statistical observations, the statistical data cube as a dataset with the observations, and the definition of its structure. (This is a technical step.)
18. _Store output RDF in a file:_ Stores the final RDF in an output TTL file.
19. _Create an output CSV file:_ Executes a SPARQL SELECT query that produces the CSV output with the statistical observations. Each observation forms a row in the output CSV file.

## Number of business entities by NACE activity and NUTS level 3
[**LP-ETL Pipeline**]()

**Inputs:**
1. 

**Outputs:**
1. 

**Expected usage:**

### Technical implementation steps:
1. 
2. 

## Number of business entities registered in given months by activity
[**LP-ETL Pipeline**]()

**Inputs:**
1. 

**Outputs:**
1. 

**Expected usage:**

### Technical implementation steps:
1. 
2. 

## Searching for business entities across STIRdata data sources, Wikidata and DBpedia
[**LP-ETL Pipeline**]()

**Inputs:**
1. 

**Outputs:**
1. 

**Expected usage:**

### Technical implementation steps:
1. 
2. 

## Profiling pipeline
[**LP-ETL Pipeline**]()

**Inputs:**
1. 

**Outputs:**
1. 

**Expected usage:**

### Technical implementation steps:
1. 
2. 

## Acknowledgements

![EU emblem](https://wayback.archive-it.org/12090/20221202205740im_/https://ec.europa.eu/inea/sites/default/files/en_square_cef_logo.png)
STIRData is co-financed by the Connecting Europe Facility Programme of the European Union, under GA n. INEA/CEF/ICT/A2019/2063078.

[LinkedPipes ETL]: https://etl.linkedpipes.com "LinkedPipes ETL"
[STIRData specification]: https://stirdata.github.io/data-specification/ "STIRData business data model"
[STIRData portal]: https://portal.stirdata.eu "STIRData portal"
[Wikidata SPARQL endpoint]: https://query.wikidata.org/sparql "Wikidata Query Service SPARQL endpoint"
[DCV statistical data cube]: https://www.w3.org/TR/vocab-data-cube/#data-cubes "Data Cube Vocabulary data cube"
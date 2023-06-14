# STIRData data consumption pipelines
This repository hosts a set of data transformation and consumption [LinkedPipes ETL] pipelines.
The pipelines serve as a demonstration of how the harmonized data published according to the [STIRData specification] and registered in a data catalog can be used to solve specific data-oriented use cases as an alternative to accessing the data through the user-facing [STIRData portal] application.

A [LinkedPipes ETL] instance can be deplyed using Docker.
The [Docker] based deployment requires [Docker Compose].
Then, it can be deployed from the `main` branch like this:
```
curl https://raw.githubusercontent.com/linkedpipes/etl/main/docker-compose.yml | docker-compose -f - up
```
When deployed, LP-ETL runs on http://localhost:8080 and is ready to import pipelines.

For custom deployments, see the [full deployment documentation](https://github.com/linkedpipes/etl/tree/main#installation-and-startup). Once deployed, see the [user documentation](https://etl.linkedpipes.com/documentation/) and [tutorials](https://etl.linkedpipes.com/tutorials/).
The [documentation of the individual LP-ETL components](https://etl.linkedpipes.com/components/) is also available directly from the component's configuration dialog.

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
1. *Extract compliant endpoints from the catalog:* The pipeline extracts the SPARQL endpoints compliant with the STIRData business data model specification based on the metadata records in a data catalog, e.g., data.europa.eu or STIRData catalog.
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
1. _Extract compliant endpoints from the catalog:_ The pipeline extracts the SPARQL endpoints compliant with the STIRData business data model specification based on the metadata records in a data catalog, e.g., data.europa.eu or STIRData catalog.
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
[**LP-ETL Pipeline**](<pipelines/Number of business entities by NACE activity and NUTS level 3 (STIRData).jsonld>)

This pipeline produces a dataset with the number of business entities with a given NACE activity registered in a given administrative region (NUTS level 3).

**Inputs:**
1. A data catalog with at least one dataset registered with a distribution in the form of a SPARQL endpoint service conforming to the [STIRData specification].
2. RDF representation of LAU regions in a [downloadable file](https://data.europa.eu/data/datasets/https-lkod-mff-cuni-cz-zdroj-datove-sady-stirdata-lau-stirdata).

**Outputs:**
1. An RDF representation of the number of business entities with NACE activity in a NUTS level 3 region in the form of a [DCV statistical data cube].
2. A CSV representation of the same data cube where each observation is represented as a separate row.

**Expected usage:**
This pipeline serves analysts who compare the number of business entities with different NACE activities and in different NUTS level 3 administrative regions. The output is a set of statistical observations with the measure of the number of business entities in four dimensions:
- NUTS level 3 administrative region
- NACE2 activity
- the date when the measures were computed.
- the source data set used to compute the number of business entities.
    - For data sources about business entities with the same NUTS level 3 on the input, there is a separate observation for each data source.

**Pre-conditions:**
The pipeline expects that if a business entity is classified in a given NACE activity, it is not classified in any broader activity.

### Technical implementation steps:
1. _Extract compliant endpoints from the catalog:_ The pipeline extracts the SPARQL endpoints compliant with the STIRData business data model specification based on the metadata records in a data catalog, e.g., data.europa.eu or STIRData catalog.
2. _Prepare query tasks to get the number of business entities per activity and administrative region:_ Prepares a SPARQL construct query task for each SPARQL endpoint extracted in the previous step. Each task contains a prepared SPARQL construct query that extracts the list of business entities (instances of the class <http://www.w3.org/ns/legal#LegalEntity>), groups them by the administrative unit they are registered in (property path <http://data.europa.eu/m8g/registeredAddress>/<http://data.europa.eu/m8g/adminUnit>) and by their business activities, and counts the number of business entities by the administrative units and activities.
    - We expect administrative regions specified at the level of LAU regions.
    - If there is no administrative unit specified a legal entity is registered in, the pipeline tries other possibilities according to the specification (postal address, street address, address of the (registered) site)
    - We expect that if a business entity is classified in a given NACE activity, it is not classified in any broader activity.
3. _Execute query tasks to get the number of business entities per activity and administrative region:_ Executes the prepared query tasks to get the number of business entities for each input endpoint.
4. _Download LAU data:_ Downloads a file with the RDF representation of LAU regions from the specified URL.
5. _Transform LAU data:_ Transforms the RDF file with LAU regions to an RDF graph. (This is a technical step.)
6. _Merge LAU data:_ Creates a single RDF graph with the RDF representation of LAU regions from the file. (This is a technical step.)
7. _Union LAU and the number of business entities:_ Makes the union of the two pipeline branches, i.e., the number of business entities in LAU regions and data about LAU regions. (This is a technical step.)
8. _Prepare output data structure definition:_ Prepares the definition of the structure of the statistical data cube with the output data. It defines four dimensions - reference area (NUTS level 3 regions), activity (NACE2), reference period (the current date), source of data about business entities (the endpoint from which the number of business entities was extracted), and the measure of the number of business entities.
9. _Transform output data structure definition to graph:_ Transforms the definition of the structure of the statistical data cube to an RDF graph. _(This is a technical step.)_
10. _Prepare output dataset definition:_ Prepares the definition of the output statistical data cube data set.
11. _Transform output dataset definition:_ Transforms the definition of the output statistical data cube definition to an RDF graph. (This is a technical step.)
12. _Construct statistical observations:_ Constructs the final statistical observations. Each observation has four dimensions and one measure as defined by the data structure definition prepared in Step 8.
    - The number of business entities per activity and NUTS region is computed as a summary of the number of business entities in the LAU regions in this NUTS region. We take the affiliation of the LAU region to the NUTS region from the RDF representation of the LAU regions extracted in Steps 4-6.
13. _Create final RDF:_ Creates the final RDF graph with the output that contains the statistical observations, the statistical data cube as a dataset with the observations, and the definition of its structure. (This is a technical step.)
14. _Store output RDF in a file:_ Stores the final RDF in an output TTL file.
15. _Create an output CSV file:_ Executes a SPARQL SELECT query that produces the CSV output with the statistical observations. Each observation forms a row in the output CSV file.

## Number of business entities registered in given months by activity
[**LP-ETL Pipeline**](<pipelines/Number of business entities registered in given months by activity.jsonld>)

This pipeline produces a dataset with the number of newly registered business entities in given months with a given NACE activity.

**Inputs:**
1. A data catalog with at least one dataset registered with a distribution in the form of a SPARQL endpoint service conforming to the [STIRData specification].
2. Required months of new registrations.

**Outputs:**
1. An RDF representation of the number of newly registered business entities in given months with NACE activity in the form of a [DCV statistical data cube].
2. A CSV representation of the same data cube where each observation is represented as a separate row.

**Expected usage:**
This pipeline serves analysts who compare the number of business entities newly registered in given months distinguished by NACE activities. The months are defined as a parameter of the pipeline. The output is a set of statistical observations with the measure of the number of newly registered business entities in three dimensions:
- NACE2 activity
- the month of the registration.
- the source data set used to compute the number of business entities.
  - For data sources about business entities with the same NUTS level 3 on the input, there is a separate observation for each data source.

### Technical implementation steps:
1. _Set registration months:_ Allows the consumer of the pipeline output to set the required months of new registrations. It is possible to set more different months. Each is set as a separate xsd:date value with the first day of the month.
2. _Transform registration months to graph:_ Transforms the configuration of the months to an RDF graph that can be further processed in the pipeline. _(This is a technical step.)_
3. _Extract compliant endpoints from the catalog:_ The pipeline extracts the SPARQL endpoints compliant with the STIRData business data model specification based on the metadata records in a data catalog, e.g., data.europa.eu or STIRData catalog.
4. _Prepare query tasks to get the number of registered business entities per activity:_ Prepares a SPARQL construct query task for each SPARQL endpoint extracted in the previous step. Each task contains a prepared SPARQL construct query that extracts the number of business entities (instances of the class <http://www.w3.org/ns/legal#LegalEntity>) grouped by their business activities and months of their registration.
    1. We expect administrative regions specified at the level of LAU regions.
    2. If there is no administrative unit specified a legal entity is registered in, the pipeline tries other possibilities according to the specification (postal address, street address, address of the (registered) site)
    3. We expect that if a business entity is classified in a given NACE activity, it is not classified in any broader activity.
    4. Required months must be set in step 1 above.
5. _Execute query tasks to get the number of registered business entities per activity:_ Executes the prepared query tasks to get the number of business entities for each input endpoint.
6. _Prepare output data structure definition:_ Prepares the definition of the structure of the statistical data cube with the output data. It defines three dimensions - activity (NACE2), reference period (the current date), source of data about business entities (the endpoint from which the number of business entities was extracted), and the measure of the number of newly registered business entities.
7. _Transform output data structure definition to graph:_ Transforms the definition of the structure of the statistical data cube to an RDF graph. (This is a technical step.)
8. _Prepare output dataset definition:_ Prepares the definition of the output statistical data cube data set. It dynamically generates the IRI of the output data cube based on the specified months. I.e., it generates a different IRI for each specification of required months.
9. _Construct statistical observations:_ Constructs the final statistical observations.
10. _Create final RDF:_ Creates the final RDF graph with the output that contains the statistical observations, the statistical data cube as a dataset with the observations, and the definition of its structure. (This is a technical step.)
11. _Prepare configuration of the output RDF file:_ It dynamically prepares the configuration of the following step that sets the output RDF file name specifically for the input combination of registration months. (This is a technical step.)
12. _Store output RDF in a file:_ Stores the final RDF in an output TTL file.
13. _Create an output CSV file:_ Executes a SPARQL SELECT query that produces the CSV output with the statistical observations. Each observation forms a row in the output CSV file.

## Searching for business entities across STIRdata data sources, Wikidata and DBpedia
[**LP-ETL Pipeline**](<pipelines/Searching for business entities across STIRdata data sources, Wikidata and DBpedia.jsonld>)

This pipeline produces a dataset with the list of business entities with the given identifiers. It provides the latest number of employees and net profit for each business entity according to Wikidata and a description of the entity in different languages according to DBpedia. The pipeline first searches for the entities in all known data sources compliant with [STIRData specification]. It then searches for the entities using the identifiers in Wikidata, followed by the look up in DBpedia.

**Inputs:**
1. A data catalog with at least one dataset registered with a distribution in the form of a SPARQL endpoint service conforming to the [STIRData specification].
2. Identifiers to search the business entities.

**Outputs:**
1. A JSON representation of the found business entities.

**Expected usage:**
This pipeline demonstrates how concrete business entities can be looked up using their identifiers in different STIRData-compliant data sources, Wikidata and DBpedia, and how the discovered records can be combined together and output as a plain simple JSON file.

### Technical implementation steps:
1. _Set identifiers:_ Allows the consumer of the pipeline output to set the required identifiers. The identifiers are specified as simple string values without any preference for the data source where the corresponding business entity should be looked up.
2. _Transform identifiers to graph:_ Transforms the configuration of the identifiers to an RDF graph that can be further processed in the pipeline. _(This is a technical step.)_
3. _Extract compliant endpoints from the catalog:_ The pipeline extracts the SPARQL endpoints compliant with the STIRData business data model specification based on the metadata records in a data catalog, e.g., data.europa.eu or STIRData catalog.
4. _Prepare query tasks to search for business entities by their identifier (STIRdata):_ Prepares a SPARQL construct query task for each SPARQL endpoint extracted in the previous step and each identifier specified in Step 1.
5. _Execute query tasks to search for business entities by their identifier (STIRdata):_ Executes the prepared query tasks to search for the business entities.
6. _Prepare query tasks to search for business entities by their identifier (Wikidata):_ Prepares a SPARQL construct query task for each identifier specified in Step 1 for Wikidata.
    - The prepared query does not consider one fixed identifying property, as many properties are used for business entity identification in Wikidata. Therefore, it considers Wikidata properties that are instances of some of the predefined business-identifying property types.
7. _Execute query tasks to search for business entities by their identifier (Wikidata):_ Executes the prepared query tasks to search for the business entities.
8. _Prepare query tasks to search for business entities by their identifier (DBpedia):_ Prepares a SPARQL construct query task for each identifier specified in Step 1 for DBpeda.
    - The query reuses the Wikidata IRIs discovered in Step 7 and owl:sameAs statements used in DBpedia linking to Wikidata IRIs.
9. _Execute query tasks to search for business entities by their identifier (DBpedia):_ Executes the prepared query tasks to search for the business entities.
10. _Construct output representation of business entities:_ Unifies the outputs of the previous look-up steps and transforms them to an RDF representation corresponding to the aimed JSON output structure. (This is a technical step.)
11. _Construct final JSON output file:_ Shapes the RDF representation to the final JSON representation.

## Profiling pipeline
[**LP-ETL Pipeline**](https://github.com/STIRData/data-harmonisation/blob/main/validation.jsonld)

**Inputs:**
1. A data catalog with at least one dataset registered with a distribution in the form of a SPARQL endpoint service conforming to the [STIRData specification].

**Outputs:**
1. A validation report in HTML, as can be seen in https://stirdata.opendata.cz/validation/

**Expected usage:**
This pipeline demonstrates how datasets according to the [STIRData specification] can be found in a data catalog and profiled on compliance with the specification, producing a report in HTML.

### Technical implementation steps:
1. _Extract compliant endpoints from the catalog:_ The pipeline extracts the SPARQL endpoints compliant with the STIRData business data model specification based on the metadata records in a data catalog, e.g., data.europa.eu or STIRData catalog.
2. _Prepare query tasks to count instances of LegalEntities with different relevant optional properties:_ For each relevant set of optional properties, queries for numbers of their instances are prepared for each found dataset.
3. _Execute the queries:_ The prepared queries are executed.
4. _Prepare query tasks to extract examples of instances missing selected properties:_ For each relevant property, queries for samples of LegalEntities missing those properties are created.
5. _Execute the queries:_ The prepared queries are executed.
6. _Compute indicators:_ Based on the query results, indicators usable for the HTML report are computed.
7. _Create HTML report:_ The HTML report is created based on the computed indicators.

## Acknowledgements

![EU emblem](https://wayback.archive-it.org/12090/20221202205740im_/https://ec.europa.eu/inea/sites/default/files/en_square_cef_logo.png)
STIRData is co-financed by the Connecting Europe Facility Programme of the European Union, under GA n. INEA/CEF/ICT/A2019/2063078.

[LinkedPipes ETL]: https://etl.linkedpipes.com "LinkedPipes ETL"
[STIRData specification]: https://stirdata.github.io/data-specification/ "STIRData business data model"
[STIRData portal]: https://portal.stirdata.eu "STIRData portal"
[Wikidata SPARQL endpoint]: https://query.wikidata.org/sparql "Wikidata Query Service SPARQL endpoint"
[DCV statistical data cube]: https://www.w3.org/TR/vocab-data-cube/#data-cubes "Data Cube Vocabulary data cube"
[Docker]: https://www.docker.com/
[Docker Compose]: https://docs.docker.com/compose/

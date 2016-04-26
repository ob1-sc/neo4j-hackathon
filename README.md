# neo4j-hackathon
Neo4J Hackathon - Neo Noobs - London Crime Discovery Visualisation 

## Data Set
* Available from: https://data.police.uk/data/
* Sample data set:
* Cypher load script:
```
LOAD CSV WITH HEADERS FROM "file:///crimes.csv" AS row
WITH row WHERE row.Location IS NOT NULL AND row.Latitude IS NOT NULL AND row.Longitude IS NOT NULL AND row.Month IS NOT NULL AND row.`Falls within` IS NOT NULL AND row.`Reported by` IS NOT NULL AND row.`Crime type` IS NOT NULL AND row.`Crime ID` IS NOT NULL
MERGE (location:Location {name: row.Location})
MERGE (point:Point {lat: row.Latitude, lon: row.Longitude})
MERGE (month:Month {name: row.Month})
MERGE (juris:Jurisdiction {name: row.`Falls within`})
MERGE (report:Reporter {name: row.`Reported by`})
MERGE (crimetype:CrimeType {name: row.`Crime type`})
MERGE (crime:Crime {crime_id: row.`Crime ID`})
SET crime.lsoa_name = row.`LSOA name`,
    crime.lsoa_code = row.`LSOA code`
MERGE (crime)-[:OCCURED_AT]->(point)
MERGE (point)-[:LOCATED_IN]->(location)
MERGE (location)-[:HAS_JURISDICTION]->(juris)
MERGE (crime)-[:OCCURED_DURING]->(month)
MERGE (crimetype)<-[:IS_TYPE]-(crime)
MERGE (crime)-[:REPORTED_BY]->(report)
```

## Data Preparation
* Added CrimeType float longitude and latitude properties to use distance/point functions:
```
$MATCH (n:Point) 
SET n.longitude = tofloat(n.lon) , n.latitude = tofloat(n.lat) 
```
NOTE: we should probably have changed the load script to do this for us but didn't realise until after we had loaded the data


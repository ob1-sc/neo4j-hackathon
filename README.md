# neo4j-hackathon
Neo4J Hackathon - Neo Noobs - London Crime Discovery Visualisation 

## Overview
Neo Noobs = two Neo4j virgins (Jacques and Simon)
Minute 1 of Hackathon = 
Minute 120 of Hackathon = see below

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

## Crime Discovery
* We wanted to find crimes within 2KM of the Hackathon so that we can visually identify crime hotspots where we can target security guard, the cypher query we used was:
```
MATCH (n:Point)<-[:OCCURED_AT]-()-[:IS_TYPE]->(y:CrimeType)
with n,distance(point(n),point({latitude:51.518,longitude:-0.0861})) as d, y
where d < 20000
return n.latitude as latitude, n.longitude as longitude, y.name as CrimeType
```

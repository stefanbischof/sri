# Competency Questions

The list of competency questions.

#### CQ 1: Who performed an assessment for a building and when?

```sparql
SELECT ?agentIRI ?fullName ?id ?email ?creationDate WHERE {
   sri:assessment1 prov:wasAttributedTo ?agentIRI ;
        dcterms:created ?creationDate . 
    ?agentIRI foaf:name ?fullName ;
        foaf:nick ?id ;
        foaf:mbox ?mbox .  
    BIND(REPLACE(STR(?mbox),"mailto:","") AS ?email) 
}
```
##### Result

| agentIRI | fullName | id | email | creationDate |
|---|---| --- | ---|---|
| sri:agent1 | "John Doe"  | "abc123" | "john.doe@example.com" | "2023-11-28"^^xsd:date |

#### CQ 2: What is the SRI score of a building?

```sparql
SELECT ?score WHERE { 
    ?dataset rdf:type qb:DataSet ;
        qb:structure sri:dsd-scores ;
        prov:wasDerivedFrom sri:assessment1 .
    ?observation qb:dataSet ?dataset ;
        sri:function sri:topDomain ;
        sri:impact sri:topImpact ;
        sri:score ?score .
}
```
##### Result

| score | 
|---|
| "3.52"^^xsd:float |

#### CQ 3: What are the impact scores of an assessment (in a specific order)?

```sparql
SELECT ?impactIRI ?impactLabel (ROUND(?score_*100)/100 as ?score) WHERE { 
    VALUES (?impactLabel ?position) {
        ("Energy efficiency"@en 1) ("Energy flexibility and storage"@en 2) ("Comfort"@en 3) ("Convenience"@en 4) ("Health well being and accessibility"@en 5) ("Maintenance and fault prediction"@en 6) ("Information to occupants"@en 7)
    }
    ?dataset rdf:type qb:DataSet ;
        qb:structure sri:dsd-scores ;
        prov:wasDerivedFrom sri:assessment1 .    
	?observation qb:dataSet ?dataset ;
        sri:function sri:topDomain ;
        sri:impact ?impactIRI ;
        sri:score ?score_ .
    ?impactIRI rdf:type sri:Impact ;
        rdfs:label ?impactLabel .
} ORDER BY ?position
```
##### Result
| impactIRI | impactLabel | score |
|---|---|---|
| sri:energy_efficiency | "Energy efficiency" | "5.77"^^xsd:float |
| sri:energy_flexibility_and_storage | "Energy flexibility and storage" | "0"^^xsd:float |
| sri:comfort | "Comfort" | "8.74"^^xsd:float |
| ... | ... | ... |



#### CQ 4: What are the SRI scores for the different SRI services of a given building and assessment?

```sparql
SELECT ?domainIRI ?domainLabel 
	(ROUND(SUM(?comfort_)*100)/100 as ?comfort) 
	(ROUND(SUM(?convenience_)*100)/100 as ?convenience)
	(ROUND(SUM(?health_)*100)/100 as ?health) 
	(ROUND(SUM(?informationToOccupants_)*100)/100 as ?informationToOccupants) 
	(ROUND(SUM(?energyFlexibility_)*100)/100 as ?energyFlexibility) 
	(ROUND(SUM(?energyEfficiency_)*100)/100 as ?energyEfficiency) 
	(ROUND(SUM(?maintenanceFaultPrediction_)*100)/100 as ?maintenanceFaultPrediction) WHERE {
    {
    ?observation sri:function ?domainIRI ;
                 sri:impact sri:comfort ;
                 sri:score ?comfort_ .
    } UNION {
    ?observation sri:function ?domainIRI ;
                 sri:impact sri:convenience ;
                 sri:score ?convenience_ .
    } UNION {
    ?observation sri:function ?domainIRI ;
                 sri:impact sri:health_well_being_and_accessibility ;
                 sri:score ?health_ .
    } UNION {
    ?observation sri:function ?domainIRI ;
                 sri:impact sri:information_to_occupants ;
                 sri:score ?informationToOccupants_ .
    } UNION {
    ?observation sri:function ?domainIRI ;
                 sri:impact sri:energy_flexibility_and_storage ;
                 sri:score ?energyFlexibility_ .
    } UNION {
    ?observation sri:function ?domainIRI ;
                 sri:impact sri:energy_efficiency ;
                 sri:score ?energyEfficiency_ .
    } UNION {
    ?observation sri:function ?domainIRI ;
                 sri:impact sri:maintenance_and_fault_prediction ;
                 sri:score ?maintenanceFaultPrediction_ .
    }
    {
        SELECT ?observation WHERE { 
            sri:building1 ^sri:hasBuilding ?dataset .
            ?dataset rdf:type qb:DataSet ;
                     qb:structure sri:dsd-scores ;
                     prov:wasDerivedFrom sri:assessment1 .
            ?observation qb:dataSet ?dataset ;
                      rdf:type qb:Observation .
        }
    }
    ?domainIRI rdfs:label ?domainLabel ; 
		   rdf:type sri:Domain .
} GROUP BY ?domainIRI ?domainLabel
```
##### Result
| domainIRI | domainLabel | comfort | convenience | health | ... |
|---|---|---|---|---|---|
| sri:heating | "Heating" | "16.67"^^xsd:decimal | "18.18"^^xsd:decimal | "40"^^xsd:decimal | ... |
| sri:dHW | "Domestic Hot Water" | "0"^^xsd:decimal | "0"^^xsd:decimal | "0"^^xsd:decimal | ... |
| sri:cooling | "Cooling" | "10"^^xsd:decimal | "18.18"^^xsd:decimal | "40"^^xsd:decimal | ... |
| ... | ... | ... | ... | ... | ... |

#### CQ 5: How did the SRI score change over time between two assessments of a building?

```sparql
SELECT ?SRI ?score1 ?score2 WHERE {{
    SELECT ("SRI Score" as ?SRI) ?score1 WHERE {
        VALUES ?assessmentIRI1 {sri:assessment1}
        ?assessmentIRI1 sri:hasBuilding ?buildingIRI1 ;
                        rdf:type qb:DataSet ;
                        qb:structure sri:dsd-assessment .	
        ?scoresIRI1 rdf:type qb:DataSet ;
                   prov:wasDerivedFrom ?assessmentIRI1 ;
                   qb:structure sri:dsd-scores .
        ?observation1 qb:dataSet ?scoresIRI1 ;
                     sri:function sri:topDomain ;
                     sri:impact sri:topImpact ;
                     sri:score ?value_1 .
        BIND(STR(ROUND(?value_1*100)/100) as ?score1)      
}}{
        SELECT ("SRI Score" as ?SRI) ?score2 WHERE {
            VALUES ?assessmentIRI2 {sri:assessment2}
            ?assessmentIRI2 sri:hasBuilding ?buildingIRI2 ;
                            rdf:type qb:DataSet ;
                            qb:structure sri:dsd-assessment .
                ?scoresIRI2 rdf:type qb:DataSet ;
                           prov:wasDerivedFrom ?assessmentIRI2 ;
                           qb:structure sri:dsd-scores .
                ?observation2 qb:dataSet ?scoresIRI2 ;
                             sri:function sri:topDomain ;
                             sri:impact sri:topImpact ;
                             sri:score ?value_2 .
                BIND(STR(ROUND(?value_2*100)/100) as ?score2)
}}} GROUP BY ?SRI ?score1 ?score2
```
##### Result

| domainLabel | score 1 | score2 |
|---|---| -- |
| SRI Score |  3.53  | 42.95 |
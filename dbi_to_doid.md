# Disease name to DOID (for DB interface)

## Parameters

* `string`
  * default: cancer

## `to_code`

```javascript
({string})=>{
  return {code: "\"" + decodeURIComponent(string).toLowerCase().split(",").join("\"^^xsd:string \"") + "\"^^xsd:string" };
};
```

## Endpoint

{{SPARQLIST_EP}}

## `search_doid`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX oboowl: <http://www.geneontology.org/formats/oboInOwl>
SELECT DISTINCT ?id
WHERE {
  VALUES ?disease_word { {{to_code.code}} }
  ?id (rdfs:label|oboowl:hasExactSynonym|oboowl:hasNarrowSynonym|oboowl:hasRelatedSynonym) ?disease .
  FILTER (LCASE(?disease) = ?disease_word)
}
```

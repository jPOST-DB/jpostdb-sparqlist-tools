# get PSM SPARQL (for DB download)

## Parameters

* `dataset`
  * default: DS1631_1
* `offset`
  * default: 0
* `limit`
  * default: 10000
* `count`
  * example: 1

## `filter`

```javascript
({offset, limit, count})=>{
  var select_line ="SELECT DISTINCT ?psm_id ?sequence ?accession ?exp_mz ?calc_mz ?charge ?jpost_score";
  var limit_line = "";
  if(count) select_line = "SELECT (COUNT(?psm_id) AS ?count)";
  else limit_line = "ORDER BY ?sequence ?psm_id ?accession DESC(?jpost_score)\nOFFSET " + offset + "\nLIMIT " + limit;
  return {select: select_line, limit: limit_line };
};
```

## Endpoint

{{SPARQLIST_EP}}

## `get_line`

```sparql
#DEFINE sql:select-option "order"
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX : <http://rdf.jpostdb.org/entry/>
{{filter.select}}
WHERE {
  :{{dataset}} jpo:hasProtein ?prt .
  ?prt rdfs:label ?accession .
  ?prt jpo:hasPeptideEvidence/jpo:hasPeptide ?pep .
  ?pep jpo:hasSequence [ a obo:MS_1001344 ;
                       rdf:value ?sequence ] .
  ?pep jpo:hasPsm ?psm .
  ?psm dct:identifier ?psm_id .
  ?psm sio:SIO_000216 [ a jpo:UniScore ;
                      sio:SIO_000300 ?jpost_score ] .
  ?psm sio:SIO_000216 [ a obo:MS_1000041 ;
                      sio:SIO_000300 ?charge ] .
  ?psm sio:SIO_000216 [ a jpo:CalculatedMassToCharge ;
                      sio:SIO_000300 ?calc_mz ] .
  ?psm sio:SIO_000216 [ a jpo:ExperimentalMassToCharge ;
                      sio:SIO_000300 ?exp_mz ] .
                      
}
{{filter.limit}}
```

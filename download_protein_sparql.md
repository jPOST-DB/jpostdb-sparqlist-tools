# get protein SPARQL (for DB download)

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
  var select_line ="SELECT DISTINCT ?protein_id ?accession ?symbol ?name ?protein_type ?pep_count ?psm_count (COUNT (DISTINCT ?uniq_pep) AS ?uniq_pep_count)";
  var limit_line = "";
  if(count) select_line = "SELECT (COUNT(?protein_id) AS ?count)";
  else limit_line = "ORDER BY ?protein_type DESC(?pep_count) \nOFFSET " + offset + "\nLIMIT " + limit;
  return {select: select_line, limit: limit_line };
};
```

## Endpoint

{{SPARQLIST_EP}}

## `get_line`

```sparql
DEFINE sql:select-option "order"
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
  ?prt dct:identifier ?protein_id .
  ?prt a ?type .
  FILTER (?type != jpo:Protein)
  FILTER (?type != obo:MS_1001591)
  ?type rdfs:label ?protein_type .
  ?prt jpo:hasDatabaseSequence ?up .
  ?up uniprot:mnemonic ?symbol .
  ?up (uniprot:recommendedName|uniprot:submittedName)/uniprot:fullName ?name .
  ?prt sio:SIO_000216 [ a obo:MS_1001097 ;
                        sio:SIO_000300 ?pep_count ] .
  ?prt sio:SIO_000216 [ a obo:MS_1002153 ;
                        sio:SIO_000300 ?psm_count ] .
  OPTIONAL {
    ?prt jpo:hasPeptideEvidence/jpo:hasPeptide ?uniq_pep .
    ?uniq_pep a jpo:UniquePeptideAtMsLevel .
  }
}
{{filter.limit}}
```

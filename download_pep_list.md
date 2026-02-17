# Discontinued
- download peptide list (for UniProt -> jPOST link) old
- download_tax_dataset_list -> download_pep_list_ds でDatasetごとに取得に変更. 
- sort, unique はPerl API側で対応

## Parameters

* `tax`
  * default: 9606
* `offset`
  * default: 0
* `limit`
  * default: 100000
* `count`
  * example: 1

## `filter`

```javascript
({offset, limit, count})=>{
  var select_line = "SELECT ?sequence ?jpost_score ?global_fdr ?dataset_id";
  var limit_line = "";
  var code_line = "?pep sio:SIO_000216 [ a jpo:UniScore ;\nsio:SIO_000300 ?jpost_score ] .\n?pep sio:SIO_000216 [ a obo:MS_1001364 ;\nsio:SIO_000300 ?global_fdr ] . ";
  if(count){ 
  	select_line = "SELECT (COUNT(?pep) AS ?count)";
    code_line = "";
  }else{
    limit_line = "ORDER BY ?sequence DESC(?jpost_score) ?global_fdr\nOFFSET " + offset + "\nLIMIT " + limit;
  }
  return {select: select_line, limit: limit_line, code: code_line};
};
```
## Endpoint

{{SPARQLIST_EP}}

## `get_up_list`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX tax: <http://identifiers.org/taxonomy/>
PREFIX dct: <http://purl.org/dc/terms/>
{{filter.select}}
WHERE {
  ?dataset jpo:hasProfile/jpo:hasSample/jpo:species tax:{{tax}} ;
  dct:identifier ?dataset_id ;
  jpo:hasPeptide ?pep .
  ?pep jpo:hasSequence/rdf:value ?sequence .
{{filter.code}}
}
{{filter.limit}}
```

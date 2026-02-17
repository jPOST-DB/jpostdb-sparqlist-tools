# get modification SPARQL

- Download 用ファイル作成 API
  - Modification

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
  var select_line ="SELECT DISTINCT ?psm_id ?modification ?site ?position\nWHERE{{\n  SELECT DISTINCT ?psm_id ?modification ?site ?position";
  var limit_line = "";
  if(count) select_line = "SELECT (COUNT(?psm_id) AS ?count)";
  else limit_line = "}}\nORDER BY ?psm_id ?position ?modification ?site\nOFFSET " + offset + "\nLIMIT " + limit;
  return {select: select_line, limit: limit_line };
};
```

## Endpoint

{{SPARQLIST_EP}}

## `get_line`

```sparql
DEFINE sql:select-option "order"
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX : <http://rdf.jpostdb.org/entry/>
{{filter.select}}
WHERE {
  :{{dataset}} jpo:hasProtein/jpo:hasPeptideEvidence/jpo:hasPeptide/jpo:hasPsm ?psm .
  ?psm dct:identifier ?psm_id .
  ?psm jpo:hasModification ?mod_blank .
  ?mod_blank a ?mod ;
             jpo:modificationSite ?site .
  OPTIONAL { ?mod_blank faldo:location/faldo:position ?position . }
  ?mod rdfs:label ?modification .
}
{{filter.limit}}
```

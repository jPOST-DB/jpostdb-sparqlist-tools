# download uniprot accession list (for UniProt -> jPOST link)

- UniProt 側での Link 作成のため API で使用
  - UniProt accession リスト

## Parameters

* `offset`
  * default: 0
* `limit`
  * default: 10000
* `count`
  * example: 1

## `filter`

```javascript
({offset, limit, count})=>{
  var select_line = "SELECT DISTINCT ?accession (REPLACE (STR (?tax), tax:, '') AS ?tax_id)";
  var limit_line = "";
  var code_line = "  ?protein \^jpo:hasProtein/jpo:hasProfile/jpo:hasSample/jpo:species ?tax .";
  if(count){
        select_line = "SELECT (COUNT(DISTINCT ?accession) AS ?count)";
    code_line = "";
  }else{
    limit_line = "ORDER BY ?tax_id ?accession \nOFFSET " + offset + "\nLIMIT " + limit;
  }
  return {select: select_line, limit: limit_line, code: code_line};
};
```
## Endpoint

{{SPARQLIST_EP}}

## `get_up_list`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX tax: <http://identifiers.org/taxonomy/>
PREFIX : <http://rdf.jpostdb.org/entry/>
{{filter.select}}
WHERE {
  ?protein jpo:hasDatabaseSequence ?up ;
           a jpo:Protein ;
           rdfs:label ?accession .
{{filter.code}}
}
{{filter.limit}}
```
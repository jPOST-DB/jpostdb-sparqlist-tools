# download pmid list (for UniProt -> jPOST link)

- UniProt 側での Link 作成のため API で使用
  - Taxonomy, Pubmed ID 対応

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
  var select_line = "SELECT DISTINCT (REPLACE (STR (?tax), tax:, '') AS ?tax_id)  (REPLACE(STR(?pubmed), 'https://rdf.ncbi.nlm.nih.gov/pubmed/', '') AS ?pubmed_id) ";
  var limit_line = "";
  if(count){ 
  	select_line = "SELECT (COUNT(DISTINCT ?pubmed) AS ?count)";
  }else{
    limit_line = "ORDER BY ?project_id \nOFFSET " + offset + "\nLIMIT " + limit;
  }
  return {select: select_line, limit: limit_line};
};
```
## Endpoint

https://tools.jpostdb.org/proxy/sparql

## `get_up_list`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX jpo: <http://rdf.jpostdb.org/ontology/jpost.owl#>
PREFIX tax: <http://identifiers.org/taxonomy/>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX px: <https://github.com/PX-RDF/ontology/blob/master/px.owl#>
PREFIX : <http://rdf.jpostdb.org/entry/>
{{filter.select}}
WHERE {
  ?project a jpo:Project ;
           dct:identifier ?project_id ;
           dct:references ?pubmed ;
           jpo:hasDataset/jpo:hasProfile/jpo:hasSample/jpo:species ?tax .
  FILTER (REGEX (STR (?pubmed), "ncbi"))
}
{{filter.limit}}
```
